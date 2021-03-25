---
type: docs
title: "Azure Configuration Manager"
linkTitle: "Azure"
weight: 30000
date: 2021-03-22
description: >
  Azure Configuration Manager 调研
---



Azure Configuration Manager



## 概述

https://docs.microsoft.com/en-us/azure/azure-app-configuration/overview

### 什么是Azure App Configuration？

Azure App Configuration 提供了一个集中管理应用程序设置和功能标志的服务。现代程序，尤其是在云中运行的程序，通常具有许多分布式的组件。将配置设置分散在这些组件中可能会导致应用程序部署期间出现难以解决的错误。使用 App Configuration 来存储应用程序的所有设置，并在一个地方保护它们的访问权。

基于云的应用程序通常运行在多个区域的多个虚拟机或容器上，并使用多个外部服务。在分布式环境中创建一个健壮和可扩展的应用程序是一个重大的挑战。

各种编程方法论帮助开发人员处理构建应用程序的日益复杂的问题。例如，"十二因素应用 "描述了许多经过良好测试的架构模式和最佳实践，用于云应用。本指南的一个关键建议是将配置与代码分开。应用程序的配置设置应该保存在其可执行文件的外部，并从其运行时环境或外部源读入。



虽然任何应用程序都可以使用 App Configuration，但以下示例是受益于使用它的应用程序类型：

- 基于Azure Kubernetes Service、Azure Service Fabric或部署在一个或多个地域的其他容器化应用的微服务。
- 无服务器应用，其中包括Azure Functions或其他事件驱动的无状态计算应用。
- 连续部署管道

App配置提供了以下优势：

- 可在几分钟内完成设置的完全托管服务
- 灵活的 key 表示和映射
- 使用标签(label)进行标记
- 按时间点的设置重播
- 专门的用户界面用于功能标志（ feature flag）管理
- 两套配置在自定义尺寸上的比较
- 通过Azure管理的身份增强安全性
- 对静止和传输中的敏感信息进行加密。
- 与流行的框架进行原生集成

App 配置是对 Azure Key Vault 的补充，后者用于存储应用秘密。App Configuration 使得以下场景更容易实现。

- 对不同环境和地域的分级配置数据进行集中管理和分发。
- 动态更改应用程序设置，而无需重新部署或重新启动应用程序。
- 实时控制功能可用性

## 最佳实践

https://docs.microsoft.com/en-us/azure/azure-app-configuration/howto-best-practices

本文讨论了使用 Azure App 配置时的常见模式和最佳实践。

### key 的组织

App Configuration 提供了两种组织 key 的选项：

- Key prefixes / key前缀
- Labels / 标签

您可以使用其中一个或两个选项来分组key。

Key prefixes是 key 的开头部分。您可以通过在键的名称中使用相同的前缀来对一组键进行逻辑分组。前缀可以包含多个由分隔符（如`/`）连接的组件，类似于URL路径，形成一个命名空间。当您在一个 App Configuration 储存中保存许多应用程序、组件服务和环境的 key 时，这种层次结构非常有用。

需要牢记的一个重要事项是，key 是您的应用程序代码引用以检索相应设置的值的东西。key 不应该改变，否则每次发生这种情况你都要修改你的代码。

Lable / 标签是 key 的属性。它们用于创建 key 的变体。例如，你可以给一个 key 的多个版本分配标签。版本可能是迭代，环境，或者其他一些上下文信息。您的应用程序可以通过指定另一个标签来请求一组完全不同的 key 值。因此，在您的代码中，所有的 key 引用都保持不变。

### 键值构成
App Configuration 将所有存储 key 视为独立的实体。App Configuration 不会尝试推断键之间的任何关系，也不会根据键的层次结构来继承键值。然而，您可以通过使用标签加上您的应用程序代码中的适当配置堆叠来聚合多组键。

让我们来看一个例子。假设您有一个名为 Asset1 的设置，其值可能会根据开发环境而变化。你创建了一个名为 "Asset1 "的键，其中有一个空标签和一个名为 "Development" 的标签。在第一个标签中，你放了Asset1的默认值，在后一个标签中，你放了一个 "Development" 的特定值。

在你的代码中，你首先检索没有任何标签的键值，然后你第二次用 "Development" 标签检索同一组键值。当你第二次检索值时，之前的键值会被覆盖。.NET Core配置系统允许您将多组配置数据 "堆叠"在一起。如果一个键存在于多个集合中，则使用包含该键的最后一个集合。在现代编程框架（如.NET Core）中，如果您使用原生配置提供者来访问App Configuration，您就可以免费获得这种堆叠功能。下面的代码片段显示了您如何在.NET Core应用程序中实现堆叠：

