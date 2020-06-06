## 集群相关 
https://cloud.tencent.com/developer/article/1483577

https://cloud.tencent.com/developer/article/1185379

https://cloud.tencent.com/developer/article/1483577

https://cloud.tencent.com/developer/article/1449603

https://cloud.tencent.com/developer/article/1554435

https://cloud.tencent.com/developer/article/1563023

https://cloud.tencent.com/developer/article/1550324

https://segmentfault.com/a/1190000007012082

https://segmentfault.com/a/1190000018875088

https://www.open-open.com/lib/view/open1433400666369.html

https://my.oschina.net/Kenyon/blog/309306

https://yq.aliyun.com/articles/225163?spm=5176.10695662.1996646101.searchclickresult.6eb615f7b54vNB

https://yq.aliyun.com/articles/357098?spm=5176.10695662.1996646101.searchclickresult.6eb615f7b54vNB

https://www.jianshu.com/p/77f07af6ca4b

http://postgres.cn/downfiles/pgconf_2018/PostgresChina2018_%E9%99%88%E6%B2%B3%E5%A0%86_PostgreSQL%E5%9F%BA%E4%BA%8EPaaS%E5%B9%B3%E5%8F%B0%E7%9A%84%E9%AB%98%E5%8F%AF%E7%94%A8%E9%9B%86%E7%BE%A4%E6%96%B9%E6%A1%88V3.pdf

https://yq.aliyun.com/articles/582876

https://yq.aliyun.com/articles/700349?spm=a2c4e.11153940.0.0.1e377e97ua6YS0

https://yq.aliyun.com/articles/126901?spm=a2c4e.11153940.0.0.47e464b8zYgt3V

https://yq.aliyun.com/articles/59251?spm=a2c4e.11153940.0.0.4ae016b5tKPTqv

https://yq.aliyun.com/articles/501454?spm=a2c4e.11153940.0.0.4ae016b5tKPTqv

https://sites.google.com/site/muguazhijia/za-tan-1/postgresqljiqunbushufangan

http://www.gdmzlgy.com/article/2359415.html

https://www.jianshu.com/p/77f07af6ca4b

https://blog.51cto.com/12924846/2363728

https://wenku.baidu.com/view/7218f1ed85254b35eefdc8d376eeaeaad1f31603.html

https://www.jianshu.com/p/09a29507fd4e

https://zhuanlan.zhihu.com/p/66517407

https://www.bbsmax.com/A/WpdKBYWZ5V/

https://www.dazhuanlan.com/2019/10/12/5da0dfead89f3/

http://www.pgsql.tech/article_102_10000013

https://daydaynews.cc/zh-hk/technology/327902.html

http://www.jeepxie.net/article/530455.html

https://www.shuzhiduo.com/A/LPdoolQwd3/

http://www.htsjk.com/shujukunews/3087.html

http://www.51testing.com/html/28/116228-810268.html

https://www.ctolib.com/topics-142914.html

https://postgres.fun/20120603135702.html

https://www.douban.com/group/topic/141827646/

http://javaquan.com/post/32838_1_1.html

http://m.vlambda.com/wz_xdNhc2XoCr.html

https://olei.me/829/

https://www.liangzl.com/get-article-detail-36217.html

https://www.520mwx.com/view/74656

https://www.jishuwen.com/d/2hSc

https://segmentfault.com/a/1190000022749232

https://segmentfault.com/a/1190000022790065

https://svtter.cn/2018/12/01/%E4%BD%BF%E7%94%A8-docker-swarm-%E6%9E%84%E5%BB%BA-postgresql-%E9%9B%86%E7%BE%A4/

https://lihaoquan.me/2018/9/29/postgresql-master-slave-ha.html

http://www.cslingjun.com/2019/11/27/PostgreSQL%E9%AB%98%E5%8F%AF%E7%94%A8%E9%9B%86%E7%BE%A4repmgr/

http://www.cslingjun.com/2019/11/27/PostgreSQL%E9%AB%98%E5%8F%AF%E7%94%A8%E9%9B%86%E7%BE%A4repmgr/

https://www.cnblogs.com/EikiXu/p/9883596.html









对于业内来说，基本都在围绕主从切换的高可用方案：

http://www.10tiao.com/html/175/201509/210974337/1.html

https://www.jianshu.com/p/ef183d0a9213

https://my.oschina.net/Kenyon/blog/98217

一主多从实现读写分离，完全依赖第三方激情软件的pgpool：

http://www.bkjia.com/Javascript/1188773.html

https://github.com/digoal/blog/blob/master/201708/20170807_01.md

https://www.jianshu.com/p/77f07af6ca4b



一主一从实现读写分离：

https://www.cnblogs.com/yjf512/p/4499547.html



我想到的一主多从，丛库依靠HAProxy多负载均衡，暴露给开发的就两个IP，写IP和读IP，有业务进行控制：

https://my.oschina.net/Kenyon/blog/664102

https://cloud.tencent.com/developer/article/1026385