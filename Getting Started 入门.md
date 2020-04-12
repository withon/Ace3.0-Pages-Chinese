# 入门

原文地址：<https://www.wowace.com/projects/ace3/pages/getting-started>

翻译：with_on

github：<https://github.com/withon/Ace3.0-Pages-Chinese>

在阅读本文时，请记住，Ace3被设计为模块化的：不是所有的插件都需要所有模块。随意挑选那些适合您的插件的部分，并忽略不需要的部分。

## 插件基础配置

首先在\<WoW Directory\>\\Interface\\Addons中创建一个文件夹。文件夹可以起任何名字，只要它是唯一的即可，所以请取一些对您的插件具有描述性的名字。您将在这个文件夹中创建一些文件。

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

在`##`行之后，.toc文件的其他部分只是组成插件的文件列表。一个常用的方法是使用一个名为“embeds.xml”的单独文件来指定哪些库您希望被嵌入在插件中（例如Ace3库）。而且，大多插件通常会有至少一个“main"Lua文件包含配置插件的初始化代码——有些人喜欢对每个插件保持一致，例如”Main.lua“或“Core.lua”，另一些人更喜欢用插件名字来命名主文件，就像是.toc那样，但扩展名为.lua。

### embeds.xml

使用这个xml文件指定需要加载的库的位置（通常使用Include引用库自身的XML文件）。例如，使用LibStub并从插件文件夹中的Libs子目录中加载Ace3库：

```xml
<Ui xsi:schemaLocation="http://www.blizzard.com/wow/ui/ ..\FrameXML\UI.xsd">
  <Script file="Libs\LibStub\LibStub.lua"/>
  <Include file="Libs\AceAddon-3.0\AceAddon-3.0.xml"/>
  <Include file="Libs\AceConsole-3.0\AceConsole-3.0.xml"/>
</Ui>
```

可以通过添加其他Include行来添加其他库。您也可以改为在插件的.toc文件中引用每个库的xml文件，但是embeds.xml可以帮助您了解代码的哪些部分属于插件本身，哪些是第三方库。

### Core.lua

这个文件不必须命名为“Core.lua”，它可以是几乎任意带有.lua扩展名的文件，只要您在.toc中引用它。.lua文件包含运行插件的实际代码。有关制作完整的基于Ace3的插件的基本知识，请参见下面标题为“使用AceAddon-3.0”的部分。

## 使用AceAddon-3.0

### 创建插件对象

在确保您已经正确地引用了AceAddon-3.0库以及LibStub后（参见上文），插件的主Lua文件（例如Core.lua）可以创建一个Ace3插实例，如下所示：

```lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon")
```

这会为您提供一个插件对象，可用于与插件相关的所有AceAddon的调用。此外，如果您希望给插件对象提供某些额外功能，则可以使用“mixin”（混入其他库）。例如，使用以下代码代替将会为您在AceAddon的方法中增加由AceConsole提供的聊天交互功能。

```lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceConsole-3.0")
```

### 标准方法

AceAddon通常需要您的插件定义3个方法（通常在您的主Lua文件中），他们在不同的时间点被调用：

```lua
function MyAddon:OnInitialize()
  -- Code that you want to run when the addon is first loaded goes here.
end
```

OnInitialize()方法在您的插件首次被游戏客户端加载时被调用。这是一个还原之前保存的设置的好时机（有关此内容的更多说明，请参阅AceConfig上的信息）。

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

如上文AceAddon部分所述，在基于Ace3的插件中访问AceConsole功能的最简单的方法是在创建插件对象是作为一个mixin导入：

``` lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceConsole-3.0")
```

本节中的大多数示例将假设您已将AceConsole作为mixin导入。此外，如果您想将它作为独立的对象访问，则可以通过LibStub导入它：

```lua
MyConsole = LibStub("AceConsole-3.0")
```

### 使用AceConsole输出

使用AceConsole输出是很简单的，只需要调用its:Print方法并将您要输出的文字作为参数。如果作为mixin使用，则AceConsole将为您的插件对象提供:Print方法，否则，请使用您创建的AceConsole对象使用:Print方法：

