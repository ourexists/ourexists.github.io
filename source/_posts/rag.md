---
title: SpringAi+Ollama+Qdrant的本地RAG知识库搭建
tags:
  - ai
  - RAG
categories:
  - [ 人工智能, RAG ]
top_img: /img/top.png
cover: /img/ai.jpg
---
### 一、技术选型

JAVA21、spring AI 1.0.0-M6、Qdrant v1.14.1、Ollama 0.9.0

### 二、环境准备

#### Qdrant安装

这里推荐使用docker进行安装，以下是docker-compose脚本。(新建text文件，复制进去重命名文件为`docker-compose.yml`,执行`docker compose -f ./{file_path}/docker-compose.yml up`)

```
services:  
 qdrant:  
 image: qdrant/qdrant:latest  
 container_name: qdrant  
 ports:  
 - "6333:6333"  
 - "6334:6334"  
 volumes:  
 - ${MOUNT_PATH}/qdrant/storage:/qdrant/storage # 数据持久化  
 environment:  
 QDRANT__SERVICE__HOST: 0.0.0.0
```

安装成功打开控制台`http://localhost:6333/dashboard`,可以看见如下界面：

![qdrant-panel.jpg](qdrant-panel.jpg)

#### Ollama安装

由于Ollama官方未完全支持docker部署，直接从官网下载安装包即可：[Download Ollama on Windows](https://ollama.com/download)。

安装成功后执行`Ollama -v`可以查看当前的安装版本。

-  修改Ollama模型存储地址
  
  >  **windows**
  > 
  > 1. 打开系统环境变量，新增环境变量`OLLAMA_MODELS=<文件路径>\Ollama\models`
  > 
  > 2. 如果是已经安装的模型路径，可以从`C:\Users\<你的用户名>\.ollama\models`将模型拷贝过来直接链接使用。

- 下载需要使用的模型
  
  > 1. 下载本地文本嵌入模型`Ollama pull nomic-embed-text`.用于将文本转换成向量。
  > 
  > 2. 下载本地大语言模型`Ollama pull mistral:instruct`.这里mistral也可以作为向量模型使用，但不推荐，测试可以使用。
  >    
  >    > 这里的模型可以自由搭配，例如：deepseek R1、LIama等等

#### RAG服务搭建

- 新建一个springboot项目，推荐使用`spring-boot-starter-parent:3.4.6`.这里不做过多讲解
  
  ```
    <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>3.4.6</version>
      </parent>  
  ```

- 引入POM相关依赖：
  
  ```
  <dependencies>
          <!-- ollama连接启动器 -->
          <dependency>
              <groupId>org.springframework.ai</groupId>
              <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
          </dependency>
  
          <!-- qdrant连接启动器 -->
          <dependency>
              <groupId>org.springframework.ai</groupId>
              <artifactId>spring-ai-qdrant-store</artifactId>
          </dependency>
  </dependencies>
  
  <dependencyManagement>
          <dependencies>
              <!-- spring ai生态的管理包 -->
              <dependency>
                  <groupId>org.springframework.ai</groupId>
                  <artifactId>spring-ai-bom</artifactId>
                  <version>1.0.0-M6</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
          </dependencies>
  </dependencyManagement>
  ```

- 添加配置
  
  ```
  spring:
    ai:
      vectorstore:
        qdrant:
          host: localhost
          collection-name: knowledge //注意这里需要先进Qdrant中建立对应Collection
          port: 6334    //注意端口不是6333，最后会解释下。
      ollama:
        base-url: http://127.0.0.1:11434
        embedding:
          model: nomic-embed-text
        chat:
          model: mistral:instruct
  ```

- 代码配置
  
  ```
  @Configuration
  public class MyAIConfiguration{
  
      /**
       * 框架默认不会注入,需要手动
       */
      @Bean
      public ChatClient chatClient(ChatModel chatModel) {
          return ChatClient
                  .builder(chatModel)
                  .build();
      }
  }
  ```

- 实现一个简易的知识库问答
  
  ```
  @Component
  public class KnowledgeLoader {
  
      private final QdrantVectorStore vectorStore;
      private final ChatClient chatClient;
  
      public KnowledgeLoader(QdrantVectorStore vectorStore,
                             ChatClient chatClient) {
          this.vectorStore = vectorStore;
          this.chatClient = chatClient;
      }
  
      public void save(String content) {
          Document doc = Document.builder()
                  .id(UUID.randomUUID().toString())
                  .text(content)
                  .build();
          vectorStore.add(List.of(doc));
      }
  
      public List<Document> retrieve(String query, int topK) {
          return vectorStore.similaritySearch(SearchRequest.builder().query(query).topK(topK).build());
      }
  
      public String answer(String question) {
          List<Document> docs = retrieve(question, 3);
          String context = docs.stream()
                  .map(Document::getText)
                  .reduce("", (acc, cur) -> acc + "\n" + cur);
  
          return chatClient.prompt()
                  .system("你是一个知识库问答助手，请根据提供的知识回答用户问题。")
                  .user("知识库内容如下：\n" + context + "\n\n问题：" + question)
                  .call()
                  .content();
      }
  }
  ```
  
  ```
  @RestController
  @RequestMapping("/ai")
  public class AskController {
  
      @Autowired
      private KnowledgeLoader knowledgeLoader;
  
      @GetMapping("ask")
      public String ask(@RequestParam String question) {
          return knowledgeLoader.answer(question);
      }
  
      @GetMapping("save")
      public void save(@RequestParam String content) {
          knowledgeLoader.save(content);
      }
  }
  ```

#### 服务测验

1. 新增知识库内容
   > GET http://localhost:10021/open/ai/save?content="Qdrant是一个开源的向量数据库，支持高效的相似性搜索。它采用余弦相似度和欧氏距离等多种距离函数。Qdrant 适合用于语义搜索、推荐系统和机器学习场景。Spring AI 是一个基于 Spring Boot 的 AI 框架，支持调用 OpenAI、Qdrant 等多种 AI 服务。它内置了 EmbeddingModel，可方便地将文本转成向量。"

2. 问答
   > GET http://localhost:10021/open/ai/ask?question=Qdrant是什么?  

## 三、踩坑注意

- 连接`qdrant`的配置端口一定要是`6334`而不能是`6333`.
  
  - `6333`端口使用的是`HTTP / REST API`通信模型,`qdrant`使用的协议为`Http/1.1`,但是`spring restClient`默认协议是`Http/2`且无法强制更改.
  
  - `6334`端口使用的`gRPC API`通信基于 HTTP/2 协议实现的

- 当前版本`EmbeddingClient`已被舍弃,不要被其它文档误导.

- 当前版本`EmbeddingModel`在框架中会自动注入到`VectorStore`中,无需手动注入.
