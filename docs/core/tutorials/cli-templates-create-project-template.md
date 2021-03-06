---
title: 创建 dotnet new 项目模板
description: 了解如何创建 dotnet new 命令的项目模板。
author: adegeo
ms.date: 12/11/2020
ms.topic: tutorial
ms.author: adegeo
ms.openlocfilehash: ed40cfd303c70c7b8f198a0f5b593bf1e1ebeaf8
ms.sourcegitcommit: d0990c1c1ab2f81908360f47eafa8db9aa165137
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/15/2020
ms.locfileid: "97513128"
---
# <a name="tutorial-create-a-project-template"></a>教程：创建项目模板

使用 .NET，可以创建和部署可生成项目、文件甚至资源的模板。 本教程是系列教程的第二部分，介绍如何创建、安装和卸载用于 `dotnet new` 命令的模板。

在本系列的这一部分中，你将了解如何：

> [!div class="checklist"]
>
> * 创建项目模板的资源
> * 创建模板配置文件夹和文件
> * 从文件路径安装模板
> * 测试项模板
> * 卸载项模板

## <a name="prerequisites"></a>先决条件

* 完成本系列教程的[第 1 部分](cli-templates-create-item-template.md)。
* 打开终端并导航到 working\templates  文件夹。

## <a name="create-a-project-template"></a>创建项目模板

项目模板生成可立即运行的项目，使用户可以轻松地使用一组有效的代码。 .NET 包含一些项目模板，例如控制台应用程序或类库。 在本例中，你将创建一个启用 C# 9.0 并生成 `async main` 入口点的新控制台项目。

在终端中，导航到 working\templates  文件夹，并创建一个名为“consoleasync”  的新子文件夹。 进入子文件夹，并运行 `dotnet new console` 以生成标准控制台应用程序。 将编辑此模板生成的文件以创建新模板。

```console
working
└───templates
    └───consoleasync
            consoleasync.csproj
            Program.cs
```

## <a name="modify-programcs"></a>修改 Program.cs

打开 program.cs  文件。 控制台项目不使用异步入口点，我们来添加它。 将代码更改为以下内容并保存文件。

```csharp
using System;
using System.Threading.Tasks;

namespace consoleasync
{
    class Program
    {
        static async Task Main(string[] args)
        {
            await Console.Out.WriteAsync("Hello World with C# 9.0!");
        }
    }
}
```

## <a name="modify-consoleasynccsproj"></a>修改 consoleasync.csproj

将项目使用的 C# 语言版本更新到 9.0 版。 编辑 consoleasync.csproj  文件并将 `<LangVersion>` 设置添加到`<PropertyGroup>` 节点。

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net5.0</TargetFramework>

    <LangVersion>9.0</LangVersion>

  </PropertyGroup>
  
</Project>
```

## <a name="build-the-project"></a>生成项目

在完成项目模板之前，应对其进行测试，确保它能够正确编译和运行。

在终端中，运行以下命令。

```dotnetcli
dotnet run
```

将获得以下输出。

```console
Hello World with C# 9.0!
```

可以使用 `dotnet run` 删除已创建的 obj  和 bin  文件夹。 删除这些文件可确保你的模板仅包含与模板相关的文件，而不包含生成操作产生的任何文件。

现在你已经创建了模板的内容，需要在模板的根文件夹中创建模板配置。

## <a name="create-the-template-config"></a>创建模板配置

模板在 .NET 中通过模板根目录中的特殊文件夹和配置文件进行识别。 在本教程中，你的模板文件夹位于 working\templates\consoleasync  。

创建模板时，除特殊配置文件夹外，模板文件夹中的所有文件和文件夹都作为模板的一部分包含在内。 此配置文件夹名为“.template.config”  。

首先，创建一个名为“.template.config”  的新子文件夹，然后进入该文件夹。 然后，创建一个名为“template.json”  的新文件。 文件夹结构应如下所示。

```console
working
└───templates
    └───consoleasync
        └───.template.config
                template.json