```lua
-- AceConsole used as a mixin for AceAddon
MyAddon:Print("Hello, world!")

-- AceConsole used separately
MyConsole:Print("Hello, world!")
```

默认情况下，:Print将会输出到默认聊天框。如果您希望输出到其他聊天框，只需要将其作为第一个参数传递给:Print，如下所示：

```lua
MyAddon:Print(ChatFrame1, "Hello, World!")
```

### 使用AceConsole配置斜杠命令

为了使您的插件能够处理斜杠命令，您需要对其进行注册，并提供对其进行处理的函数的引用。这将使用AceConsole的RegisterChatCommand()方法。请注意，第一个参数（命令）不应包含斜杠。第二个参数可以是您的插件对象的方法名称或者一个函数。

```lua
MyAddon:RegisterChatCommand("myslash", "MySlashProcessorFunc")

function MyAddon:MySlashProcessorFunc(input)
  -- Process the slash command ('input' contains whatever follows the slash command)
end
```

但是，在许多情况下，利用AceConfig自动生成斜杠命令可能比手工编写更简单。继续阅读有关如何使用AceConfig配置插件选项。

## 使用AceConfig-3.0

### 创建选项表和处理程序

AceConfig被设计的是您可以很容易的提供对插件选项的访问，同时还和最终用户的配置界面保持一致。为此，AceConfig将自动生成修改设置的接口：作为插件作者，您只需要向其提供您希望提供的选项和怎样处理他们的信息。这将通过一个传递给AceConfig的表来实现。基本示例如下所示：

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

上面的选项表定义了一个简单的文本输入设置，“msg”。set和get可以是您的插件对象的方法名称（或任何为处理程序的对象）、或者完整的函数。这些方法或函数将在那个设置请求相应操作时被调用。因此，可以将它们设置如下：

```lua
function MyAddon:GetMyMessage(info)
    return myMessageVar
end

function MyAddon:SetMyMessage(info, input)
    myMessageVar = input
end
```

有关各种类型的设置及其可能的修饰符的更多详细信息，请参阅AceConfig-3.0文档。

### 注册选项

在定义选项表后，还需要将其注册到AceConfig。您还可以选择同时将其与斜杠命令自动绑在一起。要注册它，请使用LibStub得到一个AceConfig-3.0对象，然后调用它的RegisterOprionsTable()方法：

``` lua
LibStub("AceConfig-3.0"):RegisterOptionsTable("MyAddonName", options, {"myslash", "myslashtwo"})
```

第三个参数是您希望与这个选项集绑定的斜杠命令。如果您不希望对您的选项集绑定任何命令（即，如果您只想要GUI配置或者类似配置），请将第三个参数设置为nil。

## 使用AceDB-3.0

### 准备SavedBariables

为了将插件的一些内容保存到下次加载，插件必须声明哪些需要保存（通过TOC文件的SaveVariables字段）。通常习惯用一个以插件名称加上DB后缀的表来保存数据：

```lua
## SavedVariables: MyAddonDB
```

上面的代码应该添加到插件的.toc文件中的`##`行中。即使不使用Ace，这也是保存所有设置的必须步骤。AceDB只是在SavedVariables上封装了一个更易于访问的层。

### 初始化AceDB

需要知道的是，AceDB是基于SavedVariables的。因此，为了使AceDB能够加载之前保存的值，需要在SavedVariables被加载后进行初始化。对于插件作者来说，这意味着您不应在插件的主块~~main chunk~~中创建AceDB实例，而是在OinInitialize()或更靠后的时候创建它：

```lua
function MyAddon:OnInitialize()
    self.db = LibStub("AceDB-3.0"):New("MyAddonDB")
end
```

New()的第一个参数是您在TOC中设置的SavedVariables的名字。如果需要，您还可以传递一个表来指定DB的默认值（如果尚不存在）。此外，一个给更高级的作者的简短说明：如果由于某种原因您的插件导致LoadAddon()在主块中被调用，OnIntialize将会过早执行，此时SavedVariables还未加载，因此您需要采取其他措施来延迟初始化AceDB.

