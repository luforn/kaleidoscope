## Odoo博文清单  

### odoo超级用户密码遗忘后怎么办？   
有时候我们可能会遗忘odoo管理员用户密码，无法登录就无法在界面中修改，可以采用如下方法：

1、生成加密的密码;启动python控制台，输入：  
from passlib.context import CryptContext
print(CryptContext(['pbkdf2_sha512']).encrypt('123456'))  
2、用数据库工具连接数据库，打开res_users表，找到遗忘密码的管理用户，把password字段的值改为上一步生成的加密密码。  

### 在odoo的action window中指定使用view  
```xml
<record id="action_account_bank_journal_form" model="ir.actions.act_window">
  <field name="name">Bank Accounts</field>
  <field name="res_model">account.journal</field>
  <field name="view_mode">tree,kanban,form</field>
  <field name="view_ids" eval="[(5, 0, 0),
    (0, 0, {'view_mode': 'tree', 'view_id': ref('view_account_bank_journal_tree')}),
    (0, 0, {'view_mode': 'kanban', 'view_id': ref('account_bank_journal_view_kanban')}),
    (0, 0, {'view_mode': 'form', 'view_id': ref('view_account_bank_journal_form')})]"/>
  <field name="domain">[('type', '=', 'bank')]</field>
  <field name="context">{'default_type': 'bank'}</field>
  <field name="help" type="html">
    <p class="o_view_nocontent_smiling_face">
      Create a bank account
    </p>
  </field>
</record>
```

### Odoo 列表视图如何开启横向自动滚动条  
```css
.o_list_view  td, .o_list_view th { white-space: nowrap; }
.o_list_view  > tbody > tr > td.o_handle_cell { white-space: nowrap; }
```

