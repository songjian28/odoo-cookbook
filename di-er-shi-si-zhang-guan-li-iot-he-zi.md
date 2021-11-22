# 第二十四章 管理IoT盒子

Odoo提供对物联网（IoT）的支持。IoT是一种设备/传感器网络，可在互联网上进行数据交换。通过系统连接这些设备，就可以进行使用了。例如，将Odoo连接到打印机上，就可以直接向打印机发送PDF报表了。Odoo使用称之为IoT盒子的硬件，用于连接打印机、卡尺、支付设备和脚踏开关等设备。本章中，我们将学习如何设置和配置IoT盒子。这里，我们将讲解如下课题：

* 对树莓派（Raspberry Pi）闪存IoT盒子镜像
* 通过网络连接IoT盒子
* 对Odoo添加IoT盒子
* 加载驱动及列出已连接设备
* 从设备接收输入
* 通过SSH访问IoT盒子
* 配置 POS（销售点）
* 直接向打印机发送PDF报表

注意本章的目标是安装和配置IoT盒子。开发硬件驱动不属于本书的讨论范围。如果你想要深入地学习IoT盒子，请认真研究企业版中的iot模块。

### 技术准备

IoT盒子是一个基于树莓派的设备。本章中的各小节基于Raspberry Pi 3 Model B+，可通过https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/进行购买。IoT是企业版部分的功能，因此需要有企业版才能完成本章中的操作。

本章中的所有代码可通过[GitHub仓库](https://github.com/alanhou/odoo14-cookbook/tree/main/Chapter01)进行下载。

### 对树莓派（Raspberry Pi）闪存IoT盒子镜像

本节中，你将学习如何通过IoT例子的镜像闪存microSD卡。注意本节仅适用于那些购买了空的树莓派的人。如果你不是从Odoo官方购买的IoT盒子，请跳过这一节，因为它已经预载了IoT例子镜像。

#### 准备工作

树莓派3 Model B+使用microSD卡，因此本节中我们使用了microSD卡。你需要将microSD卡连接到电脑上。

#### 如何实现…

执行如下步骤来在SD卡上安装IoT盒子镜像：

1. 将microSD卡插入你的电脑（如果电脑上没卡槽请使用适配器）。
2. 从Odoo的nightly构建上下载IoT例子镜像。镜像地址为https://nightly.odoo.com/master/iotbox/。
3. 下载并在电脑上安装Balena Etcher。下载地址为https://www.balena.io/etcher/。
4. 打开Balena Etcher，选择IoT例子镜像，并选择闪存你的microSD卡。你可以参见如下截图：\
   TODO
5. 点击Flash!按钮并等待处理完成。
6. 退出microSD卡放到树莓派中。

在完成了给定的步骤后，你的microSD卡中就载入了IoT例子镜像，并已准备好在IoT例子中使用了。

#### 运行原理…

在本节中，我们在microSD卡中安装了IoT盒子镜像。在第2步中，我们从Odoo nightly下载了从IoT盒子镜像。在nightly页面上，我们可以找到针对IoT盒子的不同镜像。你需要从Odoo nightly选择最新的镜像。在写本书时，我们使用了最新镜像otboxv18\_12.zip。IoT盒子镜像基于Raspbian Stretch Lite OS并且镜像加载时带有集成到Odoo实例中IoT盒子所需的库和模块。

在第3步中，我们下载了Balena Etcher 工具用于闪存microSD卡。在这一部分中，我们使用Balena Etcher来闪存microSD卡，但你也可以使用其它工具来闪存microSD卡。

在第4步中，我们通过IoT盒子镜像闪存了microSD卡。注意这个过程可能需要好几分钟。在完成这一过程后，microSD就可供使用了。

如果你想要验证闪存是否成功可执行如下步骤：

1. 将microSD卡挂载到树莓派中。
2. 通上电源然后通过HDMI线连接到外部显示器（在实例使用中，外部显示设置并非必须的，这里我们使用它来作验证）。
3. 系统会启动并显示如下页面：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102214043.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102214043.png)

#### 扩展知识…

在之前的Odoo版本中，PosBox用于销售点（POS）应用。IoT盒子支持PosBox的所有功能，所以如果你使用Odoo社区版而又想要集成设备，可以使用IoT镜像来通过不同设备连接Odoo实例。参见_配置 POS（销售点）_一节来获取更多信息。

### 通过网络连接IoT盒子

IoT盒子通过网络与Odoo实例进行通讯。连接IoT盒子是很关键的一步，如果出现了错误，你会在使用Odoo连接IoT盒子时碰到报错。

#### 准备工作