### 保存数据值

在配置好AceDB后，就很容易使用了。AceDB提供了8个子表：**char**，**realm**，**class**，**race**，**faction**，**factionrealm**，**profile**，**global** 。对于共享该子表的所有加载项，表中每个表的任何键和值都将被保存。例如，对于所有加载项，**realm**子表中的所有值都将被同一服务器中的任何角色保存。大多数子表的命名是不言自明的（factionrealm和realm相似，但仅限同一阵营）。唯一例外是**profile**子表，该表允许访问用户配置文件。

```lua
function MyAddon:MyFunction()
    self.db.char.myVal = "My character-specific saved value"

    self.db.global.myOtherVal = "My global saved value"
end
```

### 使用配置文件Profiles

Profiles允许你轻松的在db.profile子表中保存的配置中切换。要设置当前活跃的配置文件，只需要使用db对象的SetProfile()方法。

```lua
db:SetProfile("NewActiveProfile")
```

你可以通过GetCurrentProfile()方法获得当前活跃的配置文件，或通过GetProfiles()方法获得名称表（整数索引）和现有配置文件的个数：

``` lua
activeProfile = db:GetCurrentProfile()
possibleProfiles, numProfiles = db:GetProfiles()
```

更多相关信息（包括复制、删除、重置配置），请参阅AceDB-3.0文档。

### 使用AceDBOprions-3.0

AceDBOptions提供了一种使用AceDB和AceConfig将配置文件管理集成到插件中的便捷方法，而不需要担心SetProfile/GetProfiles。它可以通过调用一个简单的独立函数设置它，该函数能够返回一个可以被包含在传递给AceConfig的选项表中的表：

```lua
options.args.profile = LibStub("AceDBOptions-3.0"):GetOptionsTable(db)
```

你可以在将选项表传递给AceConfig之前调用它（db是AceDB中数据库对象的名称）。在这种情况下，oprions.args.profile将在你的选项表中期望“profile”命令（及子命令）的位置。

注意：生成的选项表在所有使用它的插件之间共享，请勿更改！

## 使用AceEvent-3.0

### 导入AceEvent功能

推荐使用AceEvent的方法是作为mixin，如下所示：

```lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceEvent-3.0")
```

如果您不使用AceAddon，你仍可以通过AceEvent的Embed()函数将AceEvent嵌入在对象或表中：

```lua
LibStub("AceEvent-3.0"):Embed(MyObject)
```

如果您确实不想在您的对象中嵌入AceEvent的方法，则可以实例化一个单独的AceEvent对象：

```lua
local AceEvent = LibStub("AceEvent-3.0")
```

但是，如果不嵌入它，则会失去一些功能，比如在禁用时自动注销事件以及更好的错误报告等。以下所有实例均假设您已将AceEvent嵌入，但如果你没有这么做，只需要将MyAddon:function替换为AceEvent.function，他们仍可以工作（尽管您需要提供一两个额外的参数，请参阅AceEvent-3.0文档获取更多详细信息）。

### 注册事件

简单来说，注册事件需要做的是：

```lua
MyAddon:RegisterEvent("NAME_OF_EVENT")
```

这将注册您的插件，使其接收给定插件名的事件，并尝试调用MyAddon:NAME_OF_EVENT()进行处理。

```lua
function MyAddon:NAME_OF_EVENT()
    -- process the event
end
```

您还可以指定处理程序的函数或方法名称，而不是让AceEvent查找默认方法名称：

```lua
MyAddon:RegisterEvent("NAME_OF_EVENT", "MyHandlerMethod")

function MyAddon:MyHandlerMethod()
    -- now handle it!
end

MyAddon:RegisterEvent("NAME_OF_OTHER_EVENT", function() doSomethingSpiffy() end)
```

上面的示例丢弃了事件中的所有参数。如果你希望访问他们，只需要在处理程序的参数表中包含即可。但是，请注意，在任何处理程序中，第一个传递的始终是事件的名称，之后是事件的参数：

