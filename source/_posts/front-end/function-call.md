---
title: openai Function call
categories:
  - front-end
img: ../../coverImages/openai.png
tags:
  - axios
  - openai
date: 2023-10-18 07:27:59
---

Function Call 是 openai 6 月份推出的一个新功能，可以把 GPT 作为输入输出，中间数据的处理生成由外部处理。

# 理解

Function Call 比较好理解
1. 定义函数，供 GPT 选择调用
2. GPT 作为输入端，提取用户的输入作为参数，调用自定义的函数
3. GPT 语义化输出函数的内容

![](/images/function.svg)

# 举例

```js
// openai function call example  
import OpenAI from "openai";
const openai = new OpenAI({
    apiKey: "", // 你的 key});  

// 被调用的函数，这里是静态数据  
    function getCurrentWeather(location, unit = "fahrenheit") {
    const weatherInfo = {
        "location": location,
        "temperature": "22",
        "unit": unit,
        "forecast": ["sunny", "windy"],
    };
    return JSON.stringify(weatherInfo);
}

async function runConversation() {
    // Step 1: 发生消息给 GPT    
    const messages = [{"role": "user", "content": "今天的天气怎么样?"}];
    const functions = [
        {
            // 定义函数名称  
            "name": "get_current_weather",
            // 定义函数的作用  
            "description": "Get the current weather in a given location，没有地点，需要询问用户",
            // 描述函数接受的参数的 JSON 架构对象  
            // https://json-schema.org/understanding-json-schema  
            "parameters": {
                "type": "object",
                // 函数参数  
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "The city and state, e.g. San Francisco, CA",
                    },
                    "unit": {"type": "string", "enum": ["celsius"]},
                },
                // 函数参数是否必传  
                "required": ["location"],
            },
        }
    ];

    const response = await openai.chat.completions.create({
        model: "gpt-3.5-turbo",
        messages: messages,
        // 注意，这里新增了 function 参数  
        functions: functions,
        function_call: "auto",  // auto is default, but we'll be explicit  
    });
    const responseMessage = response.choices[0].message;
    // Step 2: 检查返回的消息是否包含函数调用  
    if (responseMessage.function_call) {
        // Step 3: 调用函数  
        // Note: the JSON response may not always be valid; be sure to handle errors  
        const availableFunctions = {
            get_current_weather: getCurrentWeather,
        };  // only one function in this example, but you can have multiple  
        const functionName = responseMessage.function_call.name;
        const functionToCall = availableFunctions[functionName];
        const functionArgs = JSON.parse(responseMessage.function_call.arguments);
        const functionResponse = functionToCall(
            functionArgs.location,
            functionArgs.unit,
        );

        // Step 4: 把函数返回结果发送给 GPT        messages.push(responseMessage);  // extend conversation with assistant's reply  
        messages.push({
            "role": "function",
            "name": functionName,
            "content": functionResponse,
        });
        // 获取 GPT 的转换之后的回复。  
        const secondResponse = await openai.chat.completions.create({
            model: "gpt-3.5-turbo",
            messages: messages,
        });  // get a new response from GPT where it can see the function response  
        return secondResponse;
    }
}

runConversation().then(console.log).catch(console.error);
```

functions 函数有三个主参数：name、description 和 parameters。模型使用 description 参数来确定何时以及如何调用函数，需要提供说明。
parameters 是描述函数接受的参数的 JSON 架构对象。可以在 JSON 架构参考 中详细了解 JSON 架构对象。

# 对话举例：

```
user - 今天天气怎么样
gpt - 请给出你的城市
user - 无锡
gpt - (调用函数)今天天气晴
```

```
user - 无锡今天天气怎么样
gpt - (调用函数)今天天气晴
```


# 使用场景

可以极大的扩展GPT的使用场景，可以相对安全的输出内容。
- 客服场景：根据关键词触发相关的回答
- 搜索：根据关键词进行搜索，比如搜索语雀等知识库，输出相关内容
参考

如何将函数调用与 Azure OpenAI 服务配合使用 - Azure OpenAI Service | Microsoft Learn
Function calling and other API updates

# 参考

- [如何将函数调用与 Azure OpenAI 服务配合使用 - Azure OpenAI Service | Microsoft Learn](https://learn.microsoft.com/zh-cn/azure/ai-services/openai/how-to/function-calling)
- [Function calling and other API updates](https://openai.com/blog/function-calling-and-other-api-updates)
