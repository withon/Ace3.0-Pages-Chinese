# 入门

原文地址：<https://www.wowace.com/projects/ace3/pages/getting-started>

翻译：with_on

在阅读本文时，请记住，Ace3被设计为模块化的：不是所有的插件都需要所有模块。随意挑选那些适合你的插件的部分，并忽略不需要的部分。

## 插件基础配置

首先在\<WoW Directory\>\\Interface\\Addons中创建一个文件夹。文件夹可以起任何名字，只要它是唯一的即可，所以请取一些对你的插件具有描述性的名字。你将在这个文件夹中创建一些文件。

### .toc 文件

这个文件需要与文件夹同名，只是加上.toc后缀名。这是一个基础的文本文件，用来告诉WoW插件需要加载其他哪些文件。

``` toc
## Interface: 40000
## Title: My Addon's Title
## Notes: Some notes about this addon.
## Author: Your Name Here
## Version: 0.1

embeds.xml
Core.lua
```

以`##`开头的行向WoW提供了插件本身的信息——例如“Interface”指定了插件要加载的接口版本（在撰写本文时为40000），“Title”指定了插件在插件窗口中展示的名称，等等。

在`##`行之后，.toc文件的其他部分只是组成插件的文件列表。一个常用的方法是使用一个名为“embeds.xml”的单独文件来指定哪些库你希望被嵌入在插件中（例如Ace3库）。而且，大多插件通常会有至少一个“main"Lua文件包含配置插件的初始化代码——有些人喜欢对每个插件保持一致，例如”Main.lua“或“Core.lua”，另一些人更喜欢用插件名字来命名主文件，就像是.toc那样，但扩展名为.lua。

### embeds.xml

使用这个xml文件指定需要加载的库的位置（通常使用Include引用库自身的XML文件）。例如，使用LibStub并从插件文件夹中的Libs子目录中加载Ace3库：

```xml
<Ui xsi:schemaLocation="http://www.blizzard.com/wow/ui/ ..\FrameXML\UI.xsd">
  <Script file="Libs\LibStub\LibStub.lua"/>
  <Include file="Libs\AceAddon-3.0\AceAddon-3.0.xml"/>
  <Include file="Libs\AceConsole-3.0\AceConsole-3.0.xml"/>
</Ui>
```

可以通过添加其他Include行来添加其他库。你也可以改为在插件的.toc文件中引用每个库的xml文件，但是embeds.xml可以帮助你了解代码的哪些部分属于插件本身，哪些是第三方库。

### Core.lua

这个文件不必须命名为“Core.lua”，它可以是几乎任意带有.lua扩展名的文件，只要你在.toc中引用它。.lua文件包含运行插件的实际代码。有关制作完整的基于Ace3的插件的基本知识，请参见下面标题为“使用AceAddon-3.0”的部分。

## 使用AceAddon-3.0

### 创建插件对象

在确保你已经正确地引用了AceAddon-3.0库以及LibStub后（参见上文），插件的主Lua文件（例如Core.lua）可以创建一个Ace3插实例，如下所示：

```lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon")
```

这会为你提供一个插件对象，可用于与插件相关的所有AceAddon的调用。此外，如果你希望给插件对象提供某些额外功能，则可以使用“混入类”（混入其他库）。例如，使用以下代码代替将会为你在AceAddon的方法中增加由AceConsole提供的聊天交互功能。

```lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceConsole-3.0")
```

### 标准方法

AceAddon通常需要你的插件定义3个方法（通常在你的主Lua文件中），他们在不同的时间点被调用：

```lua
function MyAddon:OnInitialize()
  -- Code that you want to run when the addon is first loaded goes here.
end
```

OnInitialize()方法在你的插件首次被游戏客户端加载时被调用。这是一个还原之前保存的设置的好时机（有关此内容的更多说明，请参阅AceConfig上的信息）。

```lua
function MyAddon:OnEnable()
    -- Called when the addon is enabled
end

function MyAddon:OnDisable()
    -- Called when the addon is disabled
end
```

OnEnable()和OnDisable()方法在用户启用／禁用插件时被调用。与OnInitialize()不同，这也许会在不重载全部UI的情况下发生多次。

## 使用AceConsole-3.0

### 导入AceConsole

如上文AceAddon部分所述，在基于Ace3的插件中访问AceConsole功能的最简单的方法是在创建插件对象是作为一个混入类导入：

``` lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceConsole-3.0")
```

本节中的大多数示例将假设你已将AceConsole作为混入类导入。此外，如果你想将它作为独立的对象访问，则可以通过LibStub导入它：

```lua
MyConsole = LibStub("AceConsole-3.0")
```

### 使用AceConsole输出

使用AceConsole输出是很简单的，只需要调用its:Print方法并将你要输出的文字作为参数。如果作为混入类使用，则AceConsole将为你的插件对象提供:Print方法，否则，请使用你创建的AceConsole对象使用:Print方法：

```lua
-- AceConsole used as a mixin for AceAddon
MyAddon:Print("Hello, world!")

-- AceConsole used separately
MyConsole:Print("Hello, world!")
```

默认情况下，:Print将会输出到默认聊天框。如果你希望输出到其他聊天框，只需要将其作为第一个参数传递给:Print，如下所示：

```lua
MyAddon:Print(ChatFrame1, "Hello, World!")
```

### 使用AceConsole配置斜杠命令

为了使你的插件能够处理斜杠命令，你需要对其进行注册，并提供对其进行处理的函数的引用。这将使用AceConsole的RegisterChatCommand()方法。请注意，第一个参数（命令）不应包含斜杠。第二个参数可以是你的插件对象的方法名称或者一个函数。

```lua
MyAddon:RegisterChatCommand("myslash", "MySlashProcessorFunc")

function MyAddon:MySlashProcessorFunc(input)
  -- Process the slash command ('input' contains whatever follows the slash command)
end
```

但是，在许多情况下，利用AceConfig自动生成斜杠命令可能比手工编写更简单。继续阅读有关如何使用AceConfig配置插件选项。

## 使用AceConfig-3.0

### 创建选项表和处理程序

AceConfig被设计的是你可以很容易的提供对插件选项的访问，同时还和最终用户的配置界面保持一致。为此，AceConfig将自动生成修改设置的接口：作为插件作者，你只需要向其提供你希望提供的选项和怎样处理他们的信息。这将通过一个传递给AceConfig的表来实现。基本示例如下所示：

```lua
local options = {
    name = "MyAddon",
    handler = MyAddon,
    type = 'group',
    args = {
        msg = {
            type = 'input',
            name = 'My Message',
            desc = 'The message for my addon',
            set = 'SetMyMessage',
            get = 'GetMyMessage',
        },
    },
}
```

上边的选项表定义了一个简单的文本输入设置，“msg”。set和get可以是你的插件对象的方法名称（或任何为处理程序的对象）、或者完整的函数。这些方法或函数将在那个设置请求相应操作时被调用。因此，可以将它们设置如下：

```lua
function MyAddon:GetMyMessage(info)
    return myMessageVar
end

function MyAddon:SetMyMessage(info, input)
    myMessageVar = input
end
```

有关各种类型的设置及其可能的修饰符的更多详细信息，请参阅AceConfig-3.0文档。