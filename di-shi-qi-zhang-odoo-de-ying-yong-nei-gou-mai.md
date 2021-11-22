# 第十七章 Odoo的应用内购买

Odoo在版本11中引入了对IAP的内置支持。IAP用于提供循环收费服务而又无需进行复杂配置。通常，从应用商店中购买的应用只需要由客户支付一次费用，因为它们是常规模块，模块的使用无需花费开发者任何成本。对比这种，IAP应用用于向用户提供服务，因此，在提供持续性服务时会存在运营成本。在这种情况下，不可能只进行一次初始购买就提供服务。服务提供者需要以循环的方式按照使用来向用户收费。Odoo IAP解决了这一问题并提供一种按使用收费的方式。

本章中，我们将讲解如下内容：

* 应用内购买（IAP）的概念
* 在Odoo中注册一个IAP服务
* 创建一个IAP服务模块
* 授权并收取IAP积分
* 创建一个IAP客户端模块
* 在账户缺少积分时显示报价

有几种可以使用IAP的情形，比如发送文件的传真服务或短信服务。本章中，我们将创建一个小IAP服务来根据输入的ISBN数字提供完整的图书信息。

### 技术准备

学习本章要求有一个在线Odoo平台。

本章中使用的所有代码可通过GitHub仓库进行下载：https://github.com/alanhou/odoo14-cookbook/tree/main/Chapter17。

### 应用内购买（IAP）的概念

本节中，我们将探讨IAP流程中的不同实体。我们还会学习每个实体的角色以及它们如何共同完成IAP的流程。

#### 运行原理…

在IAP流程中有3种主要实体：客户、服务提供者和Odoo本身。描述如下：

* 客户是要使用服务的终端用户。为使用服务，客户需要安装由服务提供者所提供的应用。然后客户需要根据他们的使用城求购买一个服务方案。这样，客户就可以开始直接使用服务了。这避免了给客户带来的麻烦，因为无需执行复杂的配置。反而他们只要购买服务就可以开始使用了。
* 服务提供者是希望销售服务的开发人员（可能就是你）。客户会向提供者要求服务，这时服务提供者会检查客户购买的是否为有效服务以及账户中是否还有足够的积分。如果客户拥有足够的积分，服务提供者会扣除积分并向客户提供服务。
* 这里Odoo是一种中间商。它提供处理付款、积分、销售方案等的媒介。客户从Odoo充值服务积分，服务提供者在提供服务时提取积分。然后Odoo充当客户和服务提供者之间的桥梁，这样客户无需进行复杂的配置，而服务提供者无需设置支付网关、客户账号管理等。作为回报，Odoo从销售中提取佣金。在写本书时，Odoo从购买包中拿走25%作为佣金。

在这个流程中还有一个可选实体，即外部服务。在一些情况下，服务提供商使用某些外部服务。但我们在这里将忽略外部服务，因为他们是二级服务提供者。其中的一个例子可能是SMS短信服务。如果你向Odoo用户提供SMS IAP服务，那么你（服务提供者）将在内部使用SMS服务。

**IAP 服务流**

现在，我们学习所有的IAP实体如何一起协作提供服务。下图中描绘了IAP的流程：