```

使用你喜爱的文本编辑器打开 template.json  并粘贴以下 json 代码，然后保存。

```json
{
  "$schema": "http://json.schemastore.org/template",
  "author": "Me",
  "classifications": [ "Common", "Console", "C#9" ],
  "identity": "ExampleTemplate.AsyncProject",
  "name": "Example templates: async project",
  "shortName": "consoleasync",
  "tags": {
    "language": "C#",
    "type": "project"
  }
}
```

此配置文件包含模板的所有设置。 可以看到基本设置，例如 `name` 和 `shortName`，除此之外，还有一个设置为 `project` 的 `tags/type` 值。 这会将模板指定为项目模板。 你创建的模板类型不存在限制。 `item` 和 `project` 值是 .NET 建议使用的通用名称，便于用户轻松筛选正在搜索的模板类型。

`classifications` 项表示你在运行 `dotnet new` 并获取模板列表时看到的“标记”  列。 用户还可以根据分类标记进行搜索。 不要将 json 文件中的 `tags` 属性与 `classifications` 标记列表混淆。 它们虽然具有类似的名称，但截然不同。 template.json  文件的完整架构位于 [JSON 架构存储](http://json.schemastore.org/template)。 有关 template.json  文件的详细信息，请参阅 [dotnet 创建模板 wiki](https://github.com/dotnet/templating/wiki)。

现在你已有一个有效的 .template.config/template.json  文件，可以安装模板了。 在安装模板之前，请务必删除无需在模板中包含的任何额外文件夹和文件，例如 bin  或 obj  文件夹。 在终端中，导航到 consoleasync  文件夹，并运行 `dotnet new -i .\` 以安装位于当前文件夹的模板。 如果使用的是 Linux 或 macOS 操作系统，请使用正斜杠：`dotnet new -i ./`。

此命令输出安装的模板列表，其中应包括你的模板。

```dotnetcli
dotnet new -i .\
```

将获得类似于下面的输出。

```console
Usage: new [options]

Options:
  -h, --help          Displays help for this command.
  -l, --list          Lists templates containing the specified name. If no name is specified, lists all templates.

... cut to save space ...

Templates                                         Short Name               Language          Tags
--------------------------------------------      -------------------      ------------      ----------------------
Console Application                               console                  [C#], F#, VB      Common/Console
Example templates: async project                  consoleasync             [C#]              Common/Console/C#9
Class library                                     classlib                 [C#], F#, VB      Common/Library
WPF Application                                   wpf                      [C#], VB          Common/WPF
```

### <a name="test-the-project-template"></a>测试项目模板

现在你已安装了项目模板，可对其进行测试。

1. 导航到 test  文件夹

1. 使用以下命令创建一个新的控制台应用程序，该命令生成可使用 `dotnet run` 命令轻松测试的工作项目。

    ```dotnetcli
    dotnet new consoleasync
    ```

    将获得以下输出。

    ```console
    The template "Example templates: async project" was created successfully.
    ```

1. 请使用以下命令运行项目。

    ```dotnetcli
    dotnet run
    ```

    将获得以下输出。

    ```console
    Hello World with C# 9.0!
    ```

祝贺你！ 你已使用 .NET 创建并部署了项目模板。 为准备学习本系列教程的下一部分，必须卸载已创建的模板。 确保同时删除 test  文件夹中的所有文件。 这将回到干净状态，为本教程的下一个主要部分做好准备。

### <a name="uninstall-the-template"></a>卸载模板

由于模板是使用文件路径安装的，因此，必须使用绝对  文件路径将其卸载。 可以通过运行 `dotnet new -u` 命令看到已安装的模板列表。 你的模板应列在最后。 使用列出的 `Uninstall Command` 卸载模板。

```dotnetcli
dotnet new -u
```

将获得类似于下面的输出。

```console
Template Instantiation Commands for .NET Core CLI

Currently installed items:
  Microsoft.DotNet.Common.ProjectTemplates.2.2
    Details:
      NuGetPackageId: Microsoft.DotNet.Common.ProjectTemplates.2.2
      Version: 1.0.2-beta4
      Author: Microsoft
    Templates:
      Class library (classlib) C#
      Class library (classlib) F#
      Class library (classlib) VB
      Console Application (console) C#
      Console Application (console) F#
      Console Application (console) VB
    Uninstall Command:
      dotnet new -u Microsoft.DotNet.Common.ProjectTemplates.2.2

... cut to save space ...

  C:\Test\templatetutorial\working\templates\consoleasync
    Templates:
      Example templates: async project (consoleasync) C#
    Uninstall Command:
      dotnet new -u C:\working\templates\consoleasync
```

若要卸载所创建的模板，请运行输出中显示的 `Uninstall Command`。

```dotnetcli
dotnet new -u C:\working\templates\consoleasync
```

## <a name="next-steps"></a>后续步骤

在本教程中，你创建了一个项目模板。 若要了解如何将项模板和项目模板打包为易于使用的文件，请继续学习本教程系列。

> [!div class="nextstepaction"]
> [创建模板包](cli-templates-create-template-pack.md)
