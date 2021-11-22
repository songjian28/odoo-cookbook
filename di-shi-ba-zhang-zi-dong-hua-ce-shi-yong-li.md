# 第十八章 自动化测试用例



在开发大型应用时，测试用例是提升模块可靠性的良好实践。这会让你的模块更为健壮。每年Odoo发布新的软件版本时，自动化测试用例都对应用的回归测试有很大帮助，问题可能是由于版本升级所导致的。所幸Odoo框架自带有不同的自动化测试工具。Odoo包含以下3种主要类型的测试：

* Python测试用例：用于测试Python业务逻辑
* JavaScript QUnit测试：用于测试Odoo中JavaScript 的实现
* 导览：检测Python和JavaScript正常彼此协作的集成测试

本章中，我们将讲解如下小节：

* Python测试用例
* 运行打标签的Python测试用例
* 为客户端测试用例设置Headless Chrome
* 添加客户端QUnit测试用例
* 添加导览测试用例
* 通过UI（用户界面）运行客户端测试用例
* 调试客户端测试用例
* 为失败的测试用例生成视频/截图
* 为测试生成随机数据

### 技术准备

本章中，我们将详细学习这些测试用例。为在同一模块中讲解所有测试用例，我们创建了一个小模块。其Python定义如下：

```
class LibraryBook(models.Model):  
    _name = 'library.book'  
    
    name = fields.Char('Title', required=True)  
    date_release = fields.Date('Release Date')  
    author_ids = fields.Many2many('res.partner', string='Authors')  
    state = fields.Selection([('draft', 'Not Available'),('available', 'Available'),('lost', 'Lost')],'State', default="draft")  
    color = fields.Integer()   
    
    def make_available(self):    
        self.write({'state': 'available'})   
    
    def make_lost(self):    
        self.write({'state': 'lost'})
        
```