```c#
// Augment the ConfigurationBuilder with Azure App Configuration
// Pull the connection string from an environment variable
configBuilder.AddAzureAppConfiguration(options => {
    options.Connect(configuration["connection_string"])
           .Select(KeyFilter.Any, LabelFilter.Null)
           .Select(KeyFilter.Any, "Development");
});
```



### 使用标签来实现不同环境的配置

https://docs.microsoft.com/en-us/azure/azure-app-configuration/howto-labels-aspnet-core

许多应用程序需要针对不同的环境使用不同的配置。假设一个应用程序有一个配置值，定义了其后端数据库要使用的连接字符串。应用程序开发人员使用的数据库与生产中使用的数据库不同。当应用程序从开发环境转移到生产环境时，应用程序使用的数据库连接字符串必须改变。

在 Azure App 配置中，您可以使用标签为同一键定义不同的值。例如，您可以为开发和生产定义具有不同值的单个键。您可以指定在连接到 App Configuration 时要加载哪个标签。

#### 用指定的标签加载配置值

默认情况下，Azure App Configuration 只加载没有标签的配置值。如果您已经为您的配置值定义了标签，您需要指定连接到 App Configuration 时要使用的标签。

在上一节中，您为开发环境创建了一个不同的配置值。您使用HostingEnvironment.EnvironmentName 变量来动态确定应用程序当前运行在哪个环境中。要了解更多信息，请参阅在ASP.NET Core中使用多个环境。

通过向Select方法传递环境名称，加载与当前环境对应的标签的配置值。

```c#
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();
                config.AddAzureAppConfiguration(options =>
                    options
                        .Connect(Environment.GetEnvironmentVariable("AppConfigConnectionString"))
                        // Load configuration values with no label
                        .Select(KeyFilter.Any, LabelFilter.Null)
                        // Override with any configuration values specific to current hosting env
                        .Select(KeyFilter.Any, hostingContext.HostingEnvironment.EnvironmentName)
                );
            })
            .UseStartup<Startup>());
```

Select方法被调用两次。第一次，它加载没有标签的配置值。然后，它加载与当前环境相对应的带有标签的配置值。这些特定于环境的值会覆盖任何没有标签的相应值。你不需要为每个键定义环境特定值。如果一个键没有对应于当前环境的带标签的值，它就会使用没有标签的值。



### 减少对App Configuration的请求

对 App Configuration 的过多请求可能导致节流或超额费用。要减少发出的请求次数:

- 增加刷新超时，特别是当您的配置值不经常变化时。使用 SetCacheExpiration 方法指定一个新的刷新超时。

- 监视单个哨兵键(*sentinel key*)，而不是监视单个键。只有在哨兵键发生变化时才刷新所有配置。请参见在 ASP.NET Core 应用程序中使用动态配置的示例。

- 使用 Azure 事件网格在配置更改时接收通知，而不是不断轮询任何更改。



## Keys and values

https://docs.microsoft.com/en-us/azure/azure-app-configuration/concept-key-value

Azure App Configuration 将配置数据存储为键值。键值是开发人员使用的应用程序设置的一种简单而灵活的表示。

### keys

Key 作为 key-value 的标识符，用于存储和检索相应的值。通常的做法是通过使用字符分隔符（如`/`或`:`）将 key 组织到一个层次化的 namespace 中。请使用最适合您的应用程序的约定。App Configuration 将 key 作为一个整体来处理。它不会解析 key 来找出它们的名称是如何结构的，也不会对它们执行任何规则。

下面是一个基于组件服务的层次结构的 key 名字的例子：

```
AppName:Service1:ApiEndpoint    
AppName:Service2:ApiEndpoint
```

在应用程序框架内使用配置数据可能会决定键值的特定命名方案。例如，Java 的 Spring Cloud 框架定义了为 Spring 应用程序提供设置的环境资源。这些资源由包括应用程序名称和配置文件的变量进行参数化。与Spring Cloud相关的配置数据的键通常以这两个元素开始，并由分隔符分隔。

存储在 App 配置中的键是区分大小写的、基于unicode的字符串。在 App Configuration 存储中，键 app1 和 App1 是不同的。当您在应用程序中使用配置设置时，请记住这一点，因为某些框架处理配置键时不区分大小写。我们不建议使用大小写来区分键。

