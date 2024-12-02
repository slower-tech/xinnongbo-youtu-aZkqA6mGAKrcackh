
![](https://img2024.cnblogs.com/blog/35695/202411/35695-20241130173204762-1287293165.png)


2024年10月8日，微软 .NET 官方博客发布了一篇博文 [Introducing Microsoft.Extensions.AI Preview – Unified AI Building Blocks for .NET](https://github.com)，给 .NET 开发者带来了一个小惊喜，.NET 类库将增加一个统一的调用 AI 服务的抽象接口层。



> Microsoft.Extensions.AI is a set of core .NET libraries developed in collaboration with developers across the .NET ecosystem, including Semantic Kernel. These libraries provide a unified layer of C\# abstractions for interacting with AI services, such as small and large language models (SLMs and LLMs), embeddings, and middleware.


这个小惊喜对我们的 AI 之旅是场及时雨。


今年3月，我们准备尝试基于 Semantic Kernel 使用通义千问大模型开发 AI 应用。当时由于阿里云模型服务灵积 DashScope （后来阿里云百炼取代了灵积）没有提供 .NET SDK，我们自己实现了 [博文](https://github.com)。


在实现 DashScope .NET SDK 之后，由于 Microsoft.Extensions.AI 还没出生，为了能在 Semantic Kernel 中使用 DashScope SDK，我们还实现了 [博文](https://github.com)。


* SemanticKernel.DashScope 实现了3个接口：IChatCompletionService 与 ITextGenerationService 以及 ITextEmbeddingGenerationService
* KernelMemory.DashScope 实现了2个接口：ITextEmbeddingGenerator 与 ITextEmbeddingGenerator


实现这5个接口本身不是什么问题，问题是我们在实现与 Semantic Kernel 毫无关系的 DashScope SDK 时却要对 Semantic Kernel 与 Kernel Memory 产生依赖，这种纠缠以后将成为一种苦难。


现在救星出现了，有了 Microsoft.Extensions.AI，DashScope SDK 只需实现 Microsoft.Extensions.AI 的接口，从此与 Semantic Kernel 身处各自的世界，不再有任何牵连。


那如何让两个不同世界的 DashScope 与 Semantic Kernel 一起工作呢？第三者登场了，两情相悦的二人世界不需要第三者，但互不依赖的两个代码组件需要第三者——调用它们的应用开发者，但开发者要打通这两个世界需要有个前提，Semantic Kernel 要通过 Microsoft.Extensions.AI 的抽象接口调用 AI 服务，所以在同一天，微软 Semantic Kernel 官方博客也发布了一篇博文 [Microsoft.Extensions.AI: Simplifying AI Integration for .NET Partners](https://github.com):[slowerssr加速器](https://slowerss.com) ，Semantic Kernel 支持 Microsoft.Extensions.AI 已是板上钉钉。



> When Microsoft.Extensions.AI moves from preview to general availability (GA), we will enable this package in Semantic Kernel .NET


为了感谢这场及时雨，我们抢在 Semantic Kernel 之前，在 Microsoft.Extensions.AI 还处于预览版之际，在 DashScope SDK 中实现了对 Microsoft.Extensions.AI 的支持，实现了 IChatClient 与 IEmbeddingGenerator 接口，并且在11月27日发布了 nuget 包 [Cnblogs.DashScope.AI](https://github.com)


具体实现代码见 github 上的 PR [https://github.com/cnblogs/dashscope\-sdk/pull/51](https://github.com)


接下来通过示例代码简单体验一下通过 Microsoft.Extensions.AI 的接口与通义千问大模型对话。


准备 .NET 控制台项目



```
dotnet new console
dotnet add package Cnblogs.DashScope.AI --prerelease

```

在下面的 C\# 代码中调用 Microsoft.Extensions.AI 中 IChatClient 接口的 CompleteAsync 方法与通义千问大模型 qwen\-coder\-turbo 对话



```
using Cnblogs.DashScope.Core;
using Microsoft.Extensions.AI;

var apiKey = "sk-xxxxxx";
var modelId = "qwen-coder-turbo";
IChatClient client = new DashScopeClient(apiKey).AsChatClient(modelId);
var response = await client.CompleteAsync("请用一两句话谈谈你对博客园AI之旅的看法");
Console.WriteLine(response.Message);

```

运行结果如下：



```
博客园的AI之旅是一次令人兴奋的技术探索，它展示了人工智能如何在内容创作、个性化推荐和用户互动等方面提升用户体验。这次旅程不仅推动了技术的进步，也为内容创作者和读者带来了新的可能性和便利。

```

由于目前 Semantic Kernel 还不支持 Microsoft.Extensions.AI，只能这样直接调用 IChatClient 接口简单体验一下。


等 Semantic Kernel 支持 Microsoft.Extensions.AI，就不再需要我们之前实现的 SemanticKernel.DashScope 与 KernelMemory.DashScope。


今天就简单写到这，感谢您关注园子的 AI 之旅。