使用IoT盒子镜像将microSD卡挂载到树莓派中，然后通过电源连接连接树莓派。

#### 如何实现…

树莓派3 Model B+支持两种类型的网络连接 – 通过以太网或WiFi。通过以太网连接IoT盒子非常简单，只需要将IoT盒子通过RJ45网线进行连接即可，这时就可以开始使用IoT盒子了。通过WiFi连接IoT盒子会复杂些，因为你可能没有连接的显示设备。执行如下步骤来通过WiFi连接IoT盒子：

1. 为IoT盒子接电源通电（如果插上了网线，请拔掉网线并重启IoT盒子）
2. 打开电脑并连接名为IoTBox的WiFi网络，如下图所示（无需密码）：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102221840.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102221840.png)
3. 在连接了WiFi之后，你会看到弹出的IoT盒子主页，如下图所示（如未弹出，请在浏览器中输入盒子的 IP 地址）\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102225871.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102225871.png)
4. 设置IoT盒子名称并保留服务令牌为空，然后点击Next。这会跳转一个能够看到WiFi网络列表的页面：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102233851.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102233851.png)
5. &#x20;选择你想要连接的WiFi网络并填入密码。然后，点击Connect按钮。如果你填写了正确的信息，会跳转到最终页面：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102240492.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102240492.png)

在执行了这些步骤之后，你的IoT盒子就已成功连接网络并可以开始与Odoo实例进行集成了。

#### 运行原理…

通过以太网来定连接Odoo实例与IoT盒子非常的简单，只需通过RJ45网线连接IoT盒子，即可开始使用IoT盒子了。但通过WiFi连接IoT盒子则不同，它的困难之处在于IoT盒子并没有显示器或图形界面。你没有输入WiFi网络密码的界面。因此，解决这一问题的方法是拔掉IoT盒子的网线（如已连接）并进行重启。在这种情况下，IoT盒子盒子会创建其自己的WiFi热点，名为 IoTBox，参见第2步。你需要连接名为 IoTBox的WiFi，所幸无需密码。一旦连接了名为 IoTBox的WiFi，就会弹出第3步中所示的页面。这里你可以将IoT盒子命名为Assembly-line IoT Box等名称。现在保留服务令牌为空格，我们会在_向Odoo添加IoT盒子_一节中进行相关学习。然后点击Next按钮。

点击Next按钮之后，会显示一个WiFi网络列表，如第4步所示。此处你可以将IoT盒子连接到你自己的WiFi网络。确保选择正确的网络。你需要将IoT盒子连接到与Odoo实例所运行电脑连接的相同WiFi上。IoT盒子与Odoo实例之间在局域网（LAN）中进行通讯。也就是说如果两者连接到不同的网络上，IoT盒子就无法正常运作。

在选择了正确的WiFi网络后，点击Connect。然后IoT盒子会关闭它自己的执行，重新连接上所配置的WiFi网络。这样IoT盒子就准备好供下一步使用了。

### 对Odoo添加IoT盒子

我们的IoT盒子已连接上本地网络，可供Odoo使用了。在本节中，我们将通过Odoo实例连接IoT盒子。

#### 准备工作

确保IoT盒子是开启的并已连接到Odoo实例所运行电脑连接的相同的WiFi网络上。

> ℹ️需要注意以下几点，否则IoT盒子无法与 Odoo进行连接。
>
> * 如果你在本地实例上测试IoT盒子，需要使用http://192.168.1.\*:8069（本地 IP）来代替http://localhost:8069。如果使用localhost，IoT盒子会无法连接到Odoo实例。
> * 你需要通过 Odoo 实例所运行电脑使用的相同WiFi或以太网连接IoT盒子。否则IoT盒子会无法连接到Odoo实例。
> * 如果你的Odoo实例运行着多数据库。IoT盒子不会与Odoo实例进行自动连接。使用–db-filter参数来避免这一情况。

#### 如何实现…

为建立IoT盒子与Odoo的连接，首先你需要在Odoo实例中安装iot模块。需要进入Apps菜单并搜索Internet of Things模块来进行安装。该模块如下图所示。安装好模块就可以进行后续操作了：

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102253862.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102253862.png)

在安装了iot模块之后，你可以建立实例与IoT盒子之间的连接。有两种方式来建立连接：自动连接或手动连接。我们先来学习自动连接。

**自动连接IoT盒子**

执行如下步骤来将IoT盒子与Odoo实例进行自动的连接：

