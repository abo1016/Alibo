---
title: DuerOS 开放平台
tags: [dueros, 文档]
id: '264'
categories:
    - 智能语音
date: 2018-07-25 10:41:46
---

**公司智能家居项目下一步将要接入智能语音平台，现在人工智能语音平台比较成熟，相应的平台厂家也比较多，国内有讯飞、思必驰、DuerOS等等，国外有google的Google Assistant，Amazon的Alexa等。我们先尝试百度的DuerOS平台**

### 概述

DuerOS 开放平台主要面向企业级用户及个人开发者，提供对话式操作系统能力的输出（DuerOS智能设备开放平台）及输入（DuerOS技能开放平台）。

*   #### 技能开放平台
    
    技能开放平台提供了对话式技能所需的NLU(自然语义理解)处理和开发提供了直观的可视化编辑界面。通过编辑界面，可以便捷设计技能的意图、词典等内部逻辑，开发对话式技能。
    
*   #### 技能提供自然的交互方式
    
    看到技能提供的各种服务，你不禁会问手机app也可以提供这些服务，有些手机app也支持语音输入请求，那么技能提供的服务与手机app有什么区别吗？ 技能提供的是对话式交互服务。用户仅通过语音就可以完成与技能的交互，享用技能提供的服务，中间不需要借助其他交互。技能与用户交互过程是模拟用户实际生活中的交互场景，用户与技能交互时，就像与人交互一样自然。如用户使用订票技能购买火车票时，就像用户与售票员交流一样自然。 **我们需要接入的是只能家居平台，DuerOS智能家居的技能不需要关注技能与用户交互实现过程，这部分工作由DuerOS完成。我们只需考虑如何去使用智能家居技能来和我们自身的设备控制云进行交互，达到语音控制设备的目的。** ps:如果你的智能设备在使用过程中想参与到用户交互中，那你需要设计技能与用户的交互模型，此时你需要选择自定义技能实现对设备的控制。
    
*   #### 智能家居技能
    
    智能家居技能让用户通过声音来控制智能设备，查看设备的状态，如控制开灯、关灯。智能家居技能还支持智能场景的设置。智能场景是指一些智能设备的组合使用，把多个智能设备调到预先设定好的状态。如用户使用睡眠场景时，智能家居技能会调暗灯光、关上窗帘。
    
*   #### 智能家居设备工作流程
    
    [![Ptpyq0.md.png](https://s1.ax1x.com/2018/07/25/Ptpyq0.md.png)](https://imgchr.com/i/Ptpyq0)
    
*   #### 智能家居协议
    
*   **简介** 智能家居协议是DuerOS与智能家居技能之间的通讯协议。通过这些协议您可以轻松的通过语音控制家里的智能设备，与设备进行交互。智能家居协议使用HTTPS传输，协议采用JSON消息格式。
*   **认证** 智能家居协议遵循OAuth2.0规范。 从DuerOS发送到技能的每个请求都包含OAuth的access token。
*   **协议** 智能家居协议指令（directives）由Header和Payload两部分组成。
*   **Header信息** Header包含消息标识符、指令名称、命令空间和payload版本信息。 消息格式：

```json
{
    "header": {
        "namespace": "DuerOS.ConnectedHome.Discovery",
        "name": "DiscoverAppliancesRequest",
        "messageId": "6d6d6e14-8aee-473e-8c24-0d31ff9c17a2",
        "payloadVersion": "1"
    }
}
```

> 总结：百度的DuerOS可以视为真正意义上的操作系统，类似于Android系统，技能就相当于Android中的App。载体就由手机和各种智能触屏设备变成语音输入设备，并且最重要的一点是DuerOS非常开放，与Android相比开放程度有之过而无不及，相信未来DuerOS在智能语音领域也会成为如Android一样伟大的平台。