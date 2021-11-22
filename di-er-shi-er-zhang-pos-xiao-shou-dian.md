# 第二十二章 POS（销售点）

本章中，我们将讲解如下内容：

* 添加自定义JavaScript/SCSS文件
* 在键盘上添加动作按钮
* 做RPC调用
* 修改POS界面UI
* 修改已有业务逻辑
* 修改客户收据

### 引言

截至目前在本书中我们探讨了两种基本代码。一种是用于创建视图、动作、菜单、向导等的后端基本代码。第二种是用于创建网页、控制器、小插件等的前端基本代码。本章中我们将探讨第三种基础代码，用于销售点（POS）。你可能会好奇为什么 POST 需要一种不同的基础代码。这是因为它使用不同的架构，因此它可以离线运行。本章中我们将学习如何修改POS。

> ℹ️POS应用大多使用JavaScript编写。本章中假定你已有JavaScript和jQuery的基础知识。本章还使用到客户端QWeb模板和组件，因此如果不了解这些的话，请学习[第十六章 网页客户端开发](https://alanhou.org/web-client-development/)。

整章中我们使用一个名为pos\_demo的插件模块。这个pos\_demo模块会依赖于point\_of\_sale，因为我们将要对POS应用进行自定义。为快速上手本章，我们准备了一个pos\_demo的初始模块，可通过本书GitHub仓库的Chapter22/r0\_initial\_module/pos\_demo目录进行获取。

### 技术准备

本章的技术要求包括在线的Odoo平台。

本章中使用的所有代码通过GitHub仓库进行下载：https://github.com/PacktPublishing/Odoo-12-Development-Cookbook-Third-Edition/tree/master/Chapter22。

观看如下视频来查看代码实时操作：

### 添加自定义JavaScript/SCSS文件

销售点应用使用不同的资源包来管理JavaScript和样式文件。本节中，我们将学习如何向 POS 资源包中添加SCSS和JavaScript文件。

#### 准备工作

本节中，我们将把SCSS样式文件和JavaScript文件加载到POS应用中。

#### 如何实现…

按照如下步骤来在销售点应用中加载资源：

1.  新增SCSS文件/pos\_demo/static/src/scss/pos\_demo.scss 并加入如下代码;\


    ```
    .pos .pos-content {  
        .price-tag {    
            background: #00abcd;    
            width: 100%;    
            right: 0;    
            left: 0;    
            top:0;  
        }
    }
    ```
2.  新增JavaScript文件/pos\_demo/static/src/js/pos\_demo.js并加入如下代码：\


    ```
    console.log('Point of Sale JavaScript loaded');
    ```
3.  将这些JavaScript和SCSS文件注册到point\_of\_sale资源中：\


    ```
    <?xml version="1.0" encoding="utf-8"?>
    <odoo>  
        <data>    
            <template id="assets" inherit_id="point_of_sale.assets">      
                <xpath expr="." position="inside">        
                    <script type="text/JavaScript" src="/pos_demo/static/src/js/pos_demo.js"/>        
                    <link rel="stylesheet" href="/pos_demo/static/src/scss/pos_demo.scss"/>      
                </xpath>    
            </template>  
        </data>
    </odoo>
    ```

安装pos\_demo模块。要查看实时的修改，可通过Point of Sale > Dashboard菜单打开point-of-sale 会话。

#### 运行原理…

本节中，我们将一个JavaScript文件和一个SCSS文件加载到销售点应用中。第1步中，我们修改了产品卡片价格标签的背景色和边框半径。在安装了pos\_demo模块之后，你将能看到价格标签的更改：

[![【翻译中】Odoo 14开发者指南第二十二章 POS（销售点）](https://alanhou.org/homepage/wp-content/uploads/2019/05/201908210238412.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/201908210238412.png)

在第2步中，我们添加了JavaScript文件。其中我们向控制台添加了一条日志。为查看该消息，你需要打开浏览器的开发者工具。在控制台标签中，可以看到如下的日志。这表明你的JavaScript文件已成功加载。现在，我们仅在JavaScript文件中添加了日志，但在接下来的小节中，我们会添加更多代码：

[![【翻译中】Odoo 14开发者指南第二十二章 POS（销售点）](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102390967.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102390967.png)

在第3步中，我们向销售点资源中添加了前述的JavaScript和SCSS文件。销售点资源的外部ID为point\_of\_sale.assets.。此处仅有外部ID不同，其它部分和常规资源相同。如果不知道Odoo中资源如何运作，请参见[第十五章 CMS网站开发](https://alanhou.org/cms-website-development/)中的_管理静态资源_一节。

#### 扩展知识…

Odoo还有针对餐厅 POS 解决方案的插件模块。注意这个销售点餐厅模块仅是对POS应用的扩展。如果想要在餐厅模块中进行自定义，你将需要在相同的point\_of\_sale.assets资源包中添加JavaScript和SCSS文件。

### 在键盘上添加动作按钮

如我们在前一节所讨论，POS应用的设计可在离线状况下使用。因此POS应用的代码结构和剩下的Odoo应用都不同。POS应用的代码基大多使用JavaScript编写并且提供了自定义的不同工具。本节中，我们将使用一个这类工具来在键盘面板的顶部创建一个动作按钮。

#### 准备工作

本节我们将继续使用_添加自定义JavaScript/SCSS文件_一节中所创建的pos\_demo模块。我们将在键盘面板的顶部添加一个按钮。该按钮是对订单应用折扣的一个快捷方式。

#### 如何实现…

按照如下步骤来在销售点应用的键盘面板中添加一个5%折扣的动作按钮：

1.  在/static/src/js/pos\_demo.js文件中添加如下代码，它会定义一个动作按钮：\


    ```
    odoo.define('pos_demo.custom', function (require) {  
        "use strict";   
        var screens = require('point_of_sale.screens');  
        var discount_button = screens.ActionButtonWidget.extend({    
            template: 'BtnDiscount',    
            button_click: function () {      
                var order = this.pos.get_order();      
                if (order.selected_orderline) {        
                    order.selected_orderline.set_discount(5);      
                }    
            }  
        });   
        screens.define_action_button({    
            'name': 'discount_btn',    
            'widget': discount_button  
        }); 
    });
    ```
2.  在/static/src/xml/pos\_demo.xml文件中为该按钮添加QWeb模板：\


    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <templates id="template" xml:space="preserve">   
        <t t-name="BtnDiscount">    
            <div class='control-button'>      
                <i class='fa fa-gift' /> 
                5% discount    
            </div>  
        </t>  
    </templates>
    ```
3.  在声明文件中注册QWeb模板如下：\


    ```
    'qweb': [    'static/src/xml/pos_demo.xml'  ]
    ```

更新pos\_demo模块来应用修改。然后，你就会在键盘的顶部看到一个5%折扣按钮：

[![【翻译中】Odoo 14开发者指南第二十二章 POS（销售点）](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102394131.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102394131.png)

点击该按钮时，会对所选的订单应用折扣。

#### 运行原理…

要在销售点应用中创建动作按钮，需要继承ActionButtonWidget。ActionButtonWidget在point\_of\_sale.screens中定义，因此在你的代码中使用它，需要进行导入。在第1步中，我们使用require(‘point\_of\_sale.screens’)导入了界面并通过继承ActionButtonWidget创建了discount\_button。如果想要学习Odoo JavaScript中require的运行机制，请参见[第十五章 CMS网站开发](https://alanhou.org/cms-website-development/)中的_为网站扩展CSS和JavaScript_一节。

在ActionButtonWidget中，你将需要传递模板名及点击事件处理函数。查看第1步中的代码，我们通过button\_click键添加了事件处理函数。点击事件处理函数的名称必须为button\_click。它会自动为你绑定按钮点击事件并在用户点击按钮时调用该函数。在button\_click的函数体内，我们借助set\_discount()函数对订单应用了5%的折扣。我们使用了set\_discount() 函数来应用折扣。销售点应用中还有很多其它的方法如 get\_discount()。查看/point\_of\_sale/static/src/js/models.js文件来了解更多有关其它方法的内容。

接下来，你将需要向系统注册这个动作按钮。可惟通过调用define\_action\_button函数来实现，它接收两个参数：按钮名称和discount\_btn组件的引用。注意这个名称必须唯一，否则你的新按钮会替换掉已有按钮。define\_action\_button为你处理一切事务，它会渲染该按钮，在键盘面板添加按钮并处理点击事件。

在2步中，我们为按钮创建了QWeb模板。语法非常简单，我们只需要创建一个带有control-button类的\<div>元素。第3步中我们在模块声明中注册了这个模板文件。

#### 扩展知识…

define\_action\_button方法支持另一个可选参数条件。这个参数用于根据某一条件隐藏/显示该按钮。这个参数的值是一个返回布尔值的函数。根据所返回的值，POS 系统会隐藏或显示这个按钮。参见下例来获取更多信息：

```
screens.define_action_button({  
    'name': 'reprint',  
    'widget': ReprintButton,  
    'condition': function(){    
        return this.pos.config.print_via_proxy;  
    }
});
```

### 做RPC调用

虽然销售点应用可离线使用，它仍然可以对服务端进行RPC调用。RPC调用可用于任意操作，你可以使用它来进行增删改查操作或对服务端执行某个动作。本节中，我们将进行RPC调用来获取客户最近的5个订单的信息。

#### 准备工作

本节中，我们将使用_在键盘上添加动作按钮_一节中所创建的pos\_demo模块。我们将定义动作按钮。在用户点击这个动作按钮时，我们会进行RPC调用来获取订单信息并在弹窗中展示。

#### 如何实现…

按照如下步骤来显示所选择客户的最近的5张订单：

1.  通过如下代码在 /static/src/js/pos\_demo.js文件中导入RPC工具：\


    ```
    var rpc = require('web.rpc');
    ```
2.  在/static/src/js/pos\_demo.js文件中添加如下代码，这会新增一个动作按钮来在用户点击该按钮时获取并显示最近5张订单的信息：\


    ```
    var lastOrders = screens.ActionButtonWidget.extend({  
        template: 'LastOrders',  // Add step 3 and 4 here
    }); 
    screens.define_action_button({  
        'name': 'last_orders',  
        'widget': lastOrders
    });
    ```
3.  在 lastOrders组件中添加button\_click函数来管理按钮点击：\


    ```
    button_click: function () {  
        var self = this;  
        var order = this.pos.get_order();  
        if (order.attributes.client) {    
            var domain = [['partner_id', '=',  order.attributes.client.id]];    
            rpc.query({      
                model: 'pos.order',      
                method: 'search_read',      
                args: [domain, ['name', 'amount_total', 'cu']],        
                kwargs: { limit: 5 },    
            }).then(function (orders) {    
                if(orders.length > 0){      
                    var order_list = _.map(orders, function (o) {      
                        return { 'label': _.str.sprintf("%s -        TOTAL: %s", o.name, o.amount_total) };      
                    });      
                    self.show_order_list(order_list);    
                } else {      
                    self.show_error('No previous order found for      the customer');    
                }    
            });  
        } else {    
            this.show_error('Please select the customer');  
        }},
    ```
4.  添加方法来显示订单列表和错误：\


    ```
    show_order_list: function(list) {  
        this.gui.show_popup('selection', {    
            'title': 'Please select a reward',    
            'list': list,    
            'confirm': function (reward) {      
                order.apply_reward(reward);    
            },  
        });
    },
    show_error: function (message){  
        this.gui.show_popup('error', {    
            title: "Warning",   
            body: message,  
        });
    }
    ```
5.  在/static/src/xml/pos\_demo.xml 文件中为按钮添加QWeb模板：\


    ```
    <t t-name="LastOrders">    
        <div class='control-button'>        
            <i class='fa fa-shopping-cart'/>        
            Last orders    
        </div>
    </t>
    ```

更新pos\_demo模块来应用修改。然后，你将能在键盘面板上方看到一个 Last Orders 按钮。在点击这个按钮时，会在弹窗中显示订单信息：

[![【翻译中】Odoo 14开发者指南第二十二章 POS（销售点）](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102402369.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102402369.png)

#### 运行原理…

RPC工具可以在web.rpc命名空间中使用。在第1步中，我们通过require(‘web.rpc’)导入了RPC工具。

在第2步中，我们定义一个动作按钮来获取并显示选中客户的订单信息。在深入技术细节之前，让我们来理解通过这个动作按钮我们希望完成的操作。在进行点击后，我们希望显示所选客户的最近5张订单的信息。有一些情况是未选中客户或者客户此前没有订单。在这些情况下，我们需要在弹窗中显示相应的信息。

让我们回到第2步的技术细节。这里我们创建并注册了这个动作按钮。如果想要了解动作按钮更多的相关知识，请参见_在键盘上添加动作按钮_一节。在第3步中，我们添加了点击处理函数。在点击动作按钮时，会调用点击处理函数。这个函数会向服务端发送RPC调用来获取订单信息。我们使用了rpc.query()方法来进行RPC调用。以下是你可以在rpc.query() 方法中传递的参数列表：

* model：你希望对其执行操作的模型名称。
* method：你所希望调用的方法名称。
* args：方法所能接受的必填位置参数列表。
* kwargs：方法所接收的位置参数字典。

本节中，我们使用了search\_read方法来通过RPC获取数据。我们传递了客户域来过滤订单。我们还传递了limit关键字参数来仅获取5个订单。 rpc.query()是一个异步方法，返回JQuery延时对象，因此要处理该结果，你需要使用then()方法。

在第4步中，我们在lastOrders按钮中创建了show\_order\_list和show\_error方法。这些会用于在弹窗中显示订单信息和警告。我们使用了gui 的show\_popup() 方法。如果你希望了解更多有关gui方法的知识，可以查看addons/point\_of\_sale/static/src/js/gui.js文件。

在第5步中，我们为动作按钮添加了QWeb模板。POS 应用会渲染这个模板来显示动作按钮。

&#x20;

> ℹ️RPC调用不可在离线模式下使用。如果你有良好的因特网连接并且不常使用离线模式，那么可以使用RPC。虽然Odoo的销售点应用可以离线使用，有些操作，如创建或更新客户，需要有网络连接，因为这些功能在内部使用了RPC调用。

### 修改POS界面UI

POS 应用的用户界面使用QWeb模板编写。这些模板是客户端QWeb模板。因此你不能使用XPath来修改元素。客户端模板使用另一种方法来修改已有的UI元素。本节中，我们将学习如何在 POS应用中修改UI元素。

#### 准备工作

本节我们将使用_做RPC调用_一节中所创建的pos\_demo模块。我们会修改产品卡片的UI并显示每个产品的边际利润。

#### 如何实现…

按照如下步骤来在产品卡片中显示边际利润：

1.  在/static/src/js/pos\_demo.js文件中添加如下代码来获取针对产品的实际价格的额外字段：\


    ```
    var pos_model = require('point_of_sale.models');
    pos_model.load_fields("product.product", "standard_price");
    ```
2.  在/static/src/xml/pos\_demo.xml中添加如下代码来显示一个边际利润产品卡片：\


    ```
    <t t-extend="Product">  
        <t t-jquery=".price-tag" t-operation="after">    
            <span t-if="product.standard_price" class="sale_margin">      
                <t t-set="margin" t-value="product.list_price - product.standard_price"/>      
                <t t-esc="widget.format_currency(margin)"/>    
            </span>  
        </t>
    </t>
    ```
3.  为利润文本添加如下的样式：\


    ```
    .sale_margin {  
        top: 21px;  
        line-height: 15px;  
        right: 2px;  
        background: #CDDC39;  
        position: absolute;  
        border-radius: 10px;  
        padding: 0px 5px;
    }
    ```

更新pos\_demo模块来应用修改。然后，你将能够在产品卡片中看到边际利润：

[![【翻译中】Odoo 14开发者指南第二十二章 POS（销售点）](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102405249.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102405249.png)

#### 运行原理…

本节中，我们希望使用standard\_price来作为产品的购买价格。这个字段默认在 POS 应用中并没有加载。在第1步中，我们为product.product模型添加了standard\_price字段。然后中，产品数据会多一个字段：standard\_price。

第2步中，我们扩展了默认的产品卡片模板。你将需要对已有QWeb模板使用t-extend属性。然后你需要使用t-jquery属性来选择你想要执行操作的元素。这非常类似 XPath，它代替XPath它使用的是JQuery选择器。这个选择器必须匹配初始模板中的元素。本例中，我们使用了t-jquery=”.price-tag”。要进行交叉验证，你可以在初始产品模板中检查带有class=”price-tag”的元素。这个初始产品模板位于point\_of\_sale/static/src/xml/pos.xml文件中。

通过JQuery选择器，你还需要为所希望执行的操作传递参数。本节中我们使用了t-operation=”after”，它表示你的自定义会被加到所选元素之后。以下是能够使用的操作列表：

* append：将自定义内容添加为所选元素的最后一个子元素。
* prepend：将自定义内容添加为所选元素的第一个子元素。
* before：在所选元素之前添加自定义内容。
* after：在所选元素之后添加自定义内容。
* replace：将所选元素替换为自定义内容。
* inner：将所选元素的内部内容替换为自定义内容。
* attribute：修改所选元素的属性。

在第3步中，我们添加了样式来修改利润元素的位置。这会为利润元素添加背景色并将其放到价格胶囊的下方。

### 修改已有业务逻辑

在前一小节中，我们学习了如何通过RPC获取数据以及如何修改 POS 应用的 UI 界面。本节中我们将学习如何修改或继承已有的业务逻辑。

#### 准备工作

本节我们将继续使用_修改POS界面UI_一节中所创建的pos\_demo模块，其中我们获取了产品的购买价格并显示产品利润。现在，本节中我们将在销售低于产品利润的产品时向用户发送警告。

#### 如何实现…

在部分 POS 应用的业务逻辑使用JavaScript编写，因此我们仅需对其进行修改来实现本节的目标。在/static/src/js/pos\_demo.js中添加如下代码来在用户销售低于购买价格的产品时发送警告。

```
var screens = require('point_of_sale.screens'); 
screens.OrderWidget.include({  
    set_value: function (val) {    
        this._super(val);    
        var orderline = this.pos.get_order().get_selected_orderline();    
        var standard_price = orderline.product.standard_price;    
        if (orderline && standard_price) {      
            var line_price = orderline.get_base_price();      
            if (line_price < orderline.quantity * standard_price) {        
                this.gui.show_popup('alert', {          
                    title: "Warning",          
                    body: "Product price is set below product actual cost",        
                });      
            }    
        }  
    }
});
```

更新pos\_demo模块来应用修改。在更新后，对订单添加折扣，让产品的价格低于购买价格。会弹出 一个警告的窗口：

[![【翻译中】Odoo 14开发者指南第二十二章 POS（销售点）](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102411936.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102411936.png)

#### 运行原理…

为修改现有业务，我们需要使用include() 函数。这个函数将在内部使用[猴子补丁](https://en.wikipedia.org/wiki/Monkey\_patch)并替换原来的对象属性为新的属性。如果你想要访问初始方法，可以使用this.\_super属性。

OrderWidget的set\_value()函数用于通过键盘更新产品价格。相同的函数也用于应用折扣。因此在本节中，我们替换set\_value()方法为我们自己的实现。我们不希望修改默认的实现，因此首先调用了\_super()，它会调用默认的 set\_value()方法。在进行父级调用后，我们添加了逻辑来在用户将产品价格设置为地采购价格时显示警告。我们使用了gui的show\_popup()方法来显示弹窗。

我们将自己的业务逻辑放到默认的实现 (\_super)调用之后。如果想要在默认实现之前编写业务逻辑，可以通过将\_super调用放到函数的最后来实现。

> ℹ️使用include()时如不谨慎会破坏一些事物。如果方法从多个文件中继承，你必须调用 super方法，否则它会在随后的继承中跳过该逻辑。这有时会导致崩溃的内部数据状态。

### 修改客户收据

在你自定义销售点应用时，从客户获取到的常见请求是修改客户收据。本节我们将学习如何修改客户收据。

#### 准备工作

本节我们将继续使用_修改已有业务逻辑_一节中所创建的pos\_demo模块。我们将在 POS 收据中添加一行来显示用户在订单中节省了多少钱。

#### 如何实现…

按照如下步骤来在 POS 中修改客户收据：

1.  在 /static/src/js/pos\_demo.js文件中添加如下代码。这会在收据环境添加额外的数据：\


    ```
    var screens = require('point_of_sale.screens'); 
    screens.ReceiptScreenWidget.include({  
        get_receipt_render_env: function () {    
            var receipt_env = this._super();    
            var order = this.pos.get_order();    
            var saved = 0;    
            _.each(order.orderlines.models, function (line) {      
                saved += ((line.product.list_price -  line.get_base_price()) * line.quantity);    
            });    
            receipt_env['saved_amount'] = saved;    
            return receipt_env;  
        }
    });
    ```
2.  在/static/src/xml/pos\_demo.xml中添加如下代码。这会继承默认的收据模板并添加我们的自定义内容：\


    ```
    <t t-extend="PosTicket">  
        <t t-jquery=".receipt-change" t-operation="after">    
            <t t-if="saved_amount">      
                <br/>      
                <div class="pos-center-align">        
                    <t t-esc="widget.format_currency(saved_amount)"/>        
                    you saved in this order      
                </div>    
            </t> 
        </t>
    </t>
    ```

更新pos\_demo模块来应用修改。然后，添加一个带有折扣的产品并查看收据，会在收据上看到多了一行：

[![【翻译中】Odoo 14开发者指南第二十二章 POS（销售点）](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102415113.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102415113.png)

#### 运行原理…

本节中并没有什么新的内容。我们只是通过使用前面小节的模块更新了收据。在第1步中，我们重载了get\_receipt\_render\_env() 函数来向收据环境发送了更多信息。我们对比了产品的基准价格和收据上的价格来计算为客户节约了多少现金。我们通过saved\_amount键将这一数据发送到收据环境中。如果你现要了解方法重载的更多内容，可以参见_修改已有业务逻辑_一节。

在第2步中，我们修改了收据上默认的QWeb模板。实际收据的模板名为PosTicket，因此我们使用了它作为t-extend 属性的值。在第1步中我们已经修改收据所需的信息。在QWeb模板中，我们在saved\_amount键中获取到了节省的金额，因此我们在收据的最后又添加了一个\<div>元素。这会在收据上打印出所节省的金额。如果想要学习更多有关重载的知识，请参见_修改POS界面UI_一节。

#### 扩展知识…

本节中展示 的技术会更新通过 POS 收据界面打印的收据。如果你使用POS Box或 IOT盒子，这一技术不会更新所打印的收据。它使用了XML格式来向ESC/POS兼容的收据打印机打印收据。要修改这一收据，你将需要继承XmlReceipt模板而非PosTicket。参见下例，这会更新通过POS Box或IOT盒子所打印的收据：

```
<t t-extend="XmlReceipt">    
    <t t-jquery='.after-footer' t-operation='after'>        
        <div>            
            <t t-esc="saved_amount"/>            
            you saved in this order        
        </div>    
    </t>
</t>
```

Odoo使用XML-ESC/POS Python 库来生成收据。参见https://github.com/fvdsn/py-xml-escpos来了解所支持的XML结构。