1. 打开IoT菜单。
2. 在控制面板中点击Connect按钮。这时会显示如下弹窗。点击SCAN按钮：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102261412.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102261412.png)
3. 它会自动扫描所有本地IP来查找 IoT盒子。如果Odoo找到了IoT盒子，就会在右侧进行显示，参见前述截图。
4. 关闭弹窗。此时你会看到IoT盒子已添加到列表中了：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102264910.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102264910.png)

**手动连接IoT盒子**

执行如下步骤来手动建立IoT盒子与Odoo实例之间的连接：

1. 打开IoT菜单。
2. 在控制面板中点击Connect按钮。此时会显示如下弹窗。复制令牌：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102273087.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102273087.png)
3. 以8069端口打开IoT盒子的IP。这会显示IoT盒子的首页。点击Name 输入框后的Configure按钮\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102280394.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102280394.png)
4. 设置IoT盒子名称并粘贴服务令牌。然后点击Connect按钮。这会开启对IoT盒子的配置。等待这一过程的结束：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102283327.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102283327.png)
5. 查看Odoo实例中的IoT菜单。你会看到新的IoT盒子：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102290181.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102290181.png)

#### 运行原理…

在安装了Internet of Things模块之后，会显示一个新的IoT菜单，通过它可以建立IoT盒子与Odoo的连接。在本节中，我们首先使用了自动化的方法连接了IoT盒子。这一方法会扫描本地的IP范围来查找可能为IoT盒子的设备。如果查找到了IoT盒子，Odoo会将其添加为IoT盒子。注意如果你的网络中有多个IoT盒子的话，这一方法会添加所有设备。如果IoT盒子已被连接，Odoo不会重复连接该IoT盒子。

第二种方法是手动的方式。这一方法对于网络中存在多个IoT盒子且仅希望添加部分设备时会非常有用，你需要从IoT盒子的弹窗中获取令牌。然后需要访问IoT盒子的 IP并将令牌添加到配置中。这会将IoT盒子添加到你的Odoo实例中。

如果你想要在WiFi配置的过程中将IoT盒子添加到Odoo实例中，也完全可能。在_通过网络连接IoT盒子_一节中，我将保留了服务令牌字段为空。仅需在该步中添加服务令牌即可：

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102294573.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102294573.png)

> ℹ️在使用IoT盒子时避免使用DHCP网络。这是因为IoT盒子的网络配置是基于IP地址进行添加的。如果你使用DHCP网络，那么IP地址就是动态分配的。那么就有可能会因IoT盒子被分配新的 IP 地址而无法响应。要避免这一问题，你可以将IoT盒子的MAC地址与固定的 IP 地址建立一个映射。

#### 扩展知识…

一旦通过Odoo实例连接了某个IoT盒子，就不可以将其连接到其它Odoo实例上了。如果你尝试扫描已配置的IoT盒子，会显示一个警告图标：

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102301898.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102301898.png)

如果希望将已有IoT盒子与Odoo实例进行连接，你会需要清楚该配置。你可以在IoT盒子的Odoo服务配置页通过 Clear 按钮清除这一IoT盒子配置：

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102305482.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102305482.png)

### 加载驱动及列出已连接设备

IoT盒子需要加载设备驱动来连接硬件设备。本节中，我们将来学习如何加载驱动并获取所连接设备的列表。

#### 准备工作

确保IoT盒子已开启并已将其连接到Odoo实例运行电脑所连接的相同WiFi网络。

#### 如何实现…

执行如下步骤来将设备驱动载入IoT盒子：

1. 打开IoT盒子主页并点击底部的drivers list按钮：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102312562.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102312562.png)
2. drivers list按钮会将你重定向到Drivers list页面，这里会有一个Load drivers 按钮。点击该按钮来载入驱动：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102315484.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102315484.png)
3. 返回IoT盒子首页。这里你会看到一个已连接设备的列表：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102322323.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102322323.png)

在执行了这些步骤之后，该IoT盒子对你所指定的设备就准备就绪了，你可以开始在应用中使用这些设备了。

#### 运行原理…

IoT盒子并不是仅局限于企业版。也可以在社区版中作为PoSBox来使用。设备的集成是企业版的一部分，因此IoT盒子并不带有设备驱动器，需要手动进行加载 。也可以从IoT盒子的首页加载驱动。可以通过底部的Load drivers按钮进行实现。注意这仅在通过企业版的Odoo实例连接IoT盒子时才能进行实现。在加载完驱动后，你就可以在IoT盒子的首页里看到一个设备列表。也可以通过IoT > Devices菜单查看Odoo实例中所连接的设备。在这个菜单中，你会看到每个IoT盒子所连接的一组设备：

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102330313.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102330313.png)