```lua
function MyAddon:NAME_OF_EVENT(eventName, arg1, arg2, arg3)
    -- do some stuff
end

function MyAddon:NAME_OF_OTHER_EVENT(eventName, ...)
    -- do some more stuff
end
```

### 发送/接收插件间消息

AceEvent还提供对“messages”消息的支持，消息基本上和事件类似，但他们不是由WoW客户端触发的，而是由其他插件触发。如果你需要多个插件间通讯，这将很有用。注册消息与注册事件十分相似（指定处理程序也是）：

```lua
MyAddon:RegisterMessage("NAME_OF_MESSAGE")

function MyAddon:NAME_OF_MESSAGE()
    -- handle the message
end
```

发送消息到其他插件很简单：

```lua
MyAddon:SendMessage("NAME_OF_MESSAGE")
MyAddon:SendMessage("NAME_OF_OTHER_MESSAGE", arg1, arg2)
```

请注意，消息仅用于在同一客户端上运行的插件！如果你需要在不同玩家间发送消息，则需要使用AceComm。

## 使用AceComm-3.0

### 导入AceComm功能

与AceEvent一样，AceComm可以被混入、嵌入或作为单独的对象。每一个的示例（请选择一个）：

```lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceComm-3.0")
LibStub("AceComm-3.0"):Embed(MyObject)
local AceComm = LibStub("AceComm-3.0")
```

与AceEvent一样，如果将AceComm作为单独的对象，则将失去方便的自动注销和更好的错误报告。

### 向其他客户端发送消息

要将消息发送到其他客户端，请使用SendCommMessage()方法。它的基本用法有以下参数：

| 参数   | 描述                                         |
| :------ | :-------------------------------------------- |
| *prefix* | 字符串标记，使接收者监控他们想要接收的消息。必须是可打印字符（\\032 - \\255）。 |
| *text* | 发送的字符串中的实际数据，可以包含任何字符（\000除外）。长度不是问题，太长二无法在单个comm消息中发送的数据将自动拆分，并在另一终端重新组合。 |
| *distribution* | 将消息发送到哪个频道。可用的频道为“PARTY”，“RAID”，“BATTLEGROUND”，“GUILD”和“WHISPER”。 |
| *target* | 仅当distribution="WHISPER"时使用，此字符串为消息发送到的角色名：“角色名”或“角色名-服务器”。 |

```lua
MyAddon:SendCommMessage("MyPrefix", "the data to send", "RAID")
MyAddon:SendCommMessage("MyPrefix", "more data to send", "WHISPER", "charname")
```

### 接收来自其他客户端的消息

要接收消息，您的插件需要将其注册为监听者，监听希望接收的消息的前缀。这将通过RegisterComm()方法完成。

```lua
MyAddon:RegisterComm("prefix")
```

默认情况下，AceComm将尝试调用插件对象的OnCommReceived()方法。你也可以通过将方法名或函数作为RegisterComm()的第二个参数指定一个处理程序：

```lua
MyAddon:RegisterComm("prefix2", "MySecondCommHandler")
MyAddon:RegisterComm("prefix3", function() prefixthree() end)
```

处理程序被传入4个参数，这些参数与传递给SendCommMessage的参数相同，除了第4个参数。第4个参数是一个字符串，内容为消息的发送者（无论distribution类型为什么）。

```lua
function MyAddon:OnCommReceived(prefix, message, distribution, sender)
    -- process the incoming message
end
```

## 使用AceHook-3.0

### 导入AceHook功能

就像其他Ace库一样，最好将AceHook用作mixin，因为如果用户决定禁用您的插件，则不希望留下闲置钩子。

```lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceHook-3.0")
```

### 钩子功能

