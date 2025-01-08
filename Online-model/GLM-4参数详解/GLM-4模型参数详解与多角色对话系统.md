```python
import os
from zhipuai import ZhipuAI
from IPython.display import display, Code, Markdown

api_key = os.getenv("ZHIPU_API_KEY")
```

  在上一小节中，我们已经顺利完成智谱AI账号注册，并获取GLM模型API-KEY，同时完成了本地调用测试。本节我们将开始着手使用GLM-4模型，并详细介绍GLM-4调用函数参数及模型的多角色对话系统。

```python
client = ZhipuAI(api_key=api_key) 
response = client.chat.completions.create(
    model="glm-4",  # 填写需要调用的模型名称
    messages=[
        {"role": "user", "content": "你好"}
    ],
)
print(response.choices[0].message)
```

```plaintext
content='你好👋！我是人工智能助手智谱清言，可以叫我小智🤖，很高兴见到你，欢迎问我任何问题。' role='assistant' tool_calls=None
```

### 一、GLM-4调用方法

#### 1.SDK调用与HTTP调用

  和大多数其他在线大模型厂商一样，智谱AI为GLM模型同时提供了SDK调用和HTTP调用两种方法。所谓SDK调用，指的是基于SDK（软件开发工具包）的API调用是指使用由API提供者提供的专门软件库（即SDK）来进行API调用的过程。SDK是一组工具、库、文档和代码示例，旨在帮助开发者更容易、更高效地使用特定的API。在基于SDK的API调用中，开发者不需要从头开始编写用于与API通信的代码，而是可以利用SDK中的预写函数和方法。

  而HTTP调用则是指通过HTTP（超文本传输协议）在客户端和服务器之间发送请求和接收响应的过程。这是Web上交换数据和信息的基础机制。在HTTP调用中，客户端（通常是浏览器或应用程序）向服务器发送一个HTTP请求，这个请求可以是获取数据（GET请求）、提交数据（POST请求）、更新数据（PUT或PATCH请求）或删除数据（DELETE请求）。服务器接收到这个请求后，处理请求并返回一个HTTP响应，其中包含状态代码（如200表示成功，404表示未找到等）和请求的数据或结果。

  例如此前我们通过安装zhipu库来完成模型调用，本质上就是SDK调用模型，这也将是课程里面主要使用的调用方法。

![](images/edf4a881-d9a9-427f-b956-2d5932d8b646.png)

#### 2.同步调用、异步调用与SSE调用

  而在SDK调用过程中，又可以细分为同步调用、异步调用与SSE调用三种不同的调用方法。所谓的同步调用，指的是我们发起请求，等待服务器响应后方可进行下一步动作，并且服务器响应也是一次性获得全部结果。我们此前进行的模型调用尝试，就是同步调用过程；而所谓异步调用，则指的是当发起请求后，服务器会先直接响应当前任务ID，此时无论任务是否已经完成，用户都可以直接进行接下来的动作，而当服务器完成任务时，用户可以通过任务ID查看到服务器响应结果；而所谓的SSE调用，则是指流式传输，此时服务器不再是一次性返回全部内容，而是逐个字符（或者词语）进行返回。这三种调用方法实现过程如下：

* 同步调用过程

  我们可以通过调用client.chat.completions.create函数来实现同步调用，此时用户发起请求后，需要等待服务器一次性全部响应之后再进行之后的操作：

```python
client = ZhipuAI(api_key=api_key) 
response = client.chat.completions.create(
    model="glm-4",  # 填写需要调用的模型名称
    messages=[
        {"role": "user", "content": "你好"}
    ],
)
print(response.choices[0].message)
```

```plaintext
content='你好👋！我是人工智能助手智谱清言，可以叫我小智🤖，很高兴见到你，欢迎问我任何问题。' role='assistant' tool_calls=None
```

* 异步调用

  调用后会立即返回一个任务 ID ，然后用任务ID查询调用结果（根据模型和参数的不同，通常需要等待10-30秒才能得到最终结果）。需要注意的是异步调用所使用的函数并不是client.chat.completions.create，而是client.chat.asyncCompletions.create：

```python
client = ZhipuAI(api_key=api_key) 
response_async = client.chat.asyncCompletions.create(
    model="glm-4",  
    messages=[
        {"role": "user", "content": "你好"}
    ],
)
print(response_async)
```