现在，IoT盒子仅支持有限制的硬件设备，如摄像头、脚踏开关、打印机和卡尺。Odoo所推荐的设备列表请见https://www.odoo.com/page/iot-hardware。如果尚未支持你的设备，可以付费来进行驱动开发。

### 从设备接收输入

IoT盒子仅支持有限的设备。现在这些硬件设备与制造应用相集成。但如果需要，可以在你的模块中集成所支持的设备。本节中，我们将通过IoT盒子来从摄像头抓取图片。

#### 准备工作

我们将使用[第二十三章 在Odoo中管理email](https://alanhou.org/manage-emails-odoo/)中_在聊天器中记录用户修改_一节的my\_library模块。本节中，我们将新增一个在借阅者归还图书时捕获和存储图像的字段。确保打开了IoT盒子并且通过它连接了可支持的摄像头设备。

#### 如何实现…

执行如下步骤来通过IoT盒子从摄像头捕获图片：

1.  在声明文件中添加依赖：\


    ```
    ...'depends': ['base', 'mail', 'quality_mrp_iot'],...
    ```
2.  在library.book.rent模型中新增字段：\


    ```
    ...
        device_id = fields.Many2one('iot.device', string='IoT Device',  domain="[('type', '=', 'camera')]")
        ip = fields.Char(related="device_id.iot_id.ip")
        identifier = fields.Char(related='device_id.identifier')
        picture = fields.Binary()
    ...
    ```
3.  在library.book.rent模型的表单视图中添加这些字段：\


    ```
    <group>    
        <field name="book_id" domain="[('state', '=', 'available')]"/>    
        <field name="borrower_id"/>    
        <field name="ip" invisible="1"/>    
        <field name="identifier" invisible="1"/>    
        <field name="device_id" required="1"/>    
        <field name="picture" widget="iot_picture" options="{'ip_field': 'ip', 'identifier': 'identifier'}"/>
    </group>
    ```

更新my\_library模块来应用这些修改。在更新后，就会出现捕获图片的按钮：

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102334137.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102334137.png)

#### 运行原理…

在第1步中，我们在声明文件中添加了对quality\_mrp\_iot模块的依赖。quality\_mrp\_iot模块属于企业版，包含通过IoT盒子来使用摄像头请求图像的组伯。这会安装mrp模块，但为进行简化，我们将使用quality\_mrp\_iot来作为依赖。如果你不想要使用这个依赖，可以创建自己的字段组件。参见[第十六章 网页客户端开发](https://alanhou.org/web-client-development/)来学习有关组件的更多知识。

第2步中，我们添加了请求通过摄像头捕获图像的字段。捕获图像需要两个东西：设备标识符和IoT盒子的IP地址。我们希望用户能够选取摄像头，因此添加了一个device\_id字段。用户会选择捕获图片的摄像头，并且基于所选择的摄像头设备，我们通过关联字段提取出IP和设备标识符信息。基于这个字段，在有多个IoT盒子时Odoo就知道使用哪个设备来捕获图像。我们还添加了一个二进制字段picture来保存图像。

第3步中，我们在表单视图中添加了字段。注意我们对picture字段使用了iot\_picture组件。我们以隐藏字段添加了ip和identifier字段，因为不希望对用户显示这些字段；而在picture字段的选项又需要使用它们。该组件会在表单视图中添加按钮，点击按钮时，Odoo会对IoT盒子进行请求来捕获图像。IoT盒子会以响应数据返回图像。这个响应会在picture二进制字段中进行保存。

#### 扩展知识…

Odoo IoT盒子支持蓝牙卡尺。如果想要在模块中进行测试，可以使用iot\_take\_measure\_button组件来在Odoo中进行获取 。这时需要使用\<widget>标签，如下面的代码所示。注意类似iot\_picture，这里也需要在表单视图中以隐藏字段添加ip和identifier字段。

```
<widget name="iot_take_measure_button"  options="{'ip_field': 'ip',    'identifier_field': 'identifier',    'measure_field': 'measure'}"/>

```

### 通过SSH访问IoT盒子

IoT运行于Raspbian OS之上，可以通过SSH来访问IoT盒子。本节中，我们将学习如何通过SSH来连接IoT盒子。

#### 准备工作

确保已打开IoT盒子，并且IoT盒子通过WiFi与Odoo实例所在电脑相同的网络进行连接。

#### 如何实现…

为通过SSH来连接IoT盒子，你将需要IoT盒子的IP地址。可以在它的表单视图中查看到IP地址。作为示例，本节中将使用192.168.43.6来作为IoT盒子的IP地地。执行如下步骤来通过SSH访问IoT盒子：

1.  打开Terminal并执行如下命令：\


    ```
    $ ssh pi@192.168.43.6
    pi@192.168.43.6's password:
    ```
2. 在终端中会询问命令，输入raspberry来作为密码。
3.  如果你填写了正确的密码，即可通过shell进行访问。执行如下命令来查看目录：\


    ```
    pi@AssemblyIoTBox:~ $ ls -l
    total 260
    -rw-r--r-- 1 root root 244271 Jan 16 14:17init_posbox_image.log
    drwxr-xr-x 5 pi pi 4096 Jan 16 12:57 odoo
    -rw-r--r-- 1 pi pi 27 Jan 16 14:20 odoo-remote-server.conf
    -rw-r--r-- 1 pi pi 11 Jan 16 14:20 token
    -rw-r--r-- 1 pi pi 20 Jan 16 14:26 wifi_network.txt
    ```

#### 运行原理…

我们使用了一个密码为raspberry的用户pi来通过SSH访问IoT盒子。SSH连接可在需要对IoT盒子进行问题调试时使用。SSH无需进行解释，但让我们来看看IoT盒子中Odoo的运行方式。

如下信息可能会有助于调试问题：

* IoT盒子内部运行一些Odoo模块。这些模块的名称通常以hw\_开头，在社区版中也有。可以在/home/pi/odoo/addon目录下查找到这些模块。
* 如果想要查看Odoo服务蝴蝶刀 ，可以通过/var/log/odoo/odoo-server.log文件进行访问。
*   Odoo通过名为odoo的服务运行，可以使用如下命令来启动、停止或重启该服务：\


    ```
    sudo service odoo start/restart/stop
    ```
* 大多数客户会通过直接断电来关闭IoT盒子。在这种情况下IoT盒子的操作系统并没有正常关闭。为避免系统的崩溃，IoT盒子的文件系统是只读的。

#### 扩展知识…

注意IoT盒子仅与本地机器进行了连接。因此，无法通过互联网直接远程访问它的shell。如果希望远程访问IoT盒子，可以在IoT盒子的远程调试页面中粘贴ngrok认证Token，参见下图。这会在IoT盒子中启用TCP通道，这样就可以在任何地方通过SSH来连接IoT盒子了。可通过https://ngrok.com/学习更多有关ngrok的知识。

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102341926.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102341926.png)