钩住一个函数有两种常见方法：pre-hook（常规）和post-hook（安全）。Pre-hook在被钩住的函数之前（有时代替它）被调用，在“原始”情况下，pre-hooks依赖于钩子处理程序来调用它认为必要的原始函数（在Ace3中，默认的非原始的hooks或“故障安全”的hooks将在完成后自动调用原始函数）。另一方面，安全钩子(secure hooks)在被钩住的函数执行之后执行，，并且安全钩子钩住的插件指定的处理程序的返回值被丢弃。在钩住Blizzard UI的受保护元素时，使用安全钩子来避免污染执行路径是必须的（有关污染的更多详细信息，参见https://wowwiki.fandom.com/wiki/Secure_Execution_and_Tainting）。

#### 标准钩子

使用AceHook的Hook()方法放置标准钩子，通常将其混入插件对象中。需要被钩住的函数可以通过函数名或对象和方法名来指定。

```lua
MyAddon:Hook("APIFunctionName")
MyAddon:Hook(TargetObject, "TargetMethod")
```

默认情况下，AceHook会将调用重定向到你的拥有和原始函数相同的名字插件对象方法：

```lua
function MyAddon:APIFunctionName(...)
    -- handle whatever we want to do before the hooked function is called
end
```

如果你希望调用其他函数，则可以指定一个处理程序。下面的前两个示例将调用函数hadlerFunc()，第三个将调用MyAddon:handlerMethod()：

```lua
MyAddon:Hook("APIFunctionName", handlerFunc)
MyAddon:Hook(TargetObject, "TargetMethod", handlerFunc)

MyAddon:Hook("APIFunctionName", "handlerMethod")
```

如果你想要pre-hook一个通常的安全函数，并且不介意对其污染，你可以在参数列表的末尾加上true来超脱AceHook的常规安全检查。但是，这将污染执行路径：

```lua
MyAddon:Hook("APISecureFunctionName", handlerFunc, true)
```

#### 原始钩子

默认情况下，Ace3制作的钩子将会在你的钩子程序执行之后自动以原始参数自动调用被钩住的原始函数。但是，如果你希望改变传递给原始函数的参数，或者甚至不执行原始函数，则需要设置一个替代的原始挂钩。

设置原始钩子与设置常规钩子有着相同的选项，但是使用RawHook()代替：

```lua
MyAddon:RawHook("APIFunctionName")
MyAddon:RawHook(TargetObject, "TargetMethod")
```

请记住，如果您设置了原始钩子，则除非您根本不想运行它，否则您需要自己调用被钩住的原始函数！此外，与Hook()一样，如果您想要原始地预挂钩一个安全函数（if you want to raw pre-hook） ，添加true作为最后一个参数将超越AceHook的安全函数检查。

处理函数如下所示：

```lua
-- for direct function hooks
function MyAddon:APIFunctionName(...)
  -- call the original function through the self.hooks table
  self.hooks["APIFunctionName"](...)
end

-- for object hooks
function MyAddon:TargetMethod(object, ...)
  -- call the original function through the self.hooks table
  self.hooks[object]["TargetMethod"](object, ...)
end
```

你还可以决定将要返回什么值，直接返回原始函数的结果，或者返回你自己的值。

#### 安全钩子

安全的post-hooks也有与常规钩子类似的语法。只需记住你的处理函数将在被钩住的函数执行之后被调用，你返回的任何内容都会被忽略。

```lua
MyAddon:SecureHook("APISecureFunctionName")
```

### 钩子脚本

挂钩框架脚本与挂钩函数非常类似，但是不调用Hook()或其变体并指定要挂钩的函数，而是调用HookScript()的变体并指定框架和脚本：

```lua
MyAddon:HookScript(TargetFrame, "ScriptName")
```

与非脚本钩子一样，默认处理程序是您的addon对象的方法，其名称与被钩住的脚本相同。

原始版本和安全版本的脚本钩子也是可用的：

```lua
MyAddon:RawHookScript(TargetFrame, "ScriptName")
MyAddon:SecureHookScript(TargetFrame, "ScriptName")
```

### 检查是否已经钩住

IsHook()方法使您可以检查函数是否已经被钩住。与Hook()一样，您可以指定一个函数引用或者一个对象和方法名。它返回两个值，第一个是一个简单的布尔值，用于确定AceHook是否已经布置钩子，第二个是该钩子指定的处理程序的引用（如果存在，不存在则为nil）。