```plaintext
id='754217054184439348311645931336567876' request_id='8311645931336567875' model='glm-4' task_status='PROCESSING'
```

```python
response_async
```

```plaintext
AsyncTaskStatus(id='754217054184439348311645931336567876', request_id='8311645931336567875', model='glm-4', task_status='PROCESSING')
```

* SSE调用过程

  而SSE调用的流式传输则较好理解，当服务器接收到请求之后，会实时发送模型预测结果。SSE调用和同步调用所使用的函数相同，都是client.chat.completions.create，唯一不同之处在于SSE调用需要有额外参数stream，并需要将其设置为True：

```python
client = ZhipuAI(api_key=api_key) 
response_stream = client.chat.completions.create(
    model="glm-4",  
    messages=[
        {"role": "user", "content": "你好"}
    ],
    stream=True,
)
```

```python
for chunk in response_stream:
    print(chunk.choices[0].delta)
```

```plaintext
content='你好' role='assistant' tool_calls=None
content='👋！' role='assistant' tool_calls=None
content='我是' role='assistant' tool_calls=None
content='人工智能' role='assistant' tool_calls=None
content='助手' role='assistant' tool_calls=None
content='智' role='assistant' tool_calls=None
content='谱' role='assistant' tool_calls=None
content='清' role='assistant' tool_calls=None
content='言' role='assistant' tool_calls=None
content='，' role='assistant' tool_calls=None
content='可以' role='assistant' tool_calls=None
content='叫我' role='assistant' tool_calls=None
content='小' role='assistant' tool_calls=None
content='智' role='assistant' tool_calls=None
content='🤖，' role='assistant' tool_calls=None
content='很高兴' role='assistant' tool_calls=None
content='见到' role='assistant' tool_calls=None
content='你' role='assistant' tool_calls=None
content='，' role='assistant' tool_calls=None
content='欢迎' role='assistant' tool_calls=None
content='问我' role='assistant' tool_calls=None
content='任何' role='assistant' tool_calls=None
content='问题' role='assistant' tool_calls=None
content='。' role='assistant' tool_calls=None
content='' role='assistant' tool_calls=None
```

课程前期将主要以同步调用过程为主，在某些场景下将用到SSE调用。

### 二、chat.completions.create函数参数详解

#### 1.chat.completions.create函数参数解释

  在通过SDK调用模型的时候，chat.completions.create函数中的参数设置会直接决定最终模型运行模式、以及模型运行结果。因此在基础模型学习阶段，我们需要围绕chat.completions.create函数进行完整的参数解释。

* chat.completions.create函数必选参数

  整体来看，GLM模型参数和GPT模型参数高度类似，并且通过观察不难发现，chat.completions.create函数有两个必选参数，其一是model，代表的含义是当前调用的模型。根据上一小节的介绍，目前可选的模型有两个，其一是GLM-3-Turbo、其二则是GLM-4；第二个参数则是messages，代表传输到模型内部的消息队列。messages参数是一个基本构成元素为字典的列表，其内每个字典都代表一条独立的消息，每个字典都包含两个键值（Key-value）对，其中第一个Key都是字符串role（角色）表示某条消息的作者，第二个key为content（内容）表示消息具体内容。更多关于message的参数设置方法稍后介绍，总的来看，这里的messages就可以简单理解为输入给模型的信息队列，而模型接收到message之后也会输出对应的回答信息，当然也是以message形式呈现：

```python
response.choices[0].message
```

```plaintext
CompletionMessage(content='你好👋！我是人工智能助手智谱清言，可以叫我小智🤖，很高兴见到你，欢迎问我任何问题。', role='assistant', tool_calls=None)
```

* chat.completions.create函数全参数解释

  接下来围绕chat.completions.create函数的全部参数进行解释：

| 参数名称     | 类型     | 是否必填 | 参数解释      |
| -------- | ------ | ---- | --------- |
| model    | String | 是    | 所要调用的模型编码 |
| messages | List   |      |           |



📍**更多大模型技术内容学习**

**扫码添加助理英英，回复“大模型”，了解更多大模型技术详情哦👇**

![](images/f339b04b7b20233dd1509c7fb36d5c0.png)

此外，**扫码回复“入群”**，即可加入**大模型技术社群：海量硬核独家技术`干货内容`+无门槛`技术交流`！**