![【翻译中】Odoo 14开发者指南第十七章 Odoo的应用内购买](https://i.cdnl.ink/homepage/wp-content/uploads/2021/03/2021060105171335.jpg)

图17.1 – IAP工作流

以下是IAP服务流每个步骤的讲解：

1. 客户向服务提供服务商请求提供服务。通过这个请求，客户将传递账号令牌，服务提供商使用它来识别用户。（注意客户需要在服务中安装你的模块）。
2. 在接收到来自客户的请求之后，服务提供者将询问Odoo客户的账户里是否有足够的积分。如果客户有足够的积分，那么它就会创建交易来在提供服务前预留积分。
3. 在预留积分之后，服务提供者会执行服务。在某些情况下，服务提供者会调用外部服务来执行所请求的服务。
4. 在执行由客户所请求的服务之后，服务提供者回到Odoo来获取第2步中所预留的积分。如果所请求的服务由于错误未能正常提供，服务提供者会要求Odoo释放所预留的积分。
5. 最后，服务提供者会回到客户，通知他们所请求的服务已成功提供。一些服务可能会返回结果信息，这时你就会得到服务的结果。这一结果信息由客户根据他们的规格（取决于服务）来使用。

#### 扩展知识…

如果客户没有足够的积分，服务流如下：

1. 客户请求服务（和前面的流程一样）。
2. 服务提供者得到请求并询问Odoo用户是否有足够的积分。假定客户没有足够的积分。
3. 服务提供者返回到客户并告知他们在账户中没有足够的积分，显示用户可以购买服务的信息（一个Odoo服务包链接）。
4. 客户重定向到Odoo并进行服务的充值。

### 在Odoo中注册一个IAP服务

为能从客户账户中提供积分，服务提供者需要在Odoo上注册服务。还要为服务定义方案。用户将通过这一注册服务来购买方案。本节中，我们将在Odoo上注册我们的服务并为服务定义方案。

#### 准备工作…

通过IAP平台销售服务，服务提供者需要在Odoo注册服务和方案。我们将在https://iap-sandbox.odoo.com/上注册服务。这个IAP端点用于进行测试。可以免费购买一个服务包。对于生产环境，需要在https://iap.odoo.com注册服务。本节中我们将使用IAP沙盒的端点。

#### 如何实现…

按照如下步骤来在Odoo上创建IAP服务：

1. 打开https://iap-sandbox.odoo.com/并登录（如没有账户请注册）。
2. 在首页中点击My Services按钮。
3. 点击Add a Service按钮来新建服务。
4. 这会打开如下图的表单。此处，填写包含服务Logo、技术名称（必须唯一）、单位名称、隐私政策等信息。\
   ![【翻译中】Odoo 14开发者指南第十七章 Odoo的应用内购买](https://i.cdnl.ink/homepage/wp-content/uploads/2021/03/2021060909192065.jpg)\
   图17.2 – 注册IAP服务
5. 保存服务会显示服务密钥，如下图所示。注意这里的服务密钥不会再次显示：\
   ![【翻译中】Odoo 14开发者指南第十七章 Odoo的应用内购买](https://i.cdnl.ink/homepage/wp-content/uploads/2021/03/2021060908332512.jpg)\
   图17.3 – 新增的IAP服务
6. 通过在Packs版块中点击Add a pack按钮创建一些服务包（方案）。例如，Get 50 books info in 10 Euro.。下图显示了创建新服务包页面的截图：\
   ![【翻译中】Odoo 14开发者指南第十七章 Odoo的应用内购买](https://i.cdnl.ink/homepage/wp-content/uploads/2021/03/2021060909004948.jpg)\
   图17.4 – IAP新服务包

在配置结束后，服务的页面会是这样：

![【翻译中】Odoo 14开发者指南第十七章 Odoo的应用内购买](https://i.cdnl.ink/homepage/wp-content/uploads/2021/03/2021060909153819.jpg)\
图17.5 – 配置好服务包之后的IAP服务

可以随时新增服务包。也可以随时修改包的内容，但在截至写本书时，仍无法删除服务包。

#### 运行原理…

我们在https://iap-sandbox.odoo.com/上创建了一个IAP服务，因为我们希望在迁移到生产环境前测试下IAP服务。让我们来了解下创建服务时所填写的字段的用途吧：

* 技术名称（Technical name ）用于标识服务，它必须是唯一的名称。我们在这里添加了library\_isbn。（此后无法修改这一名称）
* 标签、描述和服务Logo用于提供信息。这一信息会在用户购买服务的网页中显示。
* 单位名是服务销售的单位。例如，在SMS服务中，单位名是SMS（例如100条短信$5）。本例中，我们使用了Books info作为单位名。
* Trial Credits是提供给客户用于测试的免费积分。它对每个客户仅提供一次。同时测试积分仅在用户具有有效合同后才有效，用于避免免费积分的滥用。

在提交这些详情后，就会创建好服务并且会显示服务密钥。参见本节第5步中的截图来获取更多信息。在请求服务时会使用到这个密钥获取用户的积分。安全存放这一密钥，因为它不会再次显示，但是可以在该页面中生成一个新的密钥，老密钥就会失效。

还需要为我们的服务创建方案。你需要提供方案名、描述、logo、数量和价格。**Amount**字段用于该方案的服务单位的数量。**Price**字段用于定义用户获取这个方案所要支付的金额。在本节的第6步中，我们创建了一个50 Books Info in 10 Euro的方案。这里Books Info是我们在创建服务时所提交的单位。表示如果用户购买了这一方案，他将能够获取到50本书的信息。

> **📝注：**Odoo从这一报价中抽取25%的佣金，因此请相应地进行服务方案定价。

现在，我们将在接下来的小节中创建一个IAP服务以及IAP客户端模块。

### 创建一个IAP服务模块

本节中，我们将创建一个服务模块来由服务提供者使用。这个模块将接收来自客户的IAP请求并在响应中返回服务结果。

#### 准备工作…

我们将创建iap\_isbn\_service模块。这个服务模块将处理客户的IAP请求。客户将发送带有ISBN号的图书信息请求。这个服务模块会从客户账户收取积分并返回书名、作者及封面图等信息。

要便于理解，我们将分成两节来开发一个服务模块。本节中，我们将创建一个创建图书信息表的基本模块。在客户进行请求时，服务提供者会通过在这个表中搜索来返回图书信息。下一节中，我们将添加服务模块的第二部分，在那个模块中， 我们将添加代码来获取积分。

#### 如何实现…

按照如下步骤来生成基本服务模块：

1.  创建一个新的iap\_isbn\_service模块并添加\_init\_\_.py：

    ```
    from . import models
    from . import controllers
    ```
2.  添加\_\_manifest\_\_.py及如下内容：

    ```
    {   
        'name': "IAP ISBN service",   
        'summary': "Get books information by ISBN number",   
        'website': "http://www.example.com",   
        'category': 'Uncategorized',   
        'version': '12.0.1',   
        'depends': ['iap', 'web', 'base_setup'],   
        'data': [       
            'security/ir.model.access.csv',       
            'views/book_info_views.xml',       
            'data/books_data.xml',   
        ]
    }
    ```
3.  在models/book\_info.py添加添加book.info模块及获取图书数据的方法：

    ```
    from odoo import models, fields, apiclass 
    BookInfo(models.Model):  
        _name = 'book.info'
        name = fields.Char('Books Name', required=True)
        isbn = fields.Char('ISBN', required=True)
        date_release = fields.Date('Release Date')
        cover_image = fields.Binary('BooksCover')
        author_ids = fields.Many2many('res.partner', string='Authors') 
        
        @ api.model
        def _books_data_by_isbn(self, isbn):  
            book = self.search([('isbn', '=', isbn)], limit=1)
            if book:    
                return {
                    'status': 'found', 
                    'data': {
                        'name': book.name, 
                        'isbn': book.isbn, 
                        'date_release': book.date_release,
                        'cover_image': book.cover_image,
                        'authors': [a.name for a in book.author_ids]
                    }
                }  
            else:    
                return {
                    'status': 'not found', 
                }
    ```
4.  在controller/main.py文件中添加一个http控制器（别忘记添加controllers/\_\_init\_\_.py文件）：\


    ```
    from odoo import http
    from odoo.http import request 
    class Main(http.Controller):  

        @http.route('/get_book_data', type='json', auth="public")  
        def get_book_data(self):    # 我们会在这里收取金额    
            return {      
                'test': 'data'    
            }
    ```
5.  在security/ir.model.access.csv中添加访问规则：\


    ```
    id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
    acl_book_backend_user,book_info,model_book_info,base.group_user,1,1,1,1

    ```
6.  在views/book\_info\_views.xml中添加视图、菜单和动作：\


    ```
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>  
        <!-- Form View -->    
        <record id="book_info_view_form" model="ir.ui.view">  
              <field name="name">Book Info Form</field>
              <field name="model">book.info</field>       
              <field name="arch" type="xml">            
                    <form>                
                          <sheet>                    
                                <field name="cover_image" widget='image' class="oe_avatar"/>
                                <div class="oe_title">                        
                                      <label for="name" class="oe_edit_only"/>                        
                                      <h1>                            
                                            <field name="name" class="oe_inline"/>                        
                                      </h1>                    
                                </div>                    
                                <group>                        
                                      <group>                            
                                            <field name="isbn"/>                            
                                            <field name="author_ids" widget="many2many_tags"/>                        
                                      </group>                        
                                      <group>                            
                                            <field name="date_release"/>                        
                                      </group>                    
                                </group>                
                          </sheet>            
                    </form>        
              </field>    
        </record>   
        
        <!-- Tree(list) View -->    
        <record id="books_info_view_tree" model="ir.ui.view">        
              <field name="name">Book Info List</field>        
              <field name="model">book.info</field>        
              <field name="arch" type="xml">            
                    <tree>                
                          <field name="name"/>                
                          <field name="date_release"/>            
                    </tree>  
              </field>    
        </record>    
        
        <!-- action and menus -->    
        <record id='book_info_action' model='ir.actions.act_window'>        
              <field name="name">Book info</field>        
              <field name="res_model">book.info</field>        
              <field name="view_type">form</field>        
              <field name="view_mode">tree,form</field>    
        </record>    
        
        <menuitem name="Books Data" id="books_info_base_menu"/>    
        <menuitem name="Books" id="book_info_menu" parent="books_info_base_menu" action="book_info_action"/>
     </odoo>
    ```
7.  在data/books\_data.xml中添加一些示例图书数据（不要忘记在给定目录中添加封面图）：\


    ```
    <?xml version="1.0" encoding="utf-8"?>
    <odoo noupdate="1">  
        <record id="book_info_data_1" model="book.info">    
            <field name="name">Learning PostgreSQL 10</field>    
            <field name="isbn">1788392019</field>    
            <field name="author_ids" eval="[(0, 0, {'name': 'Salahaldin Juba'}), (0, 0, {'name': 'Andrey Volkov'})]"/>    
            <field name="date_release">2017/12/01</field>    
            <field name="cover_image" type="base64"  file="iap_isbn_service/static/img/postgres.jpg"/>  
        </record> 
           
        <record id="book_info_data_2" model="book.info">    
            <field name="name">Odoo 10 Development Essentials</field>    
            <field name="isbn">9781785884887</field>    
            <field name="author_ids" eval="[(0, 0, {'name': 'Daniel Reis'})]"/>    
            <field name="date_release">2016/09/25</field>    
            <field name="cover_image" type="base64"  file="iap_isbn_service/static/img/odoo.jpg"/>  
        </record>
    </odoo>
    ```

在安装模块后，你会看到一个带有图书数据的新菜单，如下：

TODO

#### 运行原理…

现在我们创建了iap\_isbn\_service模块并创建了新的book.info表。把这张表看作主表，这里存储所有图书的数据。在客户请求图书数据时，我们将在这张表中进行搜索。如果查找 到了所请求的数据，我们将收取金额来交换图书数据。

> ℹ️如果你是出于商业目的创建这个服务的话，你就需要拥有这个世界上所有图书的信息。在现实世界中，你会需要一个外部服务来作为图书的信息来源。我们练习假定在book.info表中有所有图书的信息，并且我们仅提供表中所有的图书数据。

在模型中，我们还创建了一个\_books\_data\_by\_isbn()方法。该方法会通过ISBN号查找图书并生成相应的数据来发送回给客户。结果中的status键将用于表明是否找到了这本书的数据。它将用于在未查到图书数据时释放所预留的积分。

我们还有路由added /get\_book\_data。IAP客户会对这个URL发送请求来获取图书详情。我们还需要添加代码来为这个服务获取IAP积分，在下一节中将会进行这一操作。但是，出于测试目的，我们可以像下面这样进行测试请求：

```
curl --header "Content-Type: application/json" \  
--request POST \  
--data "{}" \  
http://localhost:8069/get_book_data
```

它会返回下面这样的信息： {“jsonrpc”: “2.0”, “id”: null, “result”: {“test”: “data”}}。

本节中剩下的步骤来自前一小节，无需进行详细的讲解。在下一节中，我们将更新这个模块来获取客户的积分并向他们返回图书数据。

### 授权并收取IAP积分

本节中我们将完成IAP服务模块。我们将使用IAP平台来授权并从客户账户中获取积分。我们还将添加可选配置来保存本章的_在Odoo中注册一个IAP服务_一节中所生成的服务密钥。

#### 准备工作…

本节中我们将使用iap\_isbn\_service模块。

因为我们使用的是IAP沙盒服务，需要在系统参数中设置一个IAP端点。按照如下步骤来设置IAP沙盒端点：

1. 启用开发者模式。
2. 打开菜单Technical > Parameters > System Parameters
3. 新建记录并添加一个键和值，如下：\
   TODO

#### 如何实现…

为完成这个服务模块，我们将添加一个配置选项来存储服务密钥。按照如下步骤添加新字段来在通过设置中设置isbn\_service\_key：

1.  在res.config.settings中添加isbn\_service\_key字段：\


    ```
    from odoo import models, fields 
    class ConfigSettings(models.TransientModel):  
        _inherit = 'res.config.settings'    
        
        isbn_service_key = fields.Char("ISBN service key", config_parameter='iap.isbn_service_key')
        
    ```
2.  在通用设置视图中添加isbn\_service\_key字段：\


    ```
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <record id="view_general_config_isbn_service" model="ir.ui.view">
            <field name="name">Configuration: IAP service key</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>        
            <field name="arch" type="xml">            
                <div id="business_documents" position="before">                
                    <h2>IAP Books ISBN service</h2>                
                    <div class="row mt16 o_settings_container">                    
                        <div class="col-12 col-lg-6 o_setting_box">                        
                            <div class="o_setting_right_pane">                            
                                <span class="o_form_label">IAP service key</span>                            
                                <div class="text-muted">Generate service in odoo IAP and add service key here</div>                            
                                <div class="content-group">                                
                                    <div class="mt16 row">                                    
                                        <label for="isbn_service_key" class="col-3 collg-3 o_light_label"/>                                    
                                        <field name="isbn_service_key" class="oe_inline" required="1"/>                                
                                    </div>                            
                                </div>                        
                            </div>                    
                        </div>                
                    </div>            
                </div>        
            </field>    
        </record>
    </odoo>
    ```

这会在通用设置中添加一个字段来存储服务密钥，如下图中所示。如果你还记得，我们在本章的_在Odoo中注册一个IAP服务_一节中生成了服务密钥。在这个字段中添加服务密钥。参见下图获取更多信息：

TODO

现在，我们将更新/get\_book\_data控制器来获取客户的积分。更新main.py文件如下：

```
from odoo import http
from odoo.http import request
from odoo.addons.iap.models import iap 
class Main(http.Controller):  
    
    @http.route('/get_book_data', type='json', auth="public")  
    def get_book_data(self, account_token, isbn_number):    
        service_key = request.env['ir.config_parameter'].sudo().get_param('iap.isbn_service_key', False)    
        if not service_key:      
            return {        
                'status': 'service is not active'      
            }    
        credits_to_reserve = 1    
        data = {}    
        with iap.charge(request.env, service_key, account_token, credits_to_reserve):      
            data = request.env['book.info'].sudo()._books_data_by_isbn(isbn_number)      
            if data['status'] == 'not found':        
                raise Exception('Book not found')    
        return data
```

更新该模块来应用这些修改。

#### 运行原理…

为从客户账户提取积分，我们将需要通过IAP平台生成服务密钥。在本章的_在Odoo中注册一个IAP服务_一节中，我们生成了一个服务密钥。（如果你丢了那个密钥也没有问题，可以通过服务页面重新生成）。我们在通用设置中添加了isbn\_service\_key字段，这样我们可以在Odoo中存储服务密钥。你可能注意到我们在文件定义中使用了config\_parameter属性。

在字段中使用这个属性会在ir.config\_parameter模型中存储值，也称之为系统参数。在保存之后，你可以在开发者模式的Technical > Parameters > System Parameters菜单中查看它的值。在获取IAP积分时，我们将从系统参数中获取服务密钥。要从系统参数中获取这些值，可以使用get\_param()。例如，我们可以像这样获取服务密钥：

```
seself.env['ir.config_parameter'].sudo().get_param('iap.isbn_service_key', False)
```

这里，第一个参数是带有我们想要访问的值的参数的键，第二参数是默认值。如果在数据库中不存在所请求的键，那么会返回默认值。

接下来，我们更新了/get\_book\_data路由。此时，它接收两个参数：

* account\_token，用于标识用户的客户令牌。客户为服务所购买的积分将会在IAP平台中与这个account\_token进行关联。服务提供者将在获取积分时发送这一令牌。
* isbn\_number是客户想要通过积分换取信息的图书的ISBN号。

> **小贴士：**此处的这些参数并不固定。我们的示例服务需要一个isbn\_number，因此进行了该值的传递。但是，你可以传递任意数量所需的参数。只是要确保你传递了account\_token，因为没有它你就无法从客户的账户中获取积分。

IAP提供iap.charge()帮助方法，它处理从客户账户中获取积分的流程。charge() 方法接收4个参数：环境、服务密钥、客户账户令牌和想要获取的金额。charge()方法管理如下内容：

* 创建交易对象并保留指定量的积分。如果客户账号没有足够的积分，那么会抛出InsufficientCreditError。
* 如果在客户账户中有足够的积分，它会在with代码块中运行代码。
* 如果with代码块成功运行，它会获取所接收到的积分。
* 如果在with代码块中的代码产生异常，它会翻译所预留的积分，因为无法完成服务请求。

在上例中，我们使用了相同的iap.charge()方法来为图书请求获取积分。我使用了我们的服务密码和客户账号令牌来为图书信息保留1份积分。然后，在with代码块中，我们使用了\_books\_data\_by\_isbn()方法来根据ISBN号获取图书数据。如果查找到了图书数据，那么它会执行这个with代码块，不产生报错并且一份积分将会从客户账户中扣除。然后，我们将向客户返回这一数据。如果未查找到图书数据，那么我们抛出异常来释放所预留的积分。

#### 扩展知识…

在示例中，我们仅处理了对一本书的数据的请求，而获取单份积分还很简单，但在获取多份积分时就会变得更复杂。一个复杂的价格结构会导致一些极端状况。我们来通过下面的盒子进行了解这一问题。假定我们想要处理多本书的请求。在用例中，客户请求了10本书的数据，但仅得到了5本书的数据。此处，如果我们完成了这一个区块并且没有遇到任何问题的话，charge()会获取到10份积分，这是错误的，因为我们仅有一定数据的数据。此外，如果我们抛出异常，那么它会释放所有这10份积分并对客户显示未找到图书信息。解决这一问题 ，Odoo在with区块中提交了交易的对象。在某些用例中，无法提供完整服务。举个例子，用户请求了10本书的数据，但我们仅有5本书的数据。在这种情况下，可以在运行过程中修改实现的金额并获取部分积分。参见以下更进一步的讲解：

```
...
isbn_list = [<assume list of 10 isbn number>]
credits_to_reserve = len(isbn_list)
data_found = []
with iap.charge(request.env, service_key, account_token,  credits_to_reserve) as transection:  
    for isbn in isbn_list:    
        data = request.env['books.info']._books_data_by_isbn(isbn)    
        if data['status'] == 'found':      
            data_found.appned(data)  
        transection.credit = len(data_found)
return data_found
```

在以上代码块中，我们更新了金额来即时地根据transection.credit获取积分，这种方式我们为查找到的图书数据收取金额。

#### 其它内容

* IAP不仅限于Odoo框架。你可以开在其它平台或框架中开发一个服务提供者模块。仅需确保它能够处理[JSON-RPC2](https://www.jsonrpc.org/specification)请求。
* 如果想要在其它平台开发一个服务提供者，你还需要手动使用IAP端点来管理交易。你将需要通过请求IAP端点来授权并获取积分。可以参见https://www.odoo.com/documentation/12.0/webservices/iap.html#json-rpc2-transaction-api获取更多有关端点的知识。

### 创建一个IAP客户端模块

在前一节中，我们创建了IAP服务模块。下面，我们将创建一个IAP客户端模块来完成IAP服务流。

#### 准备工作…

我们需要使用[第四章 创建Odoo插件模块](https://alanhou.org/creating-odoo-add-on-modules/)中的my\_library模块。我们将在图书的表单视图中添加一个按钮，点击该按钮将创建一个对IAP服务的请求并获取图书数据。

对于IAP服务流，客户向服务提供者发出请求。此处要发出一个客户请求，我们需要针对IAP服务单独运行一个服务。如果你想要在同一台机器上进行测试，可以使用不同的端口和不同数据库来运行服务实例，如下：

```
./odoo-bin -c server-config -d service_db --db-filter=^service_db$ --http-port=8070

```

这会在8070端口上运行Odoo服务。确保在这个数据库中安装了服务模块并且已添加IAP服务密钥。注意本节假定你的IAP服务运行在http://localhost:8070这一链接上。

#### 如何实现…

我们将新建一个iap\_isbn\_client模块。这个模块将继承my\_library模块并在图书的表单视图中添加按钮。点击按钮将向运行于8090端口的IAP服务发送一个请求。IAP服务将获取积分并返回所请求图书的信息。我们将在书本的记录中写入这一信息。按照如下步骤来完成IAP客户端模块：

1.  新建一个iap\_isbn\_client模块并添加\_\_init\_\_.py:

    ```
    from . import models
    ```
2.  将给定的内容添加到\_\_manifest\_\_.py中：\


    ```
    {  
        'name': "Books ISBN",  
        'summary': "Get Books Data based on ISBN",  
        'website': "http://www.example.com",  
        'category': 'Uncategorized',  
        'version': '12.0.1',  
        'depends': ['iap', 'my_library'],  
        'data': [    'views/library_books_views.xml',  ]
    }
    ```
3.  添加models/library\_book.py并通过继承library.book模型来添加一些字段：\


    ```
    from odoo import models, fields, api
    from odoo.exceptions import UserError
    from odoo.addons.iap import jsonrpc  
    class LibraryBook(models.Model):  _
        inherit = 'library.book'    
        cover_image = fields.Binary('Books Cover')  
        isbn = fields.Char('ISBN')
    ```
4.  在相同的模型中添加fetch\_book\_data()方法。它会在点击按钮时进行调用：\


    ```
    def fetch_book_data(self):  
        self.ensure_one()  
        if not self.isbn:    
            raise UserError("Please add ISBN number")    
        user_token = self.env['iap.account'].get('book_isbn')  
        params = {    
            'account_token': user_token.account_token,    
            'isbn_number': self.isbn  
        }  
        service_endpoint = 'http://localhost:8070'  
        result = jsonrpc(service_endpoint + '/get_book_data', params=params)  
        if result.get('status') == 'found':    
            self.write(self.process_result(result['data']))  
        return True
    ```
5.  添加process\_result()方法来处理IAP服务的响应：\


    ```
    @api.modeldef 
    process_result(self, result):  
        authors = []  
        existing_author_ids = []  
        for author_name in result['authors']:    
        author = self.env['res.partner'].search([('name','=',author_name)], limit=1)    
        if author:      
            existing_author_ids.append(author.id)    
        else:      
            authors.append((0, 0, {'name': author_name}))  
        if existing_author_ids:    
            authors.append((6, 0, existing_author_ids))  
        return {    
            'author_ids': authors,    
            'name': result.get('name'),    
            'isbn': result.get('isbn'),    
            'cover_image': result.get('cover_image'),    
            'date_release': result.get('date_release'),  
        }
    ```
6.  添加views/library\_books\_views.xml，并通过继承图书的表单视图来添加按钮和字段：\


    ```
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>    
        <record id="library_book_view_form_inh" model="ir.ui.view">        
            <field name="name">Library Book Form</field>        
            <field name="model">library.book</field>        
            <field name="inherit_id" ref="my_library.library_book_view_form"/>        
            <field name="arch" type="xml">            
                <xpath expr="//group" position="before">                
                    <header>                    
                        <button name="fetch_book_data" string="Fetch Book Data" type="object"/>                
                    </header>        
                </xpath>            
                <field name="date_release" position="after">                
                    <field name="isbn"/>                
                    <field name="cover_image" widget="image" class="oe_avatar"/>            
                </field>        
            </field>    
        </record>
    </odoo>
    ```

安装iap\_isbn\_client模块。这会在图书表单中添加Fetch Book Data按钮。然后添加一个有效ISBN号（如1788392019）并点击按钮。这会发送请求并从服务获取数据。如果你是首次进行IAP服务调用，那么你的Odoo实例不会有关联账号的信息，因此Odoo会弹出购买积分的页面如下：

TODO

在点击Buy credits at Odoo按钮时，会重定向到IAP服务页面，这里你会看到有关可选购买包的信息。就本节而言，你会看到本章_在Odoo中注册一个IAP服务_一节中在注册我们的服务时定义的包。参见如下截图，这是一个购买包的列表：

TODO

因我们在使用沙盒端点，你可以无需支付就购买包。然后，你可以从图书的表单视图请求图书信息。

#### 运行原理…

在服务模块中我们创建了一个路由/get\_book\_data。这个路由用于处理客户的IAP请求。因此通过这个客户端模块，我们将对该路由做一个JSON-RPC请求。IAP请求将获取积分并取得图书数据。所幸IAP为jsonrpc请求提供一个封装，我们将进行使用。

my\_library模块的library.book模型没有ISBN和cover\_image字段，因此我们通过继承在library.book模型中拥有了一些额外字段。参见[第五章 应用模型](https://alanhou.org/application-models/)中的_使用继承向模型添加功能_一节。我们通过继承添加了字段，因为不希望在iap\_isbn\_client未安装时使用这些字段。

为进行请求，我们通过继承向图书表单视图添加了一个按钮。点击按钮会触发fetch\_book\_data()方法，在该方法中，我们向服务端点做出了jsonrpc请求。通过这个请求，我们传递了两个参数：客户账号令牌和图书数据的ISBN号。

你可以通过iap.account模型的get() 方法获取客户账号。令牌的生成是自动的。你只需通过服务的名称调用get()方法。在本例中，服务名为book\_isbn。这将返回客户IAP账号的记录集，并且你可以获取到客户令牌account\_token字段。

我们做出了jsonrpc请求来获取图书信息。如果客户没有足够的积分服务模块会生成InsufficientCreditError。jsonrpc会自动处理这个异常，它会向客户显示一个弹窗来购买积分。弹窗中有一个客户可以购买服务方案的页面链接。因为我们使用的是沙盒，你可以无需支付而获取任意包。但是，在生产环境中，客户需要支付来获取服务。

点击按钮时，如果一些顺利，客户有足够的积分而我们的数据库也有所请求ISBN的数据，会从客户账户中扣除金额并且jsonrpc会返回图书数据。然后我们只需将结果传递给process\_result()方法并向图书记录写入数据。

#### 扩展知识…

如果你想了解服务的剩余金额，可以通过仪表盘所提供的链接进行查看：

TODO

### 在账户缺少积分时显示报价

如果在消费完所有的积分后进行IAP服务请求，那么服务模块会产生InsufficientCreditError，客户端模块将自动处理这个错误并显示弹窗。无论何时你的IAP账户积分消耗完，Odoo都会显示像下面这个购买更多积分的弹窗：

TODO

默认弹窗太过简单并且没有提供足够的信息。在本节中，我们将学习如何使用美观的模板替换弹窗的内容。

#### 准备工作…

本节我们将使用iap\_isbn\_service模块。付款模板由IAP服务提供者模块提供，因此它可以任何时间无需更新客户端模块而进行修改。

#### 如何实现…

按照如下步骤来添加自定义收款模板：

1.  在views/templates.xml中使用服务信息添加模板：\


    ```
    <odoo>  
        <template id="no_credit_info" name="No credit info">    
            <section class="jumbotron text-center bg-primary">      
                <div class="container pb32 pt32">        
                    <h1 class="jumbotron-heading">Library ISBN</h1>        
                    <p class="lead text-muted">          Get full book information with cover image just by the ISBN number.        </p>        
                    <span class="badge badge-warning" style="fontsize: 30px;">          20% Off        </span>      
                </div>
            </section>    
            <div class="container">      
                <div class="row">        
                    <div class="col">          
                        <div class="card mb-3">            
                            <div class="card-header">              
                                <i class="fa fa-database"/> 
                                Large books database            
                            </div>            
                            <div class="card-body">              
                                <p class="card-text">                
                                    We have largest book databse. It contains more than 2500000+ books.              
                                </p>            
                            </div>          
                        </div>        
                    </div>        
                    <div class="col">          
                        <div class="card mb-3">            
                            <div class="card-header">            
                                <i class="fa fa-image"/>            
                                With cover image            
                            </div>            
                            <div class="card-body">              
                                <p class="card-text">             
                                    More then 95% of our books having high quality book cover images.              
                                </p>            
                            </div>          
                        </div>        
                    </div>      
                </div>    
            </div>  
        </template>
    </odoo>
    ```
2.  在\_\_manifest\_\_.py中添加一个模板：\


    ```
    ...  
        'data': [    
            'security/ir.model.access.csv',    
            'views/book_info_views.xml',    
            'data/books_data.xml',    
            'views/res_config_settings.xml',    
            'views/templates.xml'  
        ]
    ...
    ```
3.  添加一个iap.charge at controllers/main.py的模板引用：\


    ```
    ...
        with iap.charge(request.env, service_key, account_token,credits_to_reserve,credit_template='iap_isbn_service.no_credit_info'):  
            data = request.env['book.info'].sudo()._books_data_by_isbn(isbn_number)  
            if data['status'] == 'not found':    
                raise Exception('Book not found')
    ```

更新模块来应用修改。在更新之后，会在客户的所有积分都消费完时看到付费弹窗：

TODO

#### 运行原理…

为在客户端显示更好看的弹窗，我们需要创建一个QWeb模板。在第1步中，我们创建了QWeb模板no\_credit\_info。这通过简单的bootstrap内容来获取。注意它仅包含静态HTML内容。在下一步中，我们向应用声明添加了模板文件。

在设计模板之后，你需要向iap.charge()方法传递XML模板引用。这可通过可选的credit\_template参数进行传递。在第3步中，我们传递一个对charge方法的模板引用。在传递了模板之后，如果抛出了InsufficientCreditError，那么模板会带着错误消息传递给客户。在客户端，如果错误页面通过模板体接收，那么这个自定义模板会在一个代替默认弹窗的弹窗中显示。

#### 扩展知识…

在模板中没有图片，但如果我想要在模板中使用图片，需要格外小心。原因是在这里不能像平常那样使用图片的绝对路径URL。因为服务模块运行在单独的服务器上，弹窗不会显示图片。要解决这一问题，你需要传递带有域名的完整图片URL，因为这个模板会在客户端屏幕上显示。例如，如果服务域名为http://localhost:8070，那么你需要使用下面这样的图片写法：

\<image src=”[http://localhost:8070/module\_name/static/img/image.png](http://localhost:8070/module\_name/static/img/image.png) “/>
