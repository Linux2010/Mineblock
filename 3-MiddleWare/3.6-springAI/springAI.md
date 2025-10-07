# Spring AI 快速入门：构建你的第一个AI应用

## 引言

在人工智能技术飞速发展的今天，越来越多的开发者希望将AI能力集成到自己的应用程序中。然而，对于习惯了传统Web开发的Java开发者来说，直接调用AI模型API往往意味着需要处理复杂的认证、数据格式转换、错误处理等问题。有没有一种更简单、更符合Spring生态的方式来集成AI能力呢？

答案是肯定的——Spring AI应运而生。作为Spring生态系统的新成员，Spring AI为Java开发者提供了一套简洁、统一的API来集成各种AI模型，让你能够专注于业务逻辑而不是底层实现细节。

## 什么是Spring AI？

Spring AI是一个AI工程领域的应用程序框架，它的目标是将Spring生态系统的设计原则（如可移植性和模块化设计）应用于人工智能领域。简单来说，它就像Spring Boot简化了Web应用开发一样，Spring AI简化了AI应用的开发。

Spring AI的核心优势包括：
- **统一的API**：无论你使用OpenAI、Azure OpenAI还是其他AI服务提供商，都可以使用相同的API
- **模块化设计**：通过starter依赖，你可以轻松集成不同的AI服务
- **Spring原生**：与Spring Boot无缝集成，支持自动配置
- **易于测试**：提供了Mock工具，方便进行单元测试

## 快速开始：创建你的第一个Spring AI应用

让我们通过一个简单的示例来演示如何使用Spring AI创建一个AI聊天应用。

### 环境准备

在开始之前，请确保你已经安装了以下工具：
- JDK 17或更高版本
- Maven 3.8或更高版本
- 一个AI服务提供商的API密钥（我们以OpenAI为例）

### 1. 创建Spring Boot项目

你可以通过Spring Initializr网站（https://start.spring.io/）创建一个新的Spring Boot项目，选择以下依赖：
- Spring Web
- Spring AI OpenAI (如果你使用的是OpenAI)

或者，你也可以使用Spring CLI创建项目：

```bash
spring init --dependencies=web,ai-openai --name=spring-ai-demo spring-ai-demo
```

### 2. 添加依赖

如果你是手动创建项目，需要在[pom.xml](file:///Users/kylin/doc/ITblock/2-LeetCode/DataStruct/%E5%B0%9A%E7%A1%85%E8%B0%B7%E9%9F%A9%E9%A1%BA%E5%B9%B3%E8%80%81%E5%B8%88%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E4%BA%AB/Algorithm/pom.xml)中添加Spring AI的依赖：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

注意：Spring AI目前还处于预览阶段，版本号可能会有所变化。

### 3. 配置API密钥

在[application.properties]()或[application.yml]()文件中配置你的OpenAI API密钥：

```properties
spring.ai.openai.api-key=YOUR_OPENAI_API_KEY
```

强烈建议不要将API密钥硬编码在代码中，可以使用环境变量：

```bash
export SPRING_AI_OPENAI_API_KEY=your-api-key-here
```

### 4. 创建控制器

创建一个简单的REST控制器来处理AI聊天请求：

```java
@RestController
@RequestMapping("/ai")
public class AIController {

    private final OpenAiClient aiClient;

    public AIController(OpenAiClient aiClient) {
        this.aiClient = aiClient;
    }

    @GetMapping("/chat")
    public String chat(@RequestParam String message) {
        Prompt prompt = new Prompt(new PromptMessage(MessageType.USER, message));
        PromptResponse response = aiClient.generate(prompt);
        return response.getGeneration().getText();
    }
}
```

### 5. 运行应用

使用以下命令启动应用：

```bash
./mvnw spring-boot:run
```

应用启动后，你可以通过以下URL测试你的AI聊天功能：

```
http://localhost:8080/ai/chat?message=你好，介绍一下Spring AI
```

### 6. 更高级的示例

让我们创建一个更复杂的示例，展示如何使用系统提示和角色设定：

```java
@RestController
@RequestMapping("/ai")
public class AdvancedAIController {

    private final OpenAiClient aiClient;

    public AdvancedAIController(OpenAiClient aiClient) {
        this.aiClient = aiClient;
    }

    @GetMapping("/guided-chat")
    public String guidedChat(@RequestParam String message, @RequestParam String role) {
        // 设置系统提示，定义AI的行为
        String systemPrompt = "你是一个" + role + "，请以专业的角度回答问题。";
        
        // 创建聊天历史
        List<PromptMessage> messages = new ArrayList<>();
        messages.add(new PromptMessage(MessageType.SYSTEM, systemPrompt));
        messages.add(new PromptMessage(MessageType.USER, message));
        
        Prompt prompt = new Prompt(messages);
        PromptResponse response = aiClient.generate(prompt);
        return response.getGeneration().getText();
    }
}
```

现在你可以这样测试：

```
http://localhost:8080/ai/guided-chat?message=请解释一下Java中的多态&role=资深Java架构师
```

## 核心概念解析

### AiClient

AiClient是Spring AI的核心接口，它抽象了与不同AI服务提供商的交互。无论你使用的是OpenAI、Azure OpenAI还是其他服务，都可以通过相同的AiClient接口进行操作。

### Prompt

Prompt代表发送给AI模型的输入。它不仅包含用户的消息，还可以包含系统提示、历史对话等上下文信息。

### Generation

Generation是AI模型的输出结果。它包含了模型生成的文本以及其他元数据。

## 支持的AI服务提供商

Spring AI目前支持多种主流的AI服务提供商：

1. **OpenAI**：支持GPT系列模型
2. **Azure OpenAI**：微软Azure平台上的OpenAI服务
3. **Amazon Bedrock**：亚马逊的AI服务
4. **Google Vertex AI**：谷歌的AI平台
5. **Hugging Face**：开源模型平台

你可以通过添加相应的starter依赖来集成这些服务。

## 实际应用场景

Spring AI可以应用于多种场景：

1. **智能客服**：构建能够理解自然语言的客服系统
2. **内容生成**：自动生成文章、邮件、报告等内容
3. **代码助手**：帮助开发者编写和审查代码
4. **数据分析**：通过自然语言查询和分析数据
5. **教育辅导**：构建智能教学助手

## 最佳实践

1. **API密钥管理**：永远不要将API密钥硬编码在代码中，使用环境变量或配置服务器
2. **错误处理**：AI服务可能会出现各种错误，要做好异常处理
3. **成本控制**：AI服务通常按使用量计费，要注意控制成本
4. **缓存策略**：对于重复的请求，可以考虑使用缓存来减少API调用
5. **测试**：使用Spring AI提供的Mock工具进行单元测试

## 总结

Spring AI为Java开发者提供了一条通往AI应用开发的康庄大道。通过简洁的API和与Spring生态系统的无缝集成，它大大降低了在Java应用中集成AI功能的门槛。

无论你是想要构建一个简单的聊天机器人，还是一个复杂的智能助手，Spring AI都能为你提供强大的支持。随着AI技术的不断发展和Spring AI的持续完善，我们有理由相信，Java开发者将能够更加轻松地拥抱AI时代。

现在就开始你的Spring AI之旅吧！只需要几个简单的步骤，你就能构建出令人惊叹的AI应用。