您可以在键名中使用除 `%` 以外的任何 unicode 字符。key name 也不能是`.`或`..`。一个键值的组合大小限制为10KB。这个限制包括键、值和所有相关的可选属性中的所有字符。在这个限制内，你可以为键设置许多层次。

#### 设计 key namespace

命名配置数据使用的 key 一般有两种方法：扁平式（flat）或分层式（hierarchical）。从应用使用的角度来看，这些方法是相似的，但分层命名提供了一些优势。

- 更容易阅读。层次化键名中的定界符就像句子中的空格一样。它们还提供了词与词之间的自然分隔。
- 更容易管理。键名层次结构表示配置数据的逻辑组。
- 更容易使用。编写一个在层次结构中模式化匹配键的查询，只检索一部分配置数据，比较简单。此外，许多较新的编程框架对分层配置数据有原生支持，这样您的应用程序就可以利用特定的配置集。

你可以用很多方式来分层组织 App Configuration 中的键。把这样的键看作是URI。每个分层键都是由一个或多个组件组成的资源路径，这些组件通过定界符连接在一起。根据您的应用程序、编程语言或框架的需要，选择使用什么字符作为定界符。在 App 配置中对不同的键使用多个定界符。

#### 给 keys 打标签

App Configuration 中的键值可以选择拥有标签属性。标签用于区分具有相同键值的键值。具有标签 A 和 B 的键 app1 在 App Configuration 存储中形成两个独立的键。默认情况下，键值没有标签。要显式引用一个没有标签的键值，请使用 `\0`（URL编码为`%00`）。

标签提供了一种方便的方式来创建键的变体。标签的一个常见用途是为同一个键值指定多个环境。

```
   Key = AppName:DbEndpoint & Label = Test
   Key = AppName:DbEndpoint & Label = Staging
   Key = AppName:DbEndpoint & Label = Production
```

#### Key-value 的版本

使用标签来创建一个键值的多个版本。例如，您可以在标签中输入应用程序版本号或 Git 提交 ID，以识别与特定软件构建相关的键值。

> 注意：
>
> 如果您要查找变更版本，App配置会自动保存过去一定时间内发生的键值的所有变更。详情请看时间点快照。

#### 查询 key-value

每个键值由其 key 加 label 唯一标识，该标签可以是`\0`。您可以通过指定模式来查询 App 配置商店的键值。App 配置商店将返回与模式相匹配的所有键值，包括其相应的值和属性。在 REST API 调用 App Configuration 时使用以下 key 模式:

| Key                         | Description                                                |
| :-------------------------- | :--------------------------------------------------------- |
| `key` is omitted or `key=*` | Matches all keys                                           |
| `key=abc`                   | Matches key name **abc** exactly                           |
| `key=abc*`                  | Matches key names that start with **abc**                  |
| `key=abc,xyz`               | Matches key names **abc** or **xyz**. Limited to five CSVs |

您也可以包含以下标签模式:

| Label                           | Description                                            |
| :------------------------------ | :----------------------------------------------------- |
| `label` is omitted or `label=*` | Matches any label, which includes `\0`                 |
| `label=%00`                     | Matches `\0` label                                     |
| `label=1.0.0`                   | Matches label **1.0.0** exactly                        |
| `label=1.0.*`                   | Matches labels that start with **1.0.**                |
| `label=%00,1.0.0`               | Matches labels `\0` or **1.0.0**, limited to five CSVs |

>  注意：
>
> `*`,` ,` , 和`\`是查询中的保留字符。如果您的键名或标签中使用了保留字符，您必须在查询中使用  ` \{Reserved Character}` 来转义它。

### values

分配给键的值也是unicode字符串。你可以对值使用所有的unicode字符。

### 使用 content-type

App Configuration中的每个 key-value 都有一个 content-type 属性。您可以选择性地使用此属性来存储有关键值中的值的类型的信息，以帮助您的应用程序正确处理它。您可以为内容类型使用任何格式。App Configuration 使用媒体类型（也称为 MIME 类型）来处理内置数据类型，如特征标志、Key Vault 引用和 JSON 键值。

## Point-in-time snapshot

https://docs.microsoft.com/en-us/azure/azure-app-configuration/concept-point-time-snapshot

Azure App 配置维护对键值所做的更改记录。此记录提供了键值更改的时间线。您可以重建任何键值的历史，并在键值历史期间（免费层商店为 7 天，标准层商店为 30 天）的任何时刻提供其过去的值。使用此功能，您可以 "时间旅行 "回溯并检索旧的键值。例如，您可以恢复最近一次部署之前使用的配置设置，以便将应用程序回滚到以前的配置。