### 配置 POS（销售点）

Odoo 12中，IoT盒子可用于MRP和Point of Sale 应用。本节中，我们将学习如何对销售点应用配置IoT盒子。

#### 准备工作

确保IoT盒子已打开并且通过与Odoo实例所处的电脑相同的WiFi网络进行了连接。同时如未安装 POS 应用请进行安装。

#### 如何实现…

执行如下步骤来对POS 应用配置IoT盒子：

1. 打开POS 应用，并通过POS会话下拉打开Settings：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102344959.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102344959.png)
2. 点击Settings按钮。会被重定向到Settings页面。搜索IotBox / Hardware Proxy（硬件代理） 怎并勾选PosBox。这会启用更多选项：\
   [![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/201908210235193.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/201908210235193.png)
3. &#x20;选择想要在 POS 会话中使用的IoT盒子。如果需要使用硬件、条码扫描器，请进行对应的勾选。
4. 通过在控制面板中点击Save按钮来保存修改。

在配置之后，你就能够在POS应用中使用IoT盒子了。

#### 运行原理…

在Odoo之前的版本中，PosBox用于将 POS 应用与硬件进行集成。在版本12中，引入了IoT盒子，并且它可以类似PosBox那样用于 POS 应用。为了在POS 应用中使用IoT盒子，需要通过Odoo实例连接IoT盒子。如果不知道如何连接IoT盒子，请参见_向Odoo添加IoT盒子_一节。一旦通过Odoo连接了IoT盒子，就可以像第2步那样在POS 应用中选择IoT盒子。

这里可以选择在POS会话中使用的硬件。在保存修改之后，如果打开了POS会话，就可以在POS中使用启用了的硬件。如果通过设置启用了对应的硬件，但其并未连接IoT盒子，就会在顶栏中看到如下的警告：

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102355467.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102355467.png)

#### 扩展知识…

POS 应用属于社区版。如果你使用的是社区版，在 POS 设置中看到的不是IoT盒子选区而是IP地址字段：

[![【翻译中】Odoo 14开发者指南第二十四章 IoT盒子](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102361762.png)](https://alanhou.org/homepage/wp-content/uploads/2019/05/2019082102361762.png)

如果想要在社区版集成硬件，需要在该字段中使用IoT盒子的IP地址。
