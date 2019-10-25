---
title: Robots.txt 完整使用指南
categories: [杂记]
tags: [seo] 
date: 2019-10-25 11:12:17
---

> 转载: [Robots.txt 完整使用指南](https://www.simcf.cc/3442.html)

``Robots.txt``是一个小文本文件，位于网站的根目录中。它告诉抓取工具是否要抓取网站的某些部分。该文件使用简单的语法，以便爬虫可以放置到位。

写得好，你将在索引天堂。写得不好，最终可能会从搜索引擎中隐藏整个网站,该文件没有官方标准。但你可以使用`Robots.txt`做更多的工作，而不是网站大纲，比如使用通配符，站点地图链接，和`Allow`指令，所有主要搜索引擎都支持这些 扩展。

在一个完美的世界里，没有人需要`Robots.txt`。如果网站上的所有页面都是供公众使用的，那么理想情况下，应该允许搜索引擎抓取所有页面。但我们并不是生活在一个完美的世界里。许多站点都有蜘蛛陷阱，规范URL问题以及需要远离搜索引擎的非公共页面，而`Robots.txt`用于使您的网站更接近完美。

## `Robots.txt`如何工作

如果你已经熟悉了`Robots.txt`的指令，但担心你做错了，请跳到常见错误部分。如果你是新手，请继续阅读 。

可以使用任何纯文本编辑器制作`Robots.txt`文件，但它必须位于站点的根目录中，并且必须命名为`Robots.txt`，您不能在子目录中使用该文件。

如果域名是example.com，则`Robots.txt`网址应为：

    http://example.com/`Robots.txt`

HTTP规范将`user-agent`定义为发送请求的东西（与接收请求的`服务器`相对）。严格来说，用户代理可以是请求网页的任何内容，包括搜索引擎抓取工具，Web浏览器或模糊的命令行 实用程序。

## 用户代理指令

在`Robots.txt`文件中，`user-agent`指令用于指定哪个爬网程序应遵守给定的规则集。该指令可以是通配符，用于指定规则适用于所有爬网程序：

    User-agent： *

或者它可以是特定爬虫的名称：

    User-agent：Googlebot

## 禁止指令

您应该通过一个或多个`disallow `指令来遵循用户代理行 ：

    User-agent：* 
    Disallow：/junk-page

上面的示例将阻止路径以`/junk-page`开头的所有URL ：

    http://example.com/junk-page 
    http://example.com/junk-page?usefulness=0 
    http://example.com/junk-page/whatever 

它不会阻止任何路径不以`/junk-page`开头的URL 。以下网址不会被 阻止：

    http://example.com/subdir/junk-page

这里的关键是`disallow`是一个简单的文本匹配。无论`Disallow：`之后出现什么都被视为一个简单的字符串（除了*和$之外，我将在下面提到）。将此字符串与URL的路径部分的开头（从域之后的第一个斜杠到URL的末尾的所有内容）进行比较，该字符串也被视为简单字符串。如果匹配，则会阻止该URL。如果他们不这样做，那就 不是。

## 允许指令

`Allow`指令不是原始标准的一部分，但现在所有主要搜索引擎都支持它。

您可以使用此伪指令指定禁止规则的例外，例如，如果您有一个要阻止的子目录，但希望抓取该子目录中的一个页面：

    User-agent：* 
    Allow：/nothing-good-in-here/except-this-one-page 
    Disallow：/nothing-good-in-here/

此示例将阻止以下 URL：

    http://example.com/nothing-good-in-here/
    http://example.com/nothing-good-in-here/somepage 
    http://example.com/nothing-good-in-here/otherpage 
    http://example.com/nothing-good-in-here/?x=y

但它不会阻止以下任何一种情况：

    http://example.com/nothing-good-in-here/except-this-one-page 
    http://example.com/nothing-good-in-here/except-this-one-page-because-i -said-so 
    http://example.com/nothing-good-in-here/except-this-one-page/that-is-really-a-directory 

同样，这是一个简单的文本匹配。将`Allow：`之后的文本与URL的路径部分的开头进行比较。如果它们匹配，即使在通常阻止它的其他地方禁止该页面，也将允许该页面。

## 通配符

所有主要搜索引擎也支持通配符运算符。这允许您在路径的一部分未知或可变时阻止页面。对于 例如：

    Disallow：/users/*/settings

*（星号）表示`匹配任何文本。`上述指令将阻止以下所有 URL：

    http://example.com/users/alice/settings 
    http://example.com/users/bob/settings 
    http://example.com/users/tinkerbell/settings 

小心！以上还将阻止以下URL（可能不是您想要的）：

    http://example.com/users/alice/extra/directory/levels/settings 
    http://example.com/users/alice/search?q=/settings 

字符串结束运算符
另一个有用的扩展是字符串结尾运算符：

    Disallow：/useless-page $

$表示URL必须在该点结束，该指令将阻止以下 URL：

    http://example.com/useless-page

但它不会阻止 以下任何一种情况：

    http://example.com/useless-pages-and-how-to-avoid-creating-them 
    http://example.com/useless-page/
    http://example.com/useless-page?a=b

## 阻止一切

您可能希望使用`Robots.txt`阻止所有暂存站点（稍后会详细介绍）或镜像站点。如果您有一个私人网站供少数知道如何找到它的人使用，那么您还希望阻止整个网站被抓取。

要阻止整个站点，请使用禁止后跟斜杠：

    User-agent：* 
    Disallow：/

## 允许一切

当您计划允许 所有内容时，我可以想到您可能选择创建`Robots.txt`文件的两个原因：

作为占位符，要向在网站上工作的任何其他人明确表示您允许一切都是故意的。

防止对`Robots.txt`的请求失败，以显示在请求日志中。

要允许整个站点，您可以使用空的禁令：

    User-agent：* 
    Disallow：

或者，您可以将`Robots.txt`文件留空，或者根本没有。爬行者会抓取所有内容，除非你告诉他们不要 。

## Sitemap 指令

虽然它是可选的，但许多`Robots.txt`文件都包含一个`sitemap` 指令：

网站地图：http：//example.com/sitemap.xml

这指定了站点地图文件的位置。站点地图是一种特殊格式的文件，列出了您要抓取的所有网址。如果您的站点具有XML网站地图，则最好包含此指令。

## 使用 `Robots.txt`的常见错误

我看到很多很多不正确的`Robots.txt`用法。其中最严重的是尝试使用该文件保密某些目录或尝试使用它来阻止恶意爬虫。

滥用`Robots.txt`的最严重后果是意外地将您的整个网站隐藏在抓取工具中。密切关注这些 事情。

### 当你去制作时忘记隐藏

所有暂存站点（尚未隐藏在密码后面）都应该包含`Robots.txt`文件，因为它们不适合公众查看。但是当您的网站上线时，您会希望每个人都能看到它。不要忘记删除或编辑此 文件。

否则，整个实时网站将从搜索结果中消失。

    User-agent：* 
    Disallow：/

您可以在测试时检查实时`Robots.txt`文件，或进行设置，这样您就不必记住这一额外步骤。使用摘要式身份验证等简单协议将登台服务器置于密码之后。然后，您可以为登台服务器提供您打算在实际站点上部署的相同`Robots.txt`文件。部署时，只需复制所有内容即可。

### 试图阻止敌对爬虫

我见过`Robots.txt`文件试图明确阻止已知的恶意抓取程序，如下所示：

    User-agent：DataCha0s/2.0 
    Disallow：/
    User-agent：ExtractorPro 
    Disallow：/
    User-agent：EmailSiphon 
    Disallow：/
    User-agent：EmailWolf 1.00 
    Disallow：/

这就像在汽车仪表板上留下一张纸条说：`亲爱的小偷：请不要偷这辆车。 谢谢！`

这毫无意义。这就像在汽车仪表板上留下一张纸条说：`亲爱的小偷：请不要偷这辆车。 谢谢！`

`Robots.txt`完全是自愿的，像搜索引擎这样的礼貌爬虫会遵守它。敌意爬行器，如电子邮件收割机，不会。爬虫没有义务遵守`Robots.txt`中的指南，但主要的选择是这样做的。

如果您正在尝试阻止错误的抓取工具，请使用用户代理阻止或IP阻止 。

### 试图保持目录的秘密

如果您要保留对公众隐藏的文件或目录，请不要将它们全部列在`Robots.txt`中，如下所示：

    User-agent：* 
    Disallow：/secret-stuff/
    Disallow：/compromising-photo.jpg 
    Disallow：/big-list-of-plaintext-passwords.csv

出于显而易见的原因，这将弊大于利。它为敌对爬虫提供了一种快速，简便的方法来查找您不希望他们找到的文件 。

这就像在你的车上留下一张纸条上写着：`亲爱的小偷：请不要看着隐藏在这辆车的杂物箱中的标有’紧急现金’的黄色信封。 谢谢！`

保持目录隐藏的唯一可靠方法是将其置于密码之后。如果你绝对不能把它放在密码后面，这里有三个创可贴解决方案。

1. 基于目录名称的前几个字符进行阻止。
如果目录是`/xyz-secret-stuff/`，则将其阻塞如下：


    Disallow：/xyz-

2. 阻止机器人元标记
将以下内容添加到HTML代码中：


    <meta name =`robots`content =`noindex，nofollow`>

3. 使用X-Robots-Tag标头阻止。
将这样的内容添加到目录的.htaccess文件中：


    标题集X-Robots-Tag`noindex，nofollow`

同样，这些是创可贴解决方案，这些都不是实际安全的替代品。如果确实需要保密，那么它确实需要在密码后面。

## 意外阻止不相关的页面

假设您需要阻止该 页面：

    http://example.com/admin

还有 目录中的所有内容：

    http://example.com/admin/

显而易见的方法是这样做 ：

    Disallow：/admin

这会阻止你想要的东西，但现在你也不小心阻止了关于宠物护理的文章页面：

    http://example.com/administer-medication-to-your-cat-the-easy-way.html

本文将与您实际尝试 阻止的页面一起从搜索结果中消失。

是的，这是一个人为的例子，但我已经看到这种事情发生在现实世界中。最糟糕的是，它通常会被忽视很长一段时间。

阻止/admin和/admin/而不阻塞任何其他内容的最安全方法是使用两个单独的行：

    Disallow：/admin $ 
    Disallow：/admin/

请记住，美元符号是一个字符串结尾的运算符，表示`URL必须在此处结束。`该指令将匹配/admin但不匹配/管理。

## 试图将`Robots.txt`放在子目录中

假设您只能控制一个巨大网站的一个子目录。

    http://example.com/userpages/yourname/

如果您需要阻止某些页面，可能会尝试添加`Robots.txt`文件，如下所示：

    http://example.com/userpages/yourname/`Robots.txt`

这不起作用，该文件将被忽略。您可以放置​​`Robots.txt`文件的唯一位置是站点根目录。

如果您无权访问站点根目录，则无法使用`Robots.txt`。一些替代选项是使用机器人元标记来阻止页面。或者，如果您可以控制`.htaccess`文件（或等效文件），则还可以使用X-Robots-Tag标头阻止页面。

## 尝试定位特定的子域

假设您有一个包含许多不同子域的站点：

    http://example.com/
    http://admin.example.com/
    http://members.example.com/
    http://blog.example.com/
    http://store.example.com/

您可能想要创建单个`Robots.txt`文件，然后尝试阻止它的子域，如下所示：

http://example.com/`Robots.txt` 

     User-agent：* 
    Disallow：admin.example.com 
    Disallow：members.example.com

这不起作用，无法在`Robots.txt`文件中指定子域（或域）。给定的`Robots.txt`文件仅适用于从中加载的子域 。

那么有没有办法阻止某些子域？是。要阻止某些子域而不阻止其他子域，您需要提供来自不同子域的不同`Robots.txt`文件。

这些`Robots.txt`文件会阻止所有内容：

    http://admin.example.com/`Robots.txt` 
    http://members.example.com/`Robots.txt` 
    
---
    
    
    User-agent：* 
    Disallow：/

这些将允许一切：

    http://example.com/
    http://blog.example.com/
    http://store.example.com/

---
    
    User-agent：* 
    Disallow：

## 使用不一致的类型情况

路径区分大小写。

    Disallow：/acme/

不会阻止`/Acme/`或 `/ACME/`。

如果你需要全部阻止它们，你需要为每个禁用一行：

    Disallow：/acme/
    Disallow：/Acme/
    Disallow：/ACME/

## 忘记了用户代理线

所述用户代理线是使用`Robots.txt`关键的。在任何允许或禁止之前，文件必须具有用户代理行。如果整个文件看起来像这样：

    Disallow：/this 
    Disallow：/that 
    Disallow：/what

实际上什么都不会被阻止，因为顶部没有用户代理行。该文件必须为：

    User-agent：* 
    Disallow：/this 
    Disallow：/that 
    Disallow：/whatever

## 其他用户代理陷阱

使用不正确的用户代理还存在其他缺陷。假设您有三个目录需要为所有抓取工具阻止，还有一个页面应该仅在Google上明确允许。显而易见（但不正确）的方法可能是尝试这样的事情 ：

    User-agent：* 
    Disallow：/admin/
    Disallow：/private/
    Disallow：/dontcrawl/
    User-agent：Googlebot 
    Allow：/dontcrawl/exception

此文件实际上允许Google抓取网站上的所有内容。Googlebot（以及大多数其他抓取工具）只会遵守更具体的用户代理行下的规则，并会忽略所有其他规则。在此示例中，它将遵守`User-agent：Googlebot`下的规则，并将忽略`User-agent： *` 下的规则。

要实现此目标，您需要为每个用户代理块重复相同的禁止规则，如下所示：

    User-agent：* 
    Disallow：/admin/
    Disallow：/private/
    Disallow：/dontcrawl/
    User-agent：Googlebot 
    Disallow：/admin/
    Disallow：/private/
    Disallow：/dontcrawl/
    Allow：/dontcrawl/exception

## 忘记路径中的主要斜线
假设您要阻止该 URL：

    http://example.com/badpage

你有以下（不正确的）`Robots.txt` 文件：

    User-agent：* 
    Disallow：错误页面

这根本不会阻止任何事情，路径必须以斜杠开头。如果没有，它永远不会匹配任何东西。阻止URL的正确方法 是：

    User-agent：* 
    Disallow：/badpage

## 使用 `Robots.txt`的提示

既然您知道如何不将敌对抓取工具发送到您的秘密内容或从搜索结果中消失您的网站，这里有一些提示可以帮助您改进`Robots.txt`文件。做得好不会提高你的排名（这是战略搜索引擎优化和内容的用途），但至少你会知道爬虫正在找到你想要他们找到的东西。

## 竞争允许和不允许

allow指令用于指定disallow规则的例外。disallow规则阻塞整个目录（例如），allow规则取消阻止该目录中的某些URL。这提出了一个问题，如果给定的URL可以匹配两个规则中的任何一个，爬虫如何决定使用哪个？

并非所有抓取工具都以完全相同的方式处理竞争允许和禁止，但Google优先考虑路径较长的规则（就字符数而言）。如果两个路径长度相同，则allow优先于disallow。例如，假设`Robots.txt`文件 是：

    User-agent：* 
    Allow：/baddir/goodpage 
    Disallow：/baddir/

路径`/baddir/goodpage`长度为16个字符，路径`/baddir/`长度仅为8个字符。在这种情况下，允许胜过 不允许。

将 允许以下URL ：

    http://example.com/baddir/goodpage 
    http://example.com/baddir/goodpagesarehardtofind 
    http://example.com/baddir/goodpage?x=y

以下内容将被 阻止：

    http://example.com/baddir/
    http://example.com/baddir/otherpage

现在考虑以下示例：

    User-agent：* 
    Aloow：/某些
    Disallow：/*页面

这些指令会阻止以下 URL吗？

    http://example.com/somepage

是。路径`/some`长度为5个字符，路径`/* page`长度为6个字符，因此disallow获胜。允许被忽略，URL将被阻止。

## 阻止特定的查询参数

假设您要阻止包含查询参数`id`的所有URL，例如 ：

    http://example.com/somepage?id=123 
    http://example.com/somepage?a=b&id=123

你可能想做这样的事情 ：

    Disallow：/* id =

这将阻止您想要的URL，但也会阻止以 `id` 结尾的任何其他查询参数：

    http://example.com/users?userid=a0f3e8201b 
    http://example.com/auction?num=9172&bid=1935.00

那么如何在不阻止`用户ID`或 `出价`的情况下阻止`id `？

如果您知道`id`将始终是第一个参数，请使用问号，如下 所示：

    Disallow：/*？id =

该指令将阻止：

    http://example.com/somepage?id=123

但它不会阻止：

    http://example.com/somepage?a=b&id=123

如果您知道`id`永远不会是第一个参数，请使用＆符号，如下 所示：

    Disallow：/*＆id =

该指令将阻止：

    http://example.com/somepage?a=b&id=123

但它不会阻止：

    http://example.com/somepage?id=123

最安全的方法是 两者兼顾：

    Disallow：/*？id = 
    Disallow：/*＆id =

没有可靠的方法来匹配两条线。

## 阻止包含不安全字符的URL

假设您需要阻止包含不安全URL的字符的URL，可能发生这种情况的一种常见情况是服务器端模板代码意外暴露给Web。对于 例如：

    http://example.com/search?q=<% var_name％>

如果您尝试像这样阻止该URL，它将无法 工作：

    User-agent：* 
    Disallow：/search？q=<％var_name％>

如果您在Google的`Robots.txt`测试工具（在Search Console中提供）中测试此指令，您会发现它不会阻止该网址。为什么？因为该指令实际上是根据 URL 检查的：

    http://example.com/search?q=%3C%%20var_name%20%%3E

所有Web 用户代理（包括抓取工具）都会自动对任何不符合URL安全的字符进行URL编码。这些字符包括：空格，小于或大于符号，单引号， 双引号和非ASCII 字符。

阻止包含不安全字符的URL的正确方法是阻止转义版本：

    User-agent：* 
    Disallow：/search？q =％3C %% 20var_name％20 %% 3E

获取URL的转义版本的最简单方法是单击浏览器中的链接，然后从地址 字段中复制并粘贴URL 。

## 如何匹配美元符号

假设您要阻止包含美元符号的所有网址，例如 ：

    http://example.com/store?price=$10

以下内容 不起作用：

    Disallow：/* $

该指令实际上会阻止站点上的所有内容。当在指令末尾使用时，美元符号表示`URL在此处结束。`因此，上面将阻止路径以斜杠开头的每个URL，后跟零个或多个字符，后跟URL的结尾。此规则适用于任何有效的URL。为了解决这个问题，诀窍是在美元符号后添加一个额外的星号，如下所示：

    Disallow：/* $ *

在这里，美元符号不再位于路径的尽头，因此它失去了它的特殊含义。该指令将匹配包含文字美元符号的任何URL。请注意，最终星号的唯一目的是防止美元符号成为最后一个 字符。

## 补充

有趣的事实：谷歌在进行语义搜索的过程中，通常会正确地解释拼写错误或格式错误的指令。例如，Google会在没有投诉的情况下接受以下任何内容：

    UserAgent：* 
    Disallow /this 
    Dissalow：/that

这并不意味着你应该忽略指令的格式和拼写，但如果你确实犯了错误，谷歌通常会让你逃脱它。但是，其他爬虫可能 不会。

人们经常在`Robots.txt`文件中使用尾随通配符。这是无害的，但它也没用; 我认为这是糟糕的形式。

对于例如：

    Disallow：/somedir/*

与以下内容完全相同 ：

    Disallow：/somedir/

当我看到这个时，我想，`这个人不明白`Robots.txt`是如何工作的。`我看到它很多。

## 概要

请记住，`Robots.txt`必须位于根目录中，必须以用户代理行开头，不能阻止恶意爬虫，也不应该用于保密目录。使用此文件的许多困惑源于人们期望它比它更复杂的事实。
