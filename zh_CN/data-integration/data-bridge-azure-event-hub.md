# 将 MQTT 数据传输到 Azure Event Hubs

{% emqxce %}
::: tip

EMQX 企业版功能。EMQX 企业版可以为您带来更全面的关键业务场景覆盖、更丰富的数据集成支持，更高的生产级可靠性保证以及 24/7 的全球技术支持，欢迎[免费试用](https://www.emqx.com/zh/try?product=enterprise)。

:::
{% endemqxce %}

[Azure Event Hub](https://azure.microsoft.com/en-us/products/event-hubs) 是一个用于数据摄取的实时托管事件流平台。EMQX 与 Azure Event Hub 的集成为用户在高吞吐量情况下提供了可靠的数据传输和处理能力。Azure Event Hubs 可作为 EMQX 与 Azure 丰富的云服务应用之间的数据通道，将物联网数据集成到 Azure Blob Storage、Azure Stream Analytics 以及部署在 Azure 虚拟机上的各类应用和服务当中。目前，EMQX 支持使用 SASL/PLAIN 身份验证、通过与 Kafka 协议兼容的 Apache Kafka 终端点进行 Azure Event Hub 集成。

本页详细介绍了 EMQX 与 Azure Event Hubs 的数据集成并提供了实用的规则和数据桥接创建指导。

## 工作原理

Azure Event Hubs 数据桥是 EMQX 的一个开箱即用功能，旨在帮助用户无缝集成 MQTT 数据流与 Azure Event Hubs，利用其丰富的服务和能力进行物联网应用开发。

![event_hubs_architecture](./assets/event_hubs_architecture.svg)

EMQX 通过规则引擎和数据桥接将 MQTT 数据转发到 Azure Event Hubs。完整的过程如下：

1. **物联网设备发布消息**：设备通过特定主题发布遥测和状态数据，触发规则引擎。
2. **规则引擎处理消息**：使用内置规则引擎，基于主题匹配处理来自特定来源的 MQTT 消息。规则引擎匹配相应的规则并处理消息，例如转换数据格式、过滤特定信息或用上下文信息丰富消息。
3. **桥接到 Azure Event Hubs**：规则触发将消息转发到 Azure Event Hubs 的动作，允许轻松配置数据属性、排序键，并将 MQTT 主题映射到 Azure Event Hubs 消息头。这为数据集成提供了更丰富的上下文信息和顺序保证，使得物联网数据处理更加灵活。

在 MQTT 消息数据写入 Azure Event Hubs 之后，您可以进行灵活的应用开发，例如：

- 实时数据处理和分析：利用 Azure Event Hubs 强大的数据处理和分析工具及其自身的流处理能力，对消息数据进行实时处理和分析，获取有价值的洞察和决策支持。
- 事件驱动功能：触发 Azure 事件处理，实现动态灵活的功能触发和处理。
- 数据存储和共享：将消息数据传输到 Azure Event Hubs 存储服务，安全存储和管理大量数据。这使您能够与其他 Azure 服务共享和分析这些数据，以满足各种业务需求。

## 特性与优势

EMQX 与 Azure Event Hubs 的数据集成可以为您的业务带来以下功能和优势：

- **高性能海量消息吞吐**：EMQX 支持海量 MQTT 客户端连接，每秒数百万条消息能够持续引入 Azure Event Hubs，可以获得极低的消息传输与存储延迟时间，并在 Azure Event Hubs 上配置保留时间实现消息量的控制。
- **灵活的数据映射**：通过配置的 Azure Event Hubs，可以实现 MQTT 主题与 Azure Event Hubs 事件中心的灵活映射，并且支持 MQTT 用户属性与 Azure Event Hubs 消息头的映射，这为数据集成提供了更丰富的上下文信息和顺序保证。
- **弹性伸缩支持**：EMQX 与 Azure Event Hubs 均可以支持弹性伸缩，能够随着应用规格进行扩展，轻松将物联网数据规模从数 MB 轻松扩展到数 TB。
- **丰富的生态系统**：得益于采用标准 MQTT 协议以及各类主流物联网传输协议的支持，EMQX 能够实现各类物联网设备的接入。结合 Azure Event Hubs 在 Azure Functions、各类编程语言 SDK 以及 Kafka 生态系统中的支持，能够轻松打通设备到云端的数据通道，实现无缝物联网数据接入与处理。

这些功能增强了集成能力和灵活性，可以帮助用户快速实现海量物联网设备数据与 Azure 的连接。让用户更便捷的获得云计算带来的数据分析和智能化能力，构建功能强大的数据驱动型应用。

## 桥接准备

本节介绍了在 EMQX 中创建 Azure Event Nubs 数据桥接之前需要做的准备工作，包括如何设置 Azure Event Hubs。

### 前置准备

- 了解[规则](./rules.md)。
- 了解[数据桥接](./data-bridges.md)。

### 设置 Azure Event Hubs

为了使用 Azure Event Hub 数据集成，必须在 Azure 账户中设置命名空间和事件中心。以下 Azure 官方文档详细介绍了如何进行设置。

- [什么是适用于 Apache Kafka 的 Azure 事件中心](https://learn.microsoft.com/zh-cn/azure/event-hubs/azure-event-hubs-kafka-overview)
- [快速入门：使用 Azure 门户创建事件中心](https://learn.microsoft.com/zh-cn/azure/event-hubs/event-hubs-create)
- [快速入门：使用 Azure 事件中心和 Apache Kafka 流式传输数据](https://learn.microsoft.com/zh-cn/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs?tabs=connection-string)
  - 遵循“连接字符串”说明，这是 EMQX 用于连接的方式。
- [获取事件中心连接字符串](https://learn.microsoft.com/zh-cn/azure/event-hubs/event-hubs-get-connection-string)

## 创建 Azure Event Hubs 数据桥接

本节演示如何通过 Dashboard 创建 Azure Event Hubs 生产者数据桥接、将数据通过数据桥接流入或流出 Azure Event Hubs。

1. 进入 EMQX Dashboard，点击**集成** -> **数据桥接**。

2. 点击页面右上角的**创建**。

3. 在**创建数据桥接**页面，点击选择 **Azure Event Hubs**，然后点击**下一步**。

4. 为数据桥接输入一个名称。名称应为大小写字母和数字的组合。

5. 配置连接信息。

   - **引导主机**：输入命名空间的主机名。默认端口为 `9093`。其他字段按实际情况设置。
   - **连接字符串**：输入命名空间的连接字符串。可以在命名空间共享访问策略的“连接字符串 - 主键”中找到。有关详细信息，请参阅 [获取事件中心连接字符串](https://learn.microsoft.com/zh-cn/azure/event-hubs/event-hubs-get-connection-string)。
   - **启用 TLS**：连接到 Azure Event Hub 时默认启用 TLS。有关 TLS 连接选项的详细信息，请参阅 [外部资源访问的 TLS](../network/overview.md#启用-tls-加密访问外部资源)。

6. 配置数据桥接信息。

   - **事件中心名称**：输入要使用的事件中心的名称。注意：此处不支持变量。
   - **Azure Event Hub 头部**：输入一个占位符，作为将在发布到 Azure Event Hub 时添加到消息中的消息标头。
   - **Azure Event Hub 头部值编码模式**：选择消息标头的值编码模式；可选值为 `none` 或 `json`。
   - **额外的 Azure Event Hub 头部信息**：您可以点击**添加**为 Azure Event Hub 消息标头提供更多的键值对。
   - **消息键**：事件中心消息键。在此处插入一个字符串，可以是纯字符串或包含占位符（${var}）的字符串。
   - **消息值**：事件中心消息值。在此处插入一个字符串，可以是纯字符串或包含占位符（${var}）的字符串。
   - **消息时间戳**：指定要使用的时间戳类型。

7. 高级设置（可选）：根据业务需求设置 **最大批次字节数**、**所需确认** 和 **分区策略**等。

8. 在点击**创建**之前，您可以点击**测试连接**测试桥接是否能够连接到 Azure Event Hub 服务器。

9. 点击**创建**，系统将提示您创建关联规则。

   对于 Azure Event Hub 生产者数据桥接，点击**创建规则**创建关联规则。有关详细操作步骤，请参阅 [为 Azure Event Hub 生产者数据桥接创建规则](#创建-azure-event-hubs-生产者数据转发规则)。

   ::: tip

   创建规则允许对与规则匹配的 Azure Event Hub 消息进行进一步的转换和过滤，然后将其转发到其他规则操作，如不同的桥接。有关创建规则的更多信息，请参阅[规则](./rules.md)。

   :::

现在，Azure Event Hubs 数据桥接应该在数据桥接列表（**集成** -> **数据桥接**）中显示，**资源状态**为 **已连接**。

## 创建 Azure Event Hubs 生产者数据转发规则

至此您已经完成数据桥接创建流程，接下来将继续创建一条规则来指定需要写入的数据：

1. 转到 Dashboard **数据集成** -> **规则页面**。

2. 点击页面右上角的创建。

3. 输入规则 ID，例如  `my_rule`。

4. 在 SQL 编辑器中输入规则，例如我们希望将 `t/#` 主题的 MQTT 消息存储至 Azure Event Hub，可通过如下规则实现：

   注意：如果要自定义 SQL 语句，请确保 `SELECT` 字段包含数据桥接中所需的所有字段。


```sql
SELECT
  *
FROM
  "t/#"
```

5. 点击**添加动作**按钮，在下拉框中选择**使用数据桥接转发**选项，选择先前创建好的 Azure Event Hubs 数据桥接。

6. 点击**添加**按钮确认添加动作。

7. 点击最下方**创建**按钮完成规则创建。

至此您已经完成整个创建过程，可以前往 **数据集成** -> **Flows** 页面查看拓扑图，此时应当看到 `t/#` 主题的消息经过名为 `my_rule` 的规则处理，处理结果交由 Azure Event Hub 进行存储。

## 测试数据桥接和规则

您可以使用 [MQTTX](https://mqttx.app/zh) 来模拟客户端向 EMQX 发送 MQTT 消息来测试数据桥接和规则的运行。

1. 使用 MQTTX 向 `t/1` 主题发布消息：

```bash
   mqttx pub -i emqx_c -t t/1 -m '{ "msg": "Hello Azure Event Hub" }'
```

2. 在**数据桥接**页面点击桥接名称查看数据桥接运行统计，命中、发送成功次数应当 +1。

3. 在 Azure 门户仪表板中检查是否将消息写入配置的事件中心。使用任何兼容 Kafka 的消费者，检查消息是否被写入配置的事件中心。有关使用 Kafka CLI 的更多信息，请参阅 [Use the Kafka CLI to Send and Receive Messages to/from Azure Event Hubs for Apache Kafka Ecosystem](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafka-cli)。