这段Python代码将有助于我们为Python业务用例编写测试用例。对于JavaScript端的测试用例，我们从[第十六章 网页客户端开发](https://alanhou.org/web-client-development/)的_创建自定义组件_一节中添加了int\_color组件。

可以从本书的GitHub仓库中获取初始模块，链接为：https://github.com/PacktPublishing/Odoo-12-Development-Cookbook-Third-Edition/tree/master/Chapter18/r0\_initial\_module

### 技术准备

本章的技术要求包含Odoo在线平台。

本章中使用的所有代码可通过GitHub仓库进行下载：https://github.com/PacktPublishing/Odoo-12-Development-Cookbook-Third-Edition/tree/master/Chapter18。

观看如下视频查看实时代码操作：

### Python测试用例

Python测试用例用于检查业务逻辑的正确性。在[第六章 基本服务端部署](https://alanhou.org/basic-server-side-development/)中，我们学习了如何修改已有应用的业务逻辑。这会使其更为重要，因为自定义可能会导致应用功能的崩溃。本章中，我们将编写测试用例来验证业务逻辑以修改图书的状态。

#### 准备工作

我们将使用GitHub仓库 Chapter18/r0\_initial\_module目录中的my\_library模块。

#### 如何实现…

按照如下步骤来在my\_library模块中添加Python测试用例：

1.  新增文件 tests/\_\_init\_\_.py如下：\


    ```
    from . import test_book_state
    ```
2.  添加一个tests/test\_book\_state.py文件，并添加测试用例如下：\


    ```
    from odoo.tests.common import TransactionCase 

    class TestBookState(TransactionCase):   
        def setUp(self, *args, **kwargs):    
            super(TestBookState, self).setUp(*args, **kwargs)    
            self.test_book = self.env['library.book'].create({'name': 'Book 1'})   
            
            def test_button_available(self):    
                """Make available button"""    
                self.test_book.make_available()    
                self.assertEqual(self.test_book.state, 'available',  'Book state should changed to available')   
                
            def test_button_lost(self):    
                """Make lost button"""    
                self.test_book.make_lost()    
                self.assertEqual(self.test_book.state, 'lost',  'Book state should changed to lost')
                
    ```

为运行该测试用例，使用如下命令启动Odoo服务：

```
./odoo-bin -c server.conf -i my_library --test-enable
```

此时，查看服务日志。如果测试用例运行成功的话你会看到如下日志：

```
... 
INFO test odoo.modules.module:odoo.addons.my_library.tests.test_book_state running tests
.... 
INFO test odoo.addons.my_library.tests.test_book_state:test_button_available(odoo.addons.my_library.tests.test_book_state.TestWizard)
... 
INFO test odoo.addons.my_library.tests.test_book_state: ` Make available button
... 
INFO test odoo.addons.my_library.tests.test_book_state:test_button_lost(odoo.addons.my_library.tests.test_book_state.TestWizard)
... 
INFO test odoo.addons.my_library.tests.test_book_state: ` Make lost button
...
INFO test odoo.addons.my_library.tests.test_book_state: Ran 2 test
```

#### 运行原理…

在Odoo中，Python测试用例在模块的tests/目录中添加。Odoo会自动识别这个目录并在该目录中运行测试。

Odoo使用Python单元测试，通过https://docs.python.org/3.5/library/unittest.html 查看Python测试用例。它还提供一些帮助类，使用unittest进行封装。这些类简化开发测试用例的进程。在本例中，我们使用了TransactionCase。TransactionCase在一个不同的事务中运行每个测试用例。一旦测试用例运行成功，事务会自动回滚。

以test\_开头的类方法被视作测试用例。在本节中，我们添加了两个测试用例。它查看修改图书状态的方法。self.assertEqual方法用于查看测试用例是否成功运行。我们在对图书记录执行了操作之后查看了图书的状态。因此 ，如果开发人员犯错而该方法未能按预期修改状态，测试用例会失败。

> ℹ️注意setUp()方法会自动调用我们所运行的每个测试用例，在本节中，我们添加了两个测试用例，因此setUp()会调用两次。按照本节的代码，在测试时仅会展示一条图书记录，因为使用TransactionCase时，每个测试用例会回滚事务。

方法中的文档字符串会在日志中打印。这有助于查看具体测试用例的状态。

#### 扩展知识…

测试组件提供如下额外测试工具类：

* SingleTransactionCase：通过这个类生成的测试用例会将所有用例运行到一个事务中，因此一个测试用例中所做的修改会在另一个测试用例中可用。这样，事务通过第一个测试方法开启，并仅在完成最后一个测试用例时结束。
* SavepointCase：它与SingleTransactionCase相同，但其测试方法在回滚保存点之前运行，而不是将所有测试方法放一个单独的事务中运行。它用于创建大型测试用例，让它们通过仅生成一次数据来实现更快的速度。此处，我们使用 setUpClass()方法来生成初始测试数据。

### 运行打标签的Python测试用例

在我们使用–test-enabled模块运行Odoo服务时，测试用例在安装模块后立即运行。如果你希望在所有模块安装完之后运行测试用例，或者希望仅针对一个模块运行测试用例，tagged()装饰器就是不二之选了。本节中，我们将讲解如何使用这个装饰器来塑造测试用例。

#### 准备工作

本节中我们将继续使用上一节的my\_library模块。我们会修改测试用例的顺序。

#### 如何实现…

在测试用例中添加tagged()装饰器（如下），让其在安装所有模块之后运行：

```
from odoo.tests.common import TransactionCase, tagged 

@tagged('-at_install', 'post_install')
class TestBookState(TransactionCase):
...
```

然后，和之前一样运行测试用例如下：

```
./odoo-bin -c server.conf -i my_library --test-enable
```

现在查看服务日志。此时你会看到如下日志后面有我们的测试用例，这表示我们的测试运行在所有模块安装之后运行了，如下：

```
... 
INFO test odoo.modules.loading: 10 modules loaded in 21.43s, 346 queries
... 
INFO test odoo.modules.loading: Modules loaded
.... 
INFO test odoo.service.server: Starting post tests
```

在这些日志中，第一行显示加载了10个模块。第二行显示所有请求的模块和它们的依赖都成功安装，第三行显示它会运行打了post\_install标签的测试用例。

#### 运行原理…

默认，所有的测试用例使用的标签为standard, at\_install 和当前模块的技术名称（本例中技术名称为my\_library）。因此，如果没有使用tagged()装饰器的话，你的测试用例就会是这三者。

本例中，我们希望在安装好所有模块之后运行测试用例。为此，我们对TestBookState类添加了tagged()装饰器。默认，测试用例有at\_install标签。因为这个标签，你的测试用例会在模块安装之后立即运行；而不会等待所有模块的安装。我们不希望如上，因此在标签函数中添加了 -at\_install来删除at\_install标签。以 -will为前缀的标签会删除该标签。

通过在tagged()函数中添加-at\_install，我们阻止了在模块安装之后的测试用例执行。因此我们没有在其中指定其它标签，测试用例不会运行。

因此，我们添加了一个post\_install标签。这个标签指定测试用例需要在所有模块都完成安装之后运行。

> ℹ️在Odoo 12之前，common.at\_install()和common.post\_install()装饰器用于修改测试用例的执行顺序。在版本12中，淘汰了这些装饰顺，转而使用at\_install和post\_install标签。

如你所见，所有测试用例默认使用标准标签来打标签。Odoo会使用标准标签运行所有这些测试用例。为避免你不想一直运行某个具体测试用例，而仅希望在请求时才运行这个测试用例。这时，你需要通过在tagged()装饰器中添加-standard来删除标准标签，需要像下面这样添加一个自定义标签：

```
@tagged('-standard', 'my_custom_tag')
class TestClass(TransactionCase):
...
```

所有非标准测试用例将使用–test-enable选项进行运行。要运行以上的测试用例，需要使用-test-tags 选项如下（注意这里我们并不需要显式地传递 –test-enable选项）：

```
./odoo-bin -c server.conf -i my_library --test-tags=my_custom_tag
```

#### 扩展知识…

在测试用例的开发过程中，仅对一个模块运行测试用例非常重要。默认，模块的技术名称会添加为标签，因此你可以使用模块的技术名称加–test-tags选项。例如，如果你希望对my\_library模块运行测试用例，可以运行服务如下：

```
./odoo-bin -c server.conf -i my_library --test-tags=my_library
```

### 为客户端测试用例设置Headless Chrome

Odoo 12使用Headless Chrome来执行JavaScript测试用例以及导览测试用例。本节中，我们将安装Headless Chrome及其它包，以运行JavaScript测试用例。

#### 如何实现…

你需要安装Chrome浏览器来启用JavaScript测试用例。对于模块的开发，我们多使用桌面版操作系统。因此，如果你的系统中已安装了Chrome浏览器，那么则无需再进行单独安装。可以使用桌面版Chrome自身来运行客户端测试用例。确保你的Chrome版本高于Chrome 59。Odoo还支持Chromium浏览器。

> ℹ️注意通过Headless Chrome，客户端测试用例在macOS和 Linux可正常运行，但Odoo并不支持Windows系统中的Headless Chrome测试用例。

在生产服务器或服务器系统中运行测试用例会有一些细微的变化。通常服务器系统没有图形化界面，因此安装Chrome浏览器的方式会不同。如果你使用的是Debian系服务器系统，可以使用如下命令来安装一个Chromium浏览器：

```
apt-get install chromium-browser
```

> ℹ️Ubuntu 18.04服务器版本默认没有启用universe仓库。因此在安装chromium-browser时可能会显示安装备选包错误。要解决这一错误，可通过如下命令来启用universe仓库：sudo add-apt-repository universe。

Odoo还对JavaScript测试用例使用WebSockets。为此，Odoo使用websocket-client Python库。使用如下命令来进行安装：

```
pip3 install websocket-client
```

#### 运行原理…

Odoo对JavaScript测试用例使用Headless Chrome。其背后的原因是它在后台运行测试用例，因此它也可以在服务器操作系统中运行。Headless Chrome青睐于在后台运行Chrome浏览器，无需打开图形化浏览器。Odoo在后台打开Chrome标签并开始在其中运行测试用例。它还对JavaScript测试用例使用JQuery QUnit。在接下来的几节中，我们将为我们的自定义JavaScript组件创建QUnit测试用例。

> ℹ️Headless Chrome在Odoo 12中引入。在更早的Odoo版本中，客户端测试用例使用phantomjs。Odoo改用Headless Chrome的原因是它提供了更多的功能，并且phantomjs项目已不再维护。

对于测试用例，Odoo在不同的进行中打开Headless Chrome，因此为查看该进程中测试用例的运行状态，Odoo服务使用了WebSockets。websocket-client Python库用于通过Odoo服务管理WebSockets与Chrome的通讯。

### 客户端QUnit测试用例

在Odoo中创建新字段或视图非常之简单。通过短短几行XML，就可以定义一个新视图。然而在底层它使用了大量的JavaScript。在客户端修改/添加新功能相当复杂，并可能会导致某些功能的崩溃。大多数客户端的问题发生时都未被察觉，因为错误仅在控制台中显示。因此，在Odoo中使用QUnit测试用例来查看不同JavaScript组件的正确性。

#### 准备工作

对于本节，我们将继续使用前一节中的my\_library模块。我们将对int\_color组件添加一个QUnit测试用例。

#### 如何实现…

按照如下步骤来为int\_color组件添加JavaScript测试用例：

1.  使用如下代码添加/static/tests/colorpicker\_tests.js：\


    ```
    odoo.define('colorpicker_tests', function (require) {  
        "use strict";   
        var FormView = require('web.FormView');  
        var testUtils = require('web.test_utils');  
        QUnit.module('Color Picker Tests', {  
            beforeEach: function () {    
                this.data = {      
                    book: {        
                        fields: {          
                            name: { string: "Name", type: "char" },          
                            color: { string: "color", type: "integer"},        
                        },        
                        records: [{          
                            id: 1,          
                            name: "Book 1",          
                            color: 1        
                        }, {          
                            id: 2,          
                            name: "Book 2",          
                            color: 3        
                        }]      
                    }     
                };  
            }}, function () {  
                QUnit.test('int_color field test cases', function  (assert) {    
                    assert.expect(2);    
                    var form = testUtils.createView({      
                        View: FormView,      
                        model: 'book',      
                        data: this.data,      
                        arch: '<form string="Books">' +      
                            '<group>' +      
                            '<field name="name"/>' +      
                            '<field name="color" widget="int_color"/>' +      
                            '</group>' +      
                            '</form>',      
                        res_id: 1,      
                        viewOptions: {        
                            mode: 'edit',      
                        },    
                    });    
                    assert.strictEqual(    
                        form.$('.o_int_colorpicker      .o_color_pill').length, 10,      
                        "colorpicker should have 10 pills");    
                        form.$('.o_int_colorpicker .o_color_pill:eq(5)').click();    
                    assert.strictEqual(      
                        form.$('.o_int_colorpicker      .o_color_5').hasClass('active'), true,      
                        "click on pill should make pill active");
                form.destroy();    
            });  
        });
    });
    ```
2.  在/views/template.xml中添加如下代码来在测试套件中注册它：\


    ```
    ...
    <template id="qunit_suite" name="colorpicker test"  inherit_id="web.qunit_suite">  
        <xpath expr="." position="inside">    
            <script type="text/javascript"  src="/my_library/static/tests/colorpicker_tests.js" />    
        </xpath>
    </template>
    ...
    ```

要运行这个测试用例，在Terminal中使用如下命令启动服务：

```
/odoo-bin -c server.conf -i my_library,web --test-enable
```

要检查测试用例是否成功运行，在日志中搜索如下内容：

```
... 
INFO test odoo.addons.web.tests.test_js.WebSuite: console log: "Color Picker Tests" passed 2 tests.
```

#### 运行原理…

Odoo中，JavaScript测试用例在/static/tests/目录中进行添加。第1步中，我们为测试用例添加了colorpicker\_tests.js文件，我们还导入了表单视图和测试工具的引用。导入了web.FormView的原因在于我们为表单视图创建了一个int\_color组件，要测试该组件，我们需要这个表单视图。web.test\_utils将为我们提供我们需要创建JavaScript测试用例的测试工具。如果你不知道导入JavaScript的运行机制，请参见[第十五章 CMS网站开发](https://alanhou.org/cms-website-development/)中的_为网站扩展CSS和JavaScript_一节。

Odoo客户端测试用例使用QUnit框架进行创建，它是针对JavaScript单元测试用例的JQuery框架。参见https://qunitjs.com/来学习更多相关知识。beforeEach函数在运行测试用例之前调用，这有助于初始化测试数据。beforeEach函数的引用由QUnit框架自身提供。

我们在beforeEach函数中初始化了一些数据。让我们来看看这些数据在测试用例中如何使用。客户端测试用例在一个隔离的环境（mock）中运行，它不对数据建立连接，因此对于这些测试用例，我们需要创建测试数据。在内部Odoo创建mock服务来模拟远程过程调用（RPC）调用并使用this.data 属性来作为数据库。因此，在beforeEach我们在this.data 属性中初始化了我们的测试数据。this.data 属性中的键被看作数据表，值包含字段的数据表行的信息。fields键用于定义数据表字段，records键用于数据行。在本例中，我们添加了带有两个属性的book表：name(char)和color(integer)。注意在这里，我们可以使用任意Odoo字段，甚至包含关联字段；例如，{string: “M2o FIeld”, type: “many2one”, relation: ‘partner’}。我们还添加了两条带有records键的图书记录。

接着，我们添加了带有QUnit.test函数的测试用例。函数中的第一个参数是一个描述该测试用例的字符串。第二个参数是我们需要为测试用例所添加代码的函数。这个函数由QUnit框架调用，它传递断言工具作为参数。本例中， 我们在assert.expect函数中传递了所预期测试用例的数量。我们添加了两个测试用例，因此传递了2。

我们希望在可编辑表单视图中添加测试用例int\_color组件，因此我们通过testUtils.createView创建了可编辑表单视图。createView函数接收不同的参数，如下：

* View是你所想要创建的视图的引用。你可以为测试用例创建任意类型的视图，只需在这里传递视图引用即可。
* model是给定视图创建所对应模型的名称。所有模型在this.data属性中列出。我们希望创建针对book模型的社图，因此在本例，我们使用了book来作为模型。
* data是我们将要在视图中使用的记录。
* arch是你所想要创建的视图的定义。因此我们希望测试int\_color组件，并传递了带有该组件的视图定义。注意你只能使用在模型中定义的字段。
* res\_id是所显示记录的记录ID。这个选项仅能用于表单视图。在本例中，表单视图将显示book 1记录的数据，因为我们为res\_id传递了1。
* viewOptions是在视图中传递的参数。这里，你可以传递所有的有效视图参数，如域、上下文、模式等等。

在通过int\_color组件创建了表单视图之后，我们添加了两个测试用例。第一个用于检查用户界面中色块的个数，第二个测试用例用于检查在点击色块后是否正确激活。我们从QUnit框架的断言工具中获得了strictEqual函数。如果前两个参数匹配的话strictEqual传递测试用例。如果不匹配，则测试用例失败。

> 小贴士：针对QUnit测试用例还有一些断言函数，如assert.deepEqual, assert.ok, and assert.notOk。要学习有关QUnit的更多知识，请参见其文档：https://qunitjs. com/。

#### 扩展知识…

Odoo为高级用例提供了更多的工具。其中一个是createAsyncView，它用于异步创建视图。要学习更多的相关知识，深入了解/addons/web/static/tests/helpers/ 目录中的JavaScript文件。

### 添加导览测试用例

你已经了解到了Python和JavaScript测试用例。两者都在隔离的环境中运行，并且它们都不与彼此进行交互。要测试JavaScript和Python之间的集成，我们使用导览测试用例。

#### 准备工作

本节我们将继续使用前一小节的my\_library模块。我们将添加一个导览测试用例来检查book模型的运行流程。

#### 如何实现…

按照如下步骤来为books添加导览测试用例：

1.  添加/static/src/js/my\_library\_tour.js文件，并添加导览如下：\


    ```
    odoo.define('my_library.tour', function (require) {  
        "use strict";   
        var core = require('web.core');  
        var tour = require('web_tour.tour');   
        var _t = core._t;   
        tour.register('library_tour', {    
            url: "/web",  }, [tour.STEPS.SHOW_APPS_MENU_ITEM, {    
                    trigger: '.o_app[data-menuxmlid="my_library.library_base_menu"]',    
                    content: _t('Manage books and authors in<b>Library app</b>.'),    
                    position: 'right'  
                }, 
                {    
                    trigger: '.o_list_button_add',    
                    content: _t("Let's create new book."),    
                    position: 'bottom'  
                }, {    
                    trigger: 'input[name="name"]',    
                    extra_trigger: '.o_form_editable',    
                    content: _t('Set the book title'),    
                    position: 'right',    
                    run: function (actions) {      
                        actions.text('Test Book');    
                    },  
                }, {    
                    trigger: '.o_int_colorpicker',    
                    extra_trigger: '.o_form_editable',    
                    content: _t('Set the book color'),    
                    position: 'right',    
                    run: function () {      
                        this.$anchor.find('.o_color_3').click();    
                    }  
                }, {    
                    trigger: '.o_form_button_save',    
                    content: _t('Save this book record'),    
                    position: 'bottom',  
                }  
        ]);  
    });
    ```
2.  添加/tests/test\_tour.py文件，并通过HttpCase运行导览如下：\


    ```
    from odoo.tests.common import HttpCase, tagged 
    class TestBookUI(HttpCase):  
     
        @tagged('post_install', '-at_install')  
        def test_01_book_tour(self):    
            """Books UI tour test case"""    
            self.browser_js("/web",      
                "odoo.__DEBUG__.services['web_tour.tour'].run('library_tour')",      
                "odoo.__DEBUG__.services['web_tour.tour'].tours.library_tour.ready",      
                login="admin"
            )
    ```

为运行测试用例，使用如下选项启动Odoo服务：

```
./odoo-bin -c server.conf -i my_library --test-enable
```

此时查看服务日志。这里如果测试用例成功运行的话你会看到如下日志：

```
...INFO test odoo.addons.my_library.tests.test_tour.TestBookUI: console log: Tour library_tour succeeded
...INFO test odoo.addons.my_library.tests.test_tour.TestBookUI: console log: ok

```

#### 运行原理…

创建导览测试用例，首先你需要创建用户界面导览。如果想要学习更多有关 UI 导览的知识，参见[第十六章 网页客户端开发](https://alanhou.org/web-client-development/)的_通过导览提升客户引导_一节。

第1步中，我们注册了一个新的带有名称library\_tour的导览。这个导览和[第十六章 网页客户端开发](https://alanhou.org/web-client-development/)的_通过导览提升客户引导_一节中我们所创建的完全相同。

这里，我们有一个额外属性：run。在run函数中，我们需要编写逻辑来执行通常由用户所执行的操作。例如，在导览的第4步中，我们要求用户输入图书标题。

要自动化这个步骤，我们添加了一个run函数来设置title字段中的值。run函数传递动作工具来作为参数。这为执行基础动作提供了一些快捷方式。最重要的如下所示：

* actions.click(element)用于点击给定的元素
* actions.dblclick(element)用于双击给定的元素
* actions.tripleclick(element)用于三击给定的元素
* actions.text(string)用于设置输入值
* action.drag\_and\_drop(to, element)用于拖拽元素
* action.keydown(keyCodes, element)用于对元素触发指定的键盘事件
* action.auto()是默认的动作。在没有向导览步骤传递run函数时，执行的是action.auto()。自动动作大多会点击导览步骤中的触发元素。这里唯一的例外是input元素。如果触发元素是input，导览会在其中设置默认值Test。这也是我们不需要为每个步骤添加run函数的原因。

此外，你可以手动的执行所有的动作，以防默认动作不能满足要求。在接下来的导览步骤中，我们希望为颜色拾取器设置值。注意是我们使用了手动动作，因为默认值在这里无益。因此，我们添加了run方法和基础JQuery代码来点颜色拾取器的第三个色块。这里，你会发现带有this.$anchor属性的触发元素。

默认会向终端用户显示已注册导览来提升引导体验。要将它们作为测试用例运行，你需要在 Headless Chrome中运行它们。进行这一实现需要使用HttpCase Python 测试用例。它会提供browser\_js方法，该方法打开URL并执行第二个参数所传递的命令。可以手动运行导览如下：

```
odoo.__DEBUG__.services['web_tour.tour'].run('library_tour')
```

在示例中，我们传递了导览的名称来作为browser\_js方法的参数。下一个参数用于在执行第一个命令前等待给定对象准备就绪。browser\_js()方法的最后一个参数是用户名。用户名用于新建测试环境，并且所有的测试动作会以用户的名义执行。

### 通过UI（用户界面）运行客户端测试用例

Odoo提供一种通过UI运行客户端测试用例的方式。这有助于通过用户界面查看每个测试用例的状态。

#### 如何实现…

你可以通过UI界面同时运行QUnit测试用例和导览测试用例。无法通过用户界面运行Python测试用例。要通过 UI 查看运行测试用例的选项，你需要启动开发者模式。

**运行QUnit测试用例**

点击调试图标来打开下拉菜单，如下图所示。点击Run JS Tests选项：

TODO

这会打开QUnit套件并且它会逐一运行测试用例，如截图中所示。默认，它仅会显示失败的测试用例。要显示所有已通过测试用例，取消Hide passed tests复选框，如下图所示：

TODO

**通过UI运行导览**

点击调试图标打开下拉菜单，如下图所示，然后点击Start Tour：

TODO

这会打开一个带有已注册导览列表的对话框，如下图所见。点击右侧的播放按钮来运行导览：

TODO

#### 运行原理…

针对QUnit的UI由QUnit框架自身所提供。这里你可以过滤针对模块的测试用例。你甚至可以仅针对一个模块运行测试用例。此处可以看到每个测试用例的进度，并且可以深入到测试用例的每一步。在内部Odoo只是在Headless Chrome中打开了同一个URL。

点击Run tours选项会显示可用导览的列表。通过点击列表中的播放按钮，可以运行导览。注意在导览通过命令行选项运行时，它在回滚的事务中运行，因此通过导览所做的更改在导览成功后会进行回滚。但是，在从用户界面运行导览时，通过导览所做的修改并不会进行回滚并保持在那里，因此使用这个选项时请当心。

### 调试客户端测试用例

开发复杂的客户端用例会很头痛。本节中我们将学习如何在Odoo中调试客户端测试用例。我们不会运行所有的测试用例，而仅仅运行单个测试用例。此外，我们将显示测试用例的用户界面。

#### 准备工作

对于本节，我们将继续使用前一节中的my\_library模块。

#### 如何实现…

按照如下步骤在调试模式下运行测试用例：

1.  打开/static/tests/colorpicker\_tests.js文件，并通过QUnit.only更新QUnit.test测试，如下：\


    ```
    ...
    QUnit.only('int_color field test cases', function (assert) {
    ...
    ```
2.  在createView函数中添加debug参数，如下：\


    ```
    var form = testUtils.createView({  
        View: FormView,  
        model: 'book',  
        data: this.data,  
        arch: '<form string="Books">' +    
            '<group>' +      
            '<field name="name"/>' +      
            '<field name="color" widget="int_color"/>' +    
            '</group>' +  
            '</form>',  
        res_id: 1,  
        viewOptions: {    
            mode: 'edit',  
        },  
        debug:1
    });
    ```

打开开发者模式交通过在顶部菜单中点击调试图标来打开下拉菜单，然后点击Run JS tests。这会打开QUnit套件。

#### 运行原理…

在第1步中，我们使用QUnit.only替换了QUnit.test。这会仅运行这个测试用例。在测试用例的开发过程中，这样会节约很多时间。注意使用QUnit.only会阻止测试用例通过命令行来运行。它仅用于调试或测试目的，交互式且仅能在通过 UI 打开测试用例时运行，因此不要忘记在开发后通过QUnit.test来替换它。

在我们的QUnit测试用例示例中，我们创建了表单视图来测试int\_color组件。如果你通过UI运行QUnit测试用例，会了解到你无法在用户界面中查看所创建的表单视图。通过QUnit套件的UI 界面，你可以查看日志。这让开发一个QUnit用例异常复杂。要解决这一问题，在createView中使用了调试参数。在第2步中，我们在createView函数中添加了debug: true。这会在浏览器中显示测试表单视图。此处，你可以通过浏览器调试器定位到文档对象模型（DOM）元素。

> ℹ️警**告：**在测试用例的最后，我们通过destroy() 方法删除了视图。如果你删除了该视图，那么你就不能看到 UI 界面中的表单视图，因此要想在浏览器中看到它的话，请在开发时删除该行。

### 为失败的测试用例生成视频/截图

Odoo 11使用PhantomJS来在服务端运行测试用例，但是 Odoo 12使用Headless Chrome。它开放了新的可能性。通过Odoo 12，你可以录制失败测试用例的视频，或者对失败的测试用例截图。

#### 如何实现…

为测试用例录屏要求有ffmpeg包。安装该包，你需要在Terminal中执行如下命令（注意这个命令仅在Debian系操作系统中可以使用）：

```
apt-get install ffmpeg
```

生成视频或截图要求有日志文件件目录位置，因此如果你没有在单独的日志文件中保存日志的话，那么可以通过在命令行中使用 –logfile 如下：

```
/ododoo-bin -c server.conf -i my_library --test-enable --logfile=/var/log/odoo.log

```

#### 运行原理…

要为失败的测试用例生成截图，你需要运行服务时带有日志文件路径。在与日志文件名一起运行测试用例时，Odoo会在相同的目录中保存失败测试用例的截图。如果你没有在单独的日志文件中保存日志的话，那么可以通过在命令行中使用 –logfile选项。

要生成测试用例的视频，Odoo使用ffmpeg包。如果没有在服务器上安装该包的话，那么它仅会保存失败测试用例的截图。在安装了该包之后，你将可以看到失败测试用例的mp4文件。注意截图和视频仅对失败了的测试用例才会生成，因此如果你想要进行相关的测试的话，需要编写一个将会失败的测试用例。

> ℹ️为测试用例生成视频会消耗大量的磁盘空间，因此请谨慎使用这一选项，仅在实在需要时使用它。
