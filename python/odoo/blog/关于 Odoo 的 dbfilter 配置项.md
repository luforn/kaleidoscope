## 关于 Odoo 的 dbfilter 配置项  
概述  
默认情况下首次访问odoo页面时，会要求选择要访问的数据库，db中的所有库都会被列出来供选择，这种在生产环境下通常是不希望的看到，如果在启动时指定连接的数据库名可以解决这个问题

.conf文件中指定 db_name = xxx

或者启动命令加参数 -d xxx

dbfilter  
当我们需要根据域名来匹配数据库时（比如saas环境）这样就不适用了，这个时候就可以用 dbfilter 这个配置项来实现

dbfilter 默认值为 .*

eg: dbfilter = ^%h$ 表示按域名精确匹配数据库服务器中名称为域名的数据库

启动参数 --db-filter='^%d$' 表示按二级域名前缀精确匹配对应名称的数据库（注意：127.0.0.1访问时会被匹配为 127 库名）

可用的匹配替代符号有 %h 和 %d

%h  
%h 代表访问访问的域名，比如 www.abc.com

%d  
当访问地址为 www.abc.com 时 %d 为 abc
当访问地址为 shop.abc.com 时 %d 为 shop  


相关源代码  
odoo中的相应的解析代码  
```python
def db_filter(dbs, httprequest=None):
    httprequest = httprequest or request.httprequest
    h = httprequest.environ.get('HTTP_HOST', '').split(':')[0]
    d, _, r = h.partition('.')
    if d == "www" and r:
        d = r.partition('.')[0]
    r = openerp.tools.config['dbfilter'].replace('%h', h).replace('%d', d)
    dbs = [i for i in dbs if re.match(r, i)]
    return dbs
```
扩展  
笔者由于在saas环境的需求扩展了两个配置参数用于控制访问的路由以及域名绑定

dbhost = xxx.com  
dbhost 用于控制针对哪个域名做二级域名的路由  
dbmap = dbmap.json  
dbmap 指定的json文件中指明了具体映射路由表及域名绑定关系  
dbmap.json 配置方法  
{“二级子域名”: [“绑定的外部域名”, “数据库名”], …}  
示例：  
```python
{
  "test": ["www.test.com", "test.odoo.com"],
  "odoo": ["","odoo-server"]    // 访问域名：www.odoo.com
  "abc": ["","abc-server"]    //访问域名：abc.odoo.com
}
```  
如果不想列出所有 db 则在 .conf 中添加 dbfilter = ^%h$

实现的关键代码如下：  
```python
def db_filter(dbs, httprequest=None):
    httprequest = httprequest or request.httprequest
    h = httprequest.environ.get('HTTP_HOST', '').split(':')[0]
    d, _, r = h.partition('.')
    rg = '^$'
    m_config = openerp.tools.config
    if r in m_config['dbhost']:
        if d == "www" and r:
            d = r.partition('.')[0]
        rg = m_config['dbfilter'].replace('%h', h).replace('%d', d)
        if d in m_config['ukey_dbname']:
            rg = '^%s$'%m_config['ukey_dbname'][d]
    else:
        if h in m_config['host_ukey']:
            ukey = m_config['host_ukey'][h]
            if ukey in m_config['ukey_dbname']:
                rg = '^%s$'%m_config['ukey_dbname'][ukey]
            else:
                rg = '^%s$'%ukey
    else:
        rg = '^$'
    dbs = [i for i in dbs if re.match(rg, i)]
    return dbs
```