```lua
hookexists, hookhandler = MyAddon:IsHooked("APIFunctionName")
```

## 使用AceLocale-3.0

### 注册翻译

使用AceLocal设置给定的语言环境的翻译非常简单。首先，为语言环境创建一个新的Lua文件。为了便于引用，建议其中包括语言环境的名字，但这不是必须的。

您需要将此新文件添加到插件的TOC文件中，并确保它在插件的主文件名之前，否则，在您的代码尝试使用它之前，它可能不会被加载！

在新文件的开头，从AceLocale获取一个新的语言环境设置对象：

```lua
local L = LibStub("AceLocale-3.0"):NewLocale("MyAddon", "enUS", true)
```

NewLocale的第一个参数是你的插件的名字（必须在所有语言环境中保持一致，以便AceLocale将它们与插件关联在一起）。第二个是语言环境标识符（常见的有enUS、deDE、frFR、koKR、ruRU、zhCN和zhTW。欧洲英语客户端是enGB，但是AceLocale自动为enGB加载enUS词条）。最后一个参数是一个布尔值，用于指定==指定您定义的语言环境是否作为默认语言环境。对于大多数插件，这在enUS中可能为true，而在其他语言环境中则为false。

在获取语言环境对象之后，只需要设置翻译表即可：

```lua
if L then

L["identifier"] = "Translation for that identifier"
L["something"] = "Translation for something"

end
```

使用`if`块是因为，如果您已经定义了语言环境。如果您不小心尝试两次为给定的语言环境定义翻译，则可以避免出现问题。

### 使用翻译

在您的主代码中（Core.lua之类），要求AceLocale给您恰当的翻译对象：

```lua
local L = LibStub("AceLocale-3.0"):GetLocale("MyAddon", true)
```

第一个参数是插件名称，与您在语言环境文件中指定的插件名称相同。第二个是一个布尔值，它指定如果找不到语言环境信息，AceLocale是否应该以静默方式失败（如果为true，则该语言环境是可选的，如果未找到，则静默返回nil）。

在获取翻译对象之后，只需替换为之前使用的预本地化字符串:

```lua
function MyAddon:MyFunction()
    self:Print(L["identifier"])
    if userinput == L["something"] then
        doSomething()
    end
end
```

### 混合变量

如果要使用混合着变量的文本来实现不同的输出，还可以在语言环境表中使用函数。因此，单词的顺序在您的脚本中不重要，并且翻译看起来会更加自然。

```lua
-- enUS/enGB:
L['Added X DKP to player Y.'] = function(X,Y)
  return 'Added ' .. X .. ' DKP for player ' .. Y .. '.';
end
-- deDE:
L['Added X DKP to player Y.'] = function(X,Y)
  return X .. ' DKP f\195\188r Spieler ' .. Y .. ' hinzugef\195\188gt.';
end
-- script.lua:
self:Print(L['Added X DKP to player Y.'](dkp_value, playername));
```

## 使用AceSerializer-3.0

### 导入AceSerializer方法

与其他Ace3模块一样，AceSerializer-3.0可作为混入类或单独的对象。

```lua
MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceSerializer-3.0")
```

### 输出序列化数据

要获得数据的序列化字符串表现，只需要使用Serialize()方法并传入所有你想要的序列化的值：

```lua
MyVal1 = 23
MyVal2 = "some text"
MyVal3 = {"foo", 42, "bar"}

serializedData = MyAddon:Serialize(MyVal1, MyVal2, MyVal3)
```

### 从序列化字符串加载数据

要获取数据的序列化字符串，请调用Deserialize()方法并将序列化字符串传入。它将返回多个值：第一个总是一个布尔值，指示成功（true）或失败（false）。如果成功，其余的返回值将是原始的序列化数据项，顺序与传递给Serialize()的顺序相同。如果失败，则第二个返回值将是一条指示失败原因的消息。

```lua
success, MyVal1, MyVal2, MyVal3 = MyAddon:Deserialize(serializedData)
if not success then
  -- handle error
end
```