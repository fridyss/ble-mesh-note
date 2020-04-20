# 3 MESH网络

本节的结构与2.1节中描述的分层体系结构相同。另外，还有关于mesh安全性和mesh网络管理的章节。

## 3.1 约定

以下约定应用于此规范文档。

### 3.1.1 字节顺序和字段组合

1. 对于网络层，下传输层，上传输层，mesh bean ，provisioning，所有多字节的数值都要以大端字节发送, 如第3.1.1.1节所述。
2. 对于访问层，基础模型，所有的多字节数值都应该是小端字节，如3.1.1.2节所述。

网络数据结构是由多个字段组成的，这些字段从上到下依次列在表中，从左到右依次出现在相应的图中。表3.1所示，一个由多个字段组成的示例数据结构的实例。

| 字段 | 大小(bits) | 字段的内容描述 |
| --- | --- | --- |
| 字段0 | 4  | 这个字段的起始位置是0字节 |
| 字段1 | 12 | 该字段的起始位置为0字节，结束位置为1字节 |
| 字段2 | 16 | 该字段的起始位置为2字节，结束位置为3字节  |
*表3.1 字段组合例程*

为了将表中定义的数据结构，转换为使用大端模式表示，高字节在低内存地址。如图3.1所示。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/field%20ordering%20example%20big%20endian.png)
*图3.1 大端字段组合*

例如 ，字段0的宽度为4位，值为0x6，字段1的宽度为12位，值为0x987，字段2的宽度为16位，值为0x1234，二进制数的值是0x69871234，传输顺序应该按0x69、0x87、0x12、0x34。

为了将表中定义的数据结构，转换为使用小端模式表示，低字节在低内存地址。如图3.2所示。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/field%20ordering%20example%20little%20endian.png)
*图3.2 小端字段组合*

例如，字段0的宽度为4位，值为0x6，字段1的宽度为12位，值为0x987，字段2宽16位，值为0x1234。二进制数的值0x12349876，传输顺序应该按0x76, 0x98, 0x 34, 0x12。

#### 3.1.1.1 大端

当以“大端”(也称为“网络字节顺序”)发送的多字节值被定义时，应用本节中的约定。例如，值0x123456应该被传输为0x12、0x34和0x56(MSB优先)。
#### 3.1.2.2 小端
当在“小端”中定义多字节值时，应用本节中的约定。例如，值0x123456应该被传输为0x56、0x34和0x12(LSB优先)。
## 3.2 特性
这个规范文档定义了4种操作特性：

* 中继特性(参考3.6.4.1)
* 代理特性(参考3.6.4.2)
* 友元特性(参考3.6.4.3)
* 低功耗特性(参考3.6.4.4)
## 3.3 承载
规范文档定义了两个mesh承载，用来传输mesh消息的。

* 广播承载（参考3.3.1）
* GATT承载（参考3.3.2）
节点应该支持其中一个，或者两者都支持。
### 3.3.1 广播承载
当使用mesh广播承载时，一个mesh数据包发送，应该包含在通用广播的PDU中，其中用 "Mesh Message AD Tpyte" 类型标识，"NetWork PDU"就是其对应的负载，如表3.2所示。
| 长度 | 广播类型 | 内容 |
| --- | --- | --- |
| 0xXX | << Mesh Message >> | NetWork PDU  |
*表3.2 Mesh Message AD Tpyte*
 任何时候，当广播事件是“Mesh Message AD Type”类型时，这个广播事件应该是不可以连接且不可以扫描的不定向。当节点在可连接或可扫描的广播中接收到这类mesh消息，则这类消息应该是被忽略的。
 
注意:不可连接广播被使用，因为在广播packets中不需要包含“Flags AD Type”,所以可以多分配两个字节到“NetWork PDU”. 为了降低所有广播通道中数据包在空中的踫撞率，在连续的广播包之间会插入一个随机时间间隔值。

一个设备如果只支持广播承载，则应该被动地执行扫描，为了避免掉失将要输入的mesh消息和"Provisioning PDU" , 需要尽可能使扫描占空比接近100%。

设备应该同时支持观者察角色和广播者角色。

### 3.3.2 GATT承载
GATT承载 ，主要的作用是使不支持广播承载功能的设备参与到mesh网络中，在设备建立连接的基楚上，GATT承载在两个设备之间通过使用“proxy 协议(参考6章节)”收发 “proxy PDUs”.

GATT承载使用通过属性协议上的特证值对mesh消息的通知进行读写操作。

GATT承载定义了两个角色：“GATT Bear Client” 和 “GATT Bear Server”.

"GATT Bear Server" 应该实例一次，并且只有一个“Mesh Proxy Serive”,具体定义在7.2章节。“GATT Bear Client” 应该支持“Mesh Proxy Serive”。

GATT承载客户端应该进行主要服务发现，主要通过使用“GATT Discover All Primary Services”的子程序 或者 “GATT Discover Primary Services by Service UUID”的子程序，发现mesh代理服务。

根据GATT的要求，GATT承载客户端比须兼容其他可选的特证值，这些特证值在profile的服务中，与其他特证值一起使用。

GATT承载客户端应该使用“GATT Discover All Characteristics of a Service”的子程序 或者 “GATT Discover Characteristics by UUID”的子程序，来发现服务特证值。

GATT承载客户端应用使用“GATT Discover All Characteristic Descriptors”的子程序去发现特证值的描述符。

GATT承载客户端应该发现“Mesh Proxy Data In”特证值, "Mesh Proxy Data Out"特证值，和 “Client Characteristic Configuration ”描述符，一但“Client Characteristic Configuration”描述符 被发现，它应启用使用此特性的通知。

为了发送一个“proxy PDU”(参考6.4章节), GATT客户端应该使用“Write Without Response”子程序去写一个"Proxy PDU"到GATT服务端，通过写“Mesh Proxy Data In”特证值。

为了接收一个“Proxy PDU”,GATT服务端应该能够接收多个"Mesh Proxy Data Out"特证值的通知，每个通知都包含一个单独的“Proxy PDU”.

## 3.4 网络层
网络层定义 "NetWork PDU"格式，该格式允许承载层传输"Lower Transport PDUs"，它对输入接口上接收到的传入消息进行解密和认证，并将其转发到上层和/或输出接口，对发送到输出网络接口的传出消息进行加密和认证，并将其转发到输出网络接口。
### 3.4.1 字节顺序
该层中的所有多字节数值应以“大端”形式发送，如第3.1.1.1节所述。
#### 3.4.2 地址 
网络层定义了四种基本类型的地址：未分配地址, 单播地址, 虚拟地址, 组地址。地址的长度为16位，其编码如表3.3所示。
| 值 | 地址类型 |
| --- | --- |
| 0b00000000-00000000 | 未分配地址 |
| 0b0xxxxxxx-xxxxxxxx（除未分配地址） | 单播地址 |
| 0b10xxxxxx-xxxxxxxx | 虚拟地址  |
| 0b11xxxxxx-xxxxxxxx | 组地址 |
*表3.3 16位地址分配 *

#### 3.4.2.1 未分配地址 
未分配地址是指尚未配置节点元素或尚未分配地址的地址，未分配地址的值应为0x0000，如图3.3所示。例如，可以通过将模型的发布地址设置为未分配的地址，禁用模型的消息发布。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_3%20unassigned%20address%20format.png)
*图3.3 未分配地址格式*
未分配的地址不得用于消息的源或目标地址字段。

#### 3.4.2.2 单播地址 
单播地址是分配给每个元素的唯一地址，单播地址的位15设置为0。单播地址不应是0x0000，因此可以为从0x0001到0x7FFF（包括0x7FFF）的任何值，如下面的图3.4所示。

如第5.4.2.5节所述，在配置期间，在网络上节点的生存期内，由provisioner将单播地址分配给节点的每个元素。provisioner可能未分配该地址，以允许使用过程重用该地址。过程定义 在3.10.7章节。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_4%20unicat%20address%20format.png)
*图3.4 单播地址格式*

单播地址应用于消息的源地址字段，也可用于消息的目标地址字段。发送到单播地址的消息最多由一个元素处理。
#### 3.4.2.3 虚拟地址 
虚拟地址表示一组目标地址，每个虚拟地址在逻辑上表示一个标签UUID，这是一个不必集中管理的128位值。一个或多个元素可以编程为发布或订阅标签UUID，标签UUID不能被传输，应作为上层传输层消息完整性检查值的附加数据字段（见第3.8.7节）。

虚拟地址是一个16位的值，它的位15设置为1，位14设置为0，位13~0设置为哈希数值。此哈希数值是标签UUID的派生，因此每个哈希表示多个标签UUID。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_4%20hash.png)
当接收到具有匹配哈希值的虚拟地址的访问消息时，作为身份验证的一部分，上层传输层将每个对应的标签UUID用作附加数据，在找到匹配项之前发送消息。

控制消息不可以使用虚拟地址。

标签uuid可以按照[8]中的定义随机生成，配置客户端可以分配和跟踪虚拟地址，然而两个设备也可以使用一些带外（OOB）机制创建虚拟地址。与组地址不同，这些可以由相关设备商定，而不是需要在集中配置数据库中注册，因为它们不太可能重复分配。

虚拟地址的一个缺点是，在配置期间，需要多段消息将标签UUID传输到发布或订阅节点。

虚拟地址可以是0x8000到0xBFFF之间的任何值，如下面的图3.5所示。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_5%20virtal%20adress%20format.png)
*图3.5 虚拟地址格式*
注意：当将32位MIC和哈希值的大小进行因式分解时，两个使用相同应用密钥但不同标签uuid的匹配虚拟地址发生冲突的可能性只有1/246=1.42×10-14。
#### 3.4.2.4 组地址 
组地址是编程为零个或多个元素的地址，组地址的位15设置为1，位14设置为1，如下图3.6所示。0xFF00到0xFFFF范围内的组地址保留给固定组地址（见表3.4），0xC000到0xFEFF范围内的地址通常可用于其他用途。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_6%20group%20address%20format.png)
*图3.6 组地址格式*

组地址只能在消息的目标地址字段中使用，发送到组地址的消息应发送到订阅此组地址的模型的所有实例。

有两种类型的组地址：可以动态分配的组地址和固定的组地址，如下表3.4定义了固定组地址。

| 值 | 固定组地址名字 |  
| --- | ---|
| 0xFF00 - 0xFFFB | RFU |  
| 0xFFFC | all-proxies |  
| 0xFFFD | all-friends |  
| 0xFFFE | all-relays |  
| 0xFFFF | all-nodes |  
*表3.4 固定组地址*

发送到“all-proxies”地址的消息应由所有使能了代理功能的节点的主要元素处理。

发送到“all-friends”地址的消息应由所有使能了好友功能的节点的主元素处理。

发送至“all-relays”地址的消息应由所有使能了中断功能的节点的主要元件处理。

发送到“all-nodes”地址的消息应由所有节点的主元素处理。

### 3.4.3 地址有效性
表3.5显示哪些地址类型在源地址字段和目标地址字段中有效。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_5%20Address%20type%20and%20message%20field%20validity.png)
*表3.5 地址类型和消息的有效性*

表3.6显示了哪些地址类型可与设备密钥和应用程序密钥一起使用。

| 地址类型 | 与设备密钥的有效性 | 与应用密钥的有效性 |
| --- | --- | --- |
| 未分配地址 | NO | NO |
| 单播地址 | YES | YES |
| 虚拟地址 | NO | YES |
| 组地址 | NO | YES |
*表3.6 地址类型和访问层密钥类型有效性*
### 3.4.4 网络PDU
mesh "NetWork PDU"格式见表3.7，如下图3.7所示：

| 字段 | 位 | 备注 |
| --- | --- | --- |
| IVI | 1 | IV Index 最低位有效 |
| NID | 7 | 派生于NetKey, 用于标识"加密密钥"和"用于保护NetWork PDU私密密钥 |
| CTL | 1 | 网络控制 |
| TTL | 7 | 存活时间 |
| SEQ | 24 | 序列号 |
| SRC | 16 | 源地址 |
| DST | 16 | 目标地址 |
| transportPDU | 8~128 |transport Protocol Data Unit |
| NetMIC | 32~64 | 网络消息完整性 |
*表3.7 NetWork PDU 字段定义*

“NetWork PDU” 被使用从单个网络密钥派生的密钥（由NID字段标识）安全保护着。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_7%20NetWork%20PDU%20format.png)
*图3.7 NetWork PDU 格式*

#### 3.4.4.1 IVI
IVI字段包含IV引索的最低有效位，用于验证和加密此 "NetWork PDU"（请参阅第3.8.3节）。
#### 3.4.4.2 NID
NID字段包含一个7位网络标识符，允许更容易地查找，用于验证和加密此"NetWork PDU"的加密密钥和私钥（请参阅第3.8.6.3.1节）。
NID值与加密密钥和私钥一起从网络密钥派生，对于“主要网络消息”和“友元与其低功耗节点之间的专用网络消息”，其派生方式不同（请参见第3.8.6.3.1节）。
#### 3.4.4.3 CTL
CTL字段只有一个bit，该bit用于确定消息是控制消息还是访问消息的一部分，如表3.8所示。

1. 如果CTL字段设置为0，则NetMIC为32位值，"transportPDU"包含的是访问消息。
2. 如果CTL字段设置为1，则NetMIC为64位值，"transportPDU"包含的是控制消息。


| CTL 字段 | 消息类型 | NetMIC大小（bits） |
| --- | --- | --- |
| 0 | 访问消息 | 32 |
| 1 | 控制消息 | 64 |
*表3.8 CTL 字段类型和NetMIC大小*

#### 3.4.4.4 TTL
TTL字段是一个7位字段。定义了以下值：

1. 0 = 未中继且不会中继。
2. 1 = 可能已中继，但不会中继。
3. 2~126 = 可能已中继且可以中继。
4. 127 = 未中继且可以中继

该字段的初始值由发送层（下传输层、上传输层、访问层、基础模型、模型）或 应用来设置，并在作为中继节点运行时由网络层使用。

TTL值为0是的作用是允许节点传输一个它知道不会丟失的"NetWork PDU", 因此，接收节点可以确定发送节点是一个单广播无线链路。
TTL值为1或者比1大的作用不能用来做这样的决定。

#### 3.4.4.5 SEQ
SEQ字段是24位整数。,对于元素发起的每个新"NetWork PDU"，SEQ字段、IV Index字段和SRC字段（见第3.4.4.6节）的组合都应该是唯一的值。
#### 3.4.4.6 SRC
SRC字段是一个16位值，用于标识发起此"NetWork PDU"的元素。该地址类型应为单播地址。

SRC字段由发起元素设置，并且被作为中继节点原封不动地转发。

#### 3.4.4.7 DST
DST字段是一个16位值，用于标识此"NetWork PDU"指向的一个或多个元素。此地址应为单播地址、组地址或虚拟地址。

DST字段由发起节点设置，并且在不受网络层中作为中继节点的影响。

#### 3.4.4.8 TransportPDU
从网络层的角度来看，“TransportPDU”字段是一个八位字节的数据队列。当CTL位为0时，“TransportPDU”字段的最大值应为128位。当CTL位为1时，“TransportPDU”字段的最大值应为96位。

“TransportPDU”字段由发起者的下层传输层设置，而且不应被网络层更改。

#### 3.4.4.9 NetMIC
NetMIC字段是一个32位或64位字段（取决于CTL位的值），用于验证DST和TransportPDU没有被更改。

当CTL位为0时，NetMIC字段应为32位。当CTL位为1时，NetMIC字段应为64位。

NetMIC由每个传输或中继该“TransportPDU”的节点的网络层设置。

### 3.4.5 网络接口
网络层支持通过多个承载器发送和接收消息。承载的多个实例可以存在，承载的每个实例通过网络接口连接到网络层。为了允许在同一节点内的元素之间发送消息，使用本地接口。

例如，一个节点可以有三个接口：
1. 一个用于通过广告承载发送和接收消息的接口。
2. 两个GATT承载接口, 每个客户端连接都通过GATT连接。

接口提供输入和输出过滤器，过滤器可以使用 "特定于承载PUD"进行配置，也可以由在节点上公开的服务进行内部配置，例如Mesh代理服务（参见第7.2节）。

#### 3.4.5.1 输入过滤接口
输入过滤接口器决定是否将传入的Mesh消息传递到网络层，并进行进一步处理，或者是否将其丢弃。
#### 3.4.5.2 输出过滤接口
输出筛选器接口决定传出的Mesh消息是传递给承载者还是丢弃，连接到“广播承载”或“GATT承载”的输出滤波器的接口，应丢弃TTL值设置为1的所有消息。
#### 3.4.5.3 本地网络接口
本地网络接口允许在同一节点内的元素之间发送消息，节点应实现本地网络接口，通过本地网络接口接收消息后，应将本条消息传递到节点的所有元素。
#### 3.4.5.4 广播承载网络接口
广播承载网络接口允许使用广播承载发送消息（见第3.3.1节）。

1. 从网络层接收到未标记为中继的“NetWork PDU"时，广播承载网络接口应使用网络传输状态的值, 在广播承载上重传“NetWork PDU",（见第4.2.19节）。

2. 当从网络层接收到标记为中继的“NetWork PDU"时，广播承载网络接口使用中继重传状态的值，在广播承载上重传网络PDU（参见第4.2.20节）.

### 3.4.6 网络层行为
#### 3.4.6.1 中继特性
中继特性用于中继/转发由节点通过广播承载接收的“NetWork PDU"，此功能是可选的，如果支持，可以启用和禁用。如果代理特征得到支持，则GATT承载和广播承载都应得到支持。
#### 3.4.6.2 代理特性
代理特性用于中继/转发”GATT“和”广播承载者“之间的节点接收到的“NetWork PDU"。此功能是可选的，如果支持，可以启用和禁用。支持此功能时，在"Mesh Proxy Service"公开（参见第7.2节）。
#### 3.4.6.3 接收一个网络PDU
消息通过网络接口从承载者传送到网络层，该接口应采用其输入过虑器定义的过滤规则（见第3.4.5.1节）。如果消息通过输入过滤器，则将其传递到网络层以进行进一步处理。

接收到的每个网络PDU都可以用附加的元数据进行标记，这些元数据可在以后用于更改此消息的处理。

接收到消息后，节点应检查NID字段值是否与一个或多个已知的NID字段值匹配。如果NID字段值与已知的NID不匹配，则应忽略该消息。如果NID字段值与已知的NID匹配，则节点应根据匹配的每个已知网络密钥对消息进行身份验证。如果消息没有根据任何已知的网络密钥进行身份验证，则应忽略消息。如果消息确实根据网络密钥进行身份验证，则SRC和DST字段被视为有效（见第3.4.3节），并且消息不在网络消息缓存中（见第3.4.6.5节），则消息应由下层传输层处理。

如下议定义，当消息被重新传输时，重发时使用的"IV Index"应与接收时使用的"IV Index"相同。

1. 如果从广播承载传送的消息被下层传输层处理，则节点支持中继性并且开启，TTL字段值应为2或者比2更大的值，如果目的地址不是该节点上的元素的单播地址，则则TTL字段值应减去1, “NetWork PDU"应标记为中继，“NetWork PDU"应重新传输到连接到广播承载的所有网络接口。建议在接收“NetWork PDU"和中继“NetWork PDU"之间引入小的随机延迟，以避免多个中继之间同时接收“NetWork PDU"的碰撞。

2. 如果从GATT承载发送的消息被下层传输层处理，则节点应支持代理功能并且开启，并且TTL字段的值为2或更大，并且目的地址不是该节点上的元素的单播地址，则TTL字段的值应减去1，“NetWork PDU"应重新传输到所有网络接口。

3. 如果从广播承载传送的消息被下层传输层处理，则节点应支持代理功能并且开启，并且TTL字段的值为2或更大，则目的地址不是此节点上元素的单播地址，则TTL字段应减去1，并且“NetWork PDU"应重新传输到连接到GATT承载的所有网络接口。

图3.8说明了传入“NetWork PDU"的处理步骤示例：
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/network%20PDU%20proccess%20steps.jpg)
*图3.8 NetWork PDU处理步骤实例*

#### 3.4.6.4 发送一个网络PDU
消息被mesh子网上下文中的元素传输，mesh子网由唯一的网络密钥标识。

* IVI字段应设置为用于为mesh子网传输的"IV Index"值的最低有效位。
* NID字段应设置为与加密密钥和私钥相关的NID值，这两个密钥用于加密和混淆。
* CTL字段应由更高层设置。
* TTL字段应由更高层设置。
* SEQ字段应由网络层设置为元素的序号。然后，对于每个新的网络PDU，序列号应增加一个。
* SRC字段应由网络层设置为元素的单播地址。该元素是用来发送 "NetWork PDU"。
* DST字段应设置为单播地址、组地址或虚拟地址，以标识一个或多个目标元素，并应由下传输层、上传输层或访问层设置。
* TransportPDU字段应由更高层设置。
* NetMIC字段应按照第3.8.7.2节的规定设置。

消息应传送到所有网络接口。每个接口应采用其输出过滤器定义的滤波规则（见第3.4.5.2节）。如果 "NetWork PDU"通过输出过滤器，则应在承载上传输。
#### 3.4.6.5 网络消息缓存
为了减少不必要的安全检查和过度中继，节点应包括所有最近看到的"NetWork PDU"的网络消息缓存。如果接收到的"NetWork PDU"已经在网络消息缓存中，则不应处理该"NetWork PDU"，应立即丢弃。如果接收到"NetWork PDU"，且该"NetWork PDU"不在网络消息缓存中，则可以处理该"NetWork PDU"（例如，检查网络安全性），如果该"NetWork PDU"是有效的"NetWork PDU"，则应将其存储在网络消息缓存中。

节点不需要缓存整个网络PDU，并且可以仅缓存其一部分以进行跟踪，例如NetMIC、SRC/SEQ或其他值。

当网络消息缓存已满，且需要缓存传入的"NetWork PDU"时，传入的新"NetWork PDU"应替换已在网络消息缓存中的最旧"NetWork PDU"。

网络消息缓存应能够存储至少两个网络pdu，尽管强烈建议定义网络消息缓存大小，应该根据与预期网络密度来定义 。
## 3.5 下层传输层
下层传输层从上层传输层接收上 "Transport PDU"，并将这些消息传输到对端设备的下层传输层。这些 "Upper Transport PDU"可以适合于单个"Lower Transport PDU"，或者可以被分割成多个"Lower Transport PDU"。在接收到消息时，下层传输层处理 ”Lower Transport PDU"，也可以将多个PDU重新组装"Upper Transport PDU"，并在重新组装完成后将这些PDU发送到上层传输层。
### 3.5.1 字节顺序
该层中的所有多个八进制数值应以“大端”形式发送，如第3.1.1.1节所述。
### 3.5.2 下层传输PDU
"Lower Transport PDU"用于将"Upper Transport PDU"发送到另一个节点。"Lower Transport PDU"的第一个字节中的最最高有效位是SEG字段，用于确定"Lower Transport PDU"是为分段或未分段的消息的格式。根据"NetWork PDU"中CTL字段和"Lower Transport PDU"SEG字段的值，使用了四种格式，如下表3.9所示。
| CTL | SEG | Lower Transport PDU 格式 |
| --- | --- | --- |
| 0 | 0 | 未分段的访问消息 |
| 0 | 1 | 分段的访问消息 |
| 1 | 0 | 未分段的控制消息 |
| 1 | 1 | 分段的控制消息 |
*表3.9 “Lower Transport PDU”格式类型*

#### 3.5.2.1 未分段访问消息
未分段的访问消息用于传输单个"NetWork PDU"的 “Lower Transport PDU”。图3.9显示了未分段的访问消息，表3.10显示了此消息的字段。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/unsegmented%20Access%20message.jpg)
*图3.9 未分段访问消息*

| 字段 | 大小（bits） | 备注 |
| --- | --- | --- |
| SEG | 1 | 0=未分段消息  |
| AKF | 1 | 应用密钥标记 |
| AID | 6 | 应用密钥标识 |
| Upper Transport Access PDU | 40~120 | 上层传输访问消息  |
*表3.10 示分段访问消息格式*

* SEG字段应设置为0。
* AKF和AID字段应由上层传输层设置，而上层传输层主要根据用于加密访问有效负载的应用密钥或设备密钥去设置（见第3.6.4.1节）。
* “Upper Transport Access PDU”由上层传输层提供。
* 此消息没有SZMIC字段。上层传输层中的TranMIC应为32位值，就好像SZMIC字段的值为0一样。

#### 3.5.2.2 分段访问消息
分段访问消息用于传输“Upper Transport Access PDU”的一段, 图3.10显示了分段访问消息的图示，表3.11显示了此消息的字段。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/segmented%20Control%20message.jpg)
*图3.10 分段访问消息*
| 字段 | 大小（bits） | 备注 |
| --- | --- | --- |
| SEG | 1 | 1=分段消息  |
| AKF | 1 | 应用密钥标记 |
| AID | 6 | 应用密钥标识 |
| SZMIC  | 1 |  TransMIC 大小  |
| SeqZero | 13 | SeqAuth的最低有效位  |
| SeqO | 5 | 段偏移数  |
| SeqN| 5 | 最后一段编号  |
| Segment m | 8~96 | 上层传输访问消息的m段 |
*表3.10 分段访问消息格式*

* SEG字段应设置为1。
* SZMIC字段指示 在“Upper Transport Access PDU”中“TransMIC” 的大小，如果SZMIC字段设置为0，则“TransMIC”是32位值。如果SZMIC字段设置为1，则“TransMIC”是64位值。
* AKF和AID字段应由上层传输层设置，而上层传输层主要根据用于加密访问有效载荷的应用密钥或设备密钥去设置（见第3.6.4.1节）。
* SeqZero字段由上层传输层设置。
* SegO字段应设置为该“Upper Transport Access PDU”中对应为哪一段，段号基于零增长。
* SegN字段应设置为该“Upper Transport Access PDU”的最后一个段号。
* Segment m字段 ，应设置为”UpperTransport Access PDU“的子集。

对于除最后一段外的所有段，”Segment m“应为（12 * m）~ (12 * m +11)之间的字节。在最后一段中，”Segment m“应为(12 * m)~"消息末尾"之间的节点。在同一个“Upper Transport Access PDU”中，每个分段访问消息对于AKF、AID、SZMIC、SeqZero、SegN应具有相同的值

#### 3.5.2.3 未分段控制消息
未分段的控制消息用于传输段确认消息或传输控制消息，图3.11显示了未分段的控制消息，表3.12显示了此消息的字段。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/unsegment%20control%20message.jpg)
*图3.11 未分段控制消息*
| 字段 | 大小（bits） | 备注 |
| --- | --- | --- |
| SEG | 1 | 0=未分段消息 |
| Opcode | 7  | 0x00=段确认消息，0x01~0xFF = 传输控制消息操作码 |
| Parameters | 0~88 | 传输控制消息参数 |
*表3.12 未分段控制消息格式*

* SEG字段应设置为0。
* Opcode字段应设置为0x00（用于段确认消息）或适当的操作码（见表3.39）。
* Parameters字段是根据操作码的要求设置的。
##### 3.5.2.3.1 段确认消息
下层传输层使用段确认消息确认对端下层传输层接收到的段，段确认消息如图3.12所示，并在表3.13中定义。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_12%20segment%20comfirm%20message.png)
*图3.12 段确认消息*
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_13%20segment%20comfirm%20message%20format.png)
*表3.13 段确认消息格式*

* SEG字段应设置为0。
* 传输控制消息的操作码字段应设置为0x00。
* OBO字段应由接收到的消息直接寻址的节点设置为0，并由代表低功率节点确认该消息的友元节点设置为1。
* SeqZero字段应设置为正在确认的上层传输层消息的SeqZero字段。
* BlockAck字段应设置为指示接收的段。最低有效位，第0位，应表示段0；最有效的位，第31位，应代表段31。如果位n设置为1，则段n被确认。如果位n设置为0，则未确认段n。大于SegN的段的任何位应设置为0，并在接收时忽略。

如果接收到的段是在TTL设置为0的情况下发送的，建议在TTL设置为0的情况下发送相应的段确认消息。
#### 3.5.2.4 分段控制消息
分段控制消息在传输控制消息不适合单个 ”NetWork PDU" 时, 用于传输部分传输控制消息。 图3.13显示了分段控制消息的图示，表3.14显示了此消息的字段。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_13%20Segmented%20Control%20message.png)
*图3.13 分段控制消息*
| 字段 | 大小（bits） | 备注 |
| --- | --- | --- |
| SEG | 1 | 1 = 分段消息 |
| Opcode | 7 | 0x00 = 保留，0x01~0x7f = "Transport Control message" |
| RFU | 1 | 保留给未来使用 |
| SeqZero | 13 | SeqAuth的最低有效位 |
| SeqO | 5 | 段偏移 |
| SeqN | 5 | 最后一段编号 |
| Segment m | 8~64 |  “Upper Transport Control PDU”的m段 |
*表3.14 分段控制消息格式*

* SEG字段应设置为1。
* 操作码字段由上层传输层设置，表示字段参数的格式。值为0x00是保留，在收到时不应发送和忽略。
* SeqZero字段由上层传输层设置。
* SegO字段应设置为“Upper Transport Control PDU“的段号（基于零）。
* SegN字段应设置为 ”Upper Transport Control PDU“的最后一个段号（基于零）。
* ”Segment m“ 字段应设置为来自”Upper Transport Control PDU“的子集。”Segment m“应为( 8 * m ~ 8 * m + 7)之间的字节，但最后一段消息应设置为（8 * m）~ “消息末尾”的之间的字节 。

每个分段控制消息的操作码、SeqZero和SegN值应与“Upper Transport Control PDU"相同。

### 3.5.3 分段和重组
为了传输大于15个字节的”Upper Transport  PDU“，下传输层会对”Upper Transport  PDU“进行分段并重新组装。这些段传送到对端的下层传输层，主要采用了"block acknowledgment"方案，以最小化传播消息的次数。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/segmentation%20and%20reassembly%20for%20a%20two-segment%20PDU.jpg)
*图3.14 两段PDU的分割与重组实例*
图3.14显示了，正在发送的”Upper Transport  PDU“，该PDU有单字节长度的操作码字段，长度都为3字节的“网络密钥引索和应用密钥”的字段，AppKey有16个字节。这意味着，当使用应用程序密钥加密和验证时，“Upper Transport  PDU”是24个字节。
这被下层传输层分割成两段，分别段0和段1。每个网段都有一个表头，该表头标识网段号，然后传递到网络层，在整个网络层中对它进行计算处理。网络层使用该"NetWork PDU"的序列号对"NetWork PDU"进行加密，然后对这些消息进行模糊处理，以便只有NID（和IV索引）字节在明文中可见。因此，可以使用两个"NetWork PDU"安全地传送单一访问的消息。

”Upper Transport Access PDU“和 ”Upper Transport Control PDU“的分割过程是相同的，下面的描述将这两种PDU类型视为相同的，除非明确说明。

注：”Upper Transport Access PDU“ 和 ”Upper Transport Control PDU“的段大小不同

#### 3.5.3.1 分段
下层传输层将上层传输PDU分割成一个或多个下层传输PDU。下层传输层一次只能将”Upper Transport  PDU“的单个的分段访问消息或分段控制消息传输到同一目标地址。

下层传输层应仅为同一目的地的另一个"Upper Transport  PDU"传输分段消息，直到最后一个”Upper Transport  PDU“的所有段, 可以被 确认为”确认状态“或者”取消状态“，

如果"Upper Transport  PDU"可以使用未分段的消息格式容纳到单个"Lower Transport  PDU"中，则下层传输层应该使用未分段的消息来传输这个"Upper Transport  PDU"。

如果"Upper Transport  PDU"可以使用分段消息格式容纳到单个"Lower Transport  PDU"中，则下层传输层可以使用单个分段消息来传输该"Upper Transport  PDU"。否则，应使用两个或多个分段消息。

分段的消息在下层传输层被确认，但未分段的消息不被确认。当使用分段消息比未分段消息更有效地传输"Upper Transport  PDU"时，应使用单分段消息。

例如，如果消息被发送到访问层进行确认，多段消息已经被接收并对其进行了操作，并且该消息已丢失，则必须再次发送完整的多段消息。相反，如果应用确认消息以单个分段访问消息的形式发送，则应用确认消息将在下层传输层发送，可能使用该段的多个传输，从而从而消除再次重新传输多段消息的需要。

"Upper Transport Access PDU”的每段应为12个字节，但最后一段可能更短。
"Upper Transport Control PDU"的每段应为8个字节，但最后一段可能更短。

例如，当使用32位transMIC，如果"Upper Transport Access PDU”是42个字节，在段0中, 则是前12个字节（字节0~11），在段1中，则是第二组12个字节（字节12~23）；在段2中，则是第三组12个字节 (字节24~35），在段3中，则是剩下的6个字节（字节36~41）。

例如，如果"Upper Transport Control PDU"是42个字节，在段0中，则是前8个字节（字节0~7），在段0中，则是第二组8个字节（字节8~15），在段2中，则是第三组8个字节（字节15~23），在段3中， 则昌第四组8个字节（字节24~31），在段4中，第五组8个字节（字节32~39），在段5中，则是其余2个字节(字节40~41）。

使用SegO字段识别"Upper Transport Access PDU"的每个段。这些段使用"SeqAuth"值链接在一起，该值用于加密和验证"Upper Transport Access PDU"。

这些段使用SeqAuth值链接在一起，该值用于加密和验证"Upper Transport Access PDU"。每个用于"Upper Transport Access PDU"的"Lower Transport  PDU"应具有相同的IV索引, 该值用于加密和认证"Upper Transport Access PDU"的SeqAuth值。

使用SegO字段识别"Upper Transport Control PDU"的每个段, 这些段SeqAuth值连接在一起, 这个值使用用于发送"Upper Transport Control PDU"的第一段。每个用于"Upper Transport Access PDU"的"Lower Transport  PDU"应具有相同的IV索引，SeqAuth值用于识别"Upper Transport Access PDU"。

SeqAuth由IV索引和第一段的序列号（SEQ）组成，因此是大小56bit，其中IV索引是最高有效位，序列号是最低有效位。分段消息和分段确认消息中仅包括该值的最低有效13位（称为SeqZero）。在重新组合完整的分段访问消息时，SeqAuth值可以从任何分段中的IV Index、SeqZero和SEQ导出，通过确定SeqZero字段在SEQ-8191和SEQ 之间的最大SeqAuth值，例如，如果消息的接收到的SEQ是0x647262，IV索引是0x58437AF2，接收到的SeqZero值是0x1849，则SeqAuth值是0x58437AF2645849。如果消息的接收SEQ为0x647262，接收SeqZero值为0x1263，则SeqAuth值为0x58437AF2645263。

由于SeqZero的大小有限，一旦SEQ比SeqAuth高8192，就不可能发送分段消息。当SEQ比SeqAuth高8192时，如果分段消息尚未被确认，则应取消上层传输PDU的传送

消息的每个段都包括其段偏移量编号和最后一个段编号。

例如，在上述42个字节的"Upper Transport Control PDU"中，段编号为0、1、2、3、4和5，最后一个段编号为5。段号（SegO）和最后段号（SegN）都包含在消息中，在接收到消息的任何段后，允许接收器总是确定"Upper Transport  PDU"的大小（最接近8个字节）。

#### 3.5.3.2 重组
重组在接收设备中进行，当使用低功耗节点特性时，消息的确认由友元节点执行，并且低功耗节点不会发送段确认消息。

在接收到分段消息时，应检查SeqAuth值以确定"Upper Transport  PDU"是否正在接收或已经提前接收。如果分段消息尚未全部被接收，则接收设备应分配足够的内存空间，大小由最后一个段号（SegN）确定，在接收时存储"Upper Transport  PDU"的段，并跟踪其接收的段，然后它将认为该消息正在被接收。

如果低功率节点特性不在使用中，并且如果消息被发送到单播地址，并且如果节点此时无法接收该"Upper Transport  PDU"，例如因为节点正忙或资源不足而无法重新组合该消息，则该节点应向源节点发送信号，表明它无法通过接收此"Upper Transport  PDU"，方法是将BlockAck值设置为0x00000000。

如果分段消息正在接收过程中，则应使用段号（SegO）来确定该分段消息中的"Upper Transport  PDU"字节放在何处。接收器应更新BlockAck值，以记录段的成功交付。

一旦接收到所有段，上传输层应检查"Upper Transport  PDU"（见第3.6.4.2节）。

#### 3.5.3.3 分段行为
一旦"Upper Transport  PDU"被分段，下层传输层将发送该消息的每个分段的初始的"Upper Transport  PDU"。如果消息的目标地址是单播地址，则下层传输层将期望来自该节点或来自代表该节点的友元节点的段确认消息。如果消息的地址是虚拟地址或组地址，则这些设备不会发送段确认消息。
”Upper Transport  PDU"分段会形成很多段“Lower Transport  PDU”,如果分段的"Upper Transport  PDU"被发送到组地址或虚拟地址，则下层传输层应发送所有分段的"Upper Transport  PDU"。建议在发送"Lower Transport  PDU"的间隔引入小的随机延迟。

注意：发送到组或虚拟地址的“Lower Transport  PDU”未被收件人确认，因此消息传递状态未知，应视为未确认。上述推荐的行为旨在显著提高分段“Lower Transport  PDU”的成功传递的概率。

以下要求适用于当“Lower Transport  PDU”被发送到单播地址时。

1. 当发送“Lower Transport  PDU”时，应启动一个段传输定时器。定时器定义了段确认消息预计在何时收到，该计时器应设置为至少（200+50 * TTL）毫秒。

2. 如果段确认消息OBO字段设置为0，DST字段设置为该元素的单播地址，SeqZero字段设置为分段消息的SeqZero，SRC字段设置为分段消息的目标地址，则段确认消息是有效确认的分段消息。

3. 如果段确认消息OBO字段设置为1，并且DST字段设置为此元素的单播地址，并且SeqZero字段设置为分段消息的SeqZero，则段确认消息是有效确认的分段消息。对于给定的SeqAuth，只有来自第一个SRC地址接收到的段确认消息，才应被视为有效。
注意：接收到OBO字段设置为1的段确认消息，并不意味着该段消息已被传递到最终目的地，而只是它已被传递到该低功耗节点的朋友。消息存储在友元队列中，但如果其他消息被接收到该低功率节点或友元终止，则可以丢弃该消息。

4. 如果接收到的段确认消息是有效确认的分段消息，则下层传输层应重置段传输计时器，并重新传输所有未确认的“Lower Transport  PDU”。如果接收到段确认消息是确认"Upper Transport  PDU"的所有“Lower Transport  PDU”，则"Upper Transport  PDU"完成。如果接收到BlockAck字段设置为0x00000000的段确认消息，则应立即取消"Upper Transport  PDU"，并通知上层传输已取消。

5. 如果段传输计时器过期，并且没有收到有效确认的分段消息，则下层传输层应重新传输所有未确认的"Upper Transport  PDU"。
    注意：当重新传输每个较低传输PDU时，段传输计时器被重置并再次启动。

每个"Upper Transport  PDU"的"Lower Transport  PDU”应至少传输两次，除非提前确认。如果下层传输层在没有确认所有"Lower Transport  PDU”之前，就停止重新传输"Lower Transport  PDU”，则上层传输PDU也应该被取消。

#### 3.5.3.4 重组行为
本节仅适用于不使用低功耗特性的情况。
下层传输层具有序列认证值、块确认值和不完整计时器，计时器来自每个不完整的分段消息，而且源地址是相同。不完整分段消息是一种多段消息，它缺少部分分段，且其不完整计时器尚未过期。不完整计时器定义了下层传输层传输间隔等待的最大时间。

接收SeqAuth值大于序列认证值的多段消息的段的下层传输层，应为该不完整分段消息启动不完整计时器。不完整计时器应设置为至少10秒。

如果下层传输层接收到SeqAuth值小于序列认证值的消息段，则应忽略该段。如果下层传输层接收到新消息的段，则应将该段中的SeqAuth值保存为新的序列认证值。
注意：序列认证值逻辑上包括IV索引，因此，如果使用上次的IV索引接收"Lower Transport  PDU，则这将是小于序列认证值的SeqAuth值。

如果下层传输层接收到多段消息的一段，但此时无法接受该多段消息，因为它当前正忙或资源不足，无法接受该消息，并且如果该消息的目标地址是单播地址，下层传输层应使用段确认消息进行响应，段确认消息的BlockAck字段设置为0x00000000。

下层传输层应启动确认计时器，计时器定义下层传输层发送段确认消息的时间量。确认计时器应设置为至少（150+50 * TTL）毫秒。
如果在确认计时器处于非活动状态时，下层传输层接收到另一段的序列认证值，则应重新启动确认计时器。
    注意：如果下层传输层在确认计时器处于活动状态时接收到序列验证值，则确认计时器不会重新启动。
    
当不完整计时器处于活动状态时，如果较低传输层接收到用于序列认证的任何段，则应重新启动不完整计时器。

下层传输层应将接收到的每个段标记为块确认值，该块确认值可随后回传到源节点。
当接收到分段消息的所有段时，下层传输层应发送段确认消息，并且将BlockAck字段设置为序列认证值的块确认值。取消不完整定时器和确认定时器，并将重组后的报文发送到上层传输层。如果段是分段访问消息，则应按照第3.6.4.2节的规定处理重组的消息。如果段是分段控制信息，则应按照第3.6.5节的规定处理重组的消息。在至少10秒的计时器过期后，下层传输层应丢弃为发送方源地址存储的序列身份验证值和块身份验证值。

当确认计时器到期时，下层传输层应发送段确认消息，且BlockAck字段设置为序列认证值的块确认值。

当不完整计时器过期时，下层传输层应认为接收到的消息已失败，并取消确认计时器。任何一段的被取消消息，其序列认证值由下层传输层存储，应被忽略。在至少10秒的计时器过期后，下层传输层应丢弃为该源地址存储的序列身份验证值和块身份验证值。

如果下层传输层接收到序列认证值的另一段，并且消息已经完全接收（并且没有接收到具有新序列认证值的段），然后，它应立即发送一条段确认消息，并将BlockAck字段设置为该SeqAuth的块确认值。

如果设备作为低功率节点的友元节点，则它应重新组合目标地址为低功率节点的分段消息，并按所述操作，但它应在分段确认消息中将OBO字段设置为1。否则，OBO字段应设置为0。

注意：当接收到的分段消息是发往组或虚拟地址时，下层传输层不会启动确认计时器，并且不发送段确认消息。
#### 3.5.3.5 低动耗特性重组行为
本节仅在使用低功耗特性时适用。

下层传输层具有序列认证值和每个源设备的SeqAuth字段的块确认值。

如果下层传输层接收到SeqAuth值小于序列认证值的消息段，则应忽略该段。如果较低传输层接收到新消息的段，则应将该段中的SeqAuth值保存为新的序列认证值。

如果友元被终止（见第3.6.6.4.2节），则应取消先前部分接收的多段消息。

注意：序列认证值逻辑上包括IV索引，因此，如果使用先前的IV索引接收"Lower Transport  PDU”，则这将是小于序列认证值的SeqAuth值。

下层传输层应开始重新组合新消息，当接收SeqAuth值大于序列认证值的多段消息的段，并取消以前部分接收的任何消息。下传输层应将接收到的每个段标记为块确认值

当分段消息的所有消息段均已接收到时，下层传输层应将重组后的消息发送到上层传输层。如果段是分段访问消息，则应按照第3.6.4.2节的规定处理重新组装的消息。如果段是分段控制信息，则应按照第3.6.5节的规定处理重新组装的信息。

### 3.5.4 下层传输层行为

#### 3.5.4.1 发送一个下层传输PDU
"Lower Transport PDU"应传送到网络层。
#### 3.5.4.2 接收一个下层传输PDU
如果"Lower Transport PDU"是分段消息或分段确认消息，则应按照第3.5.3.4节中的定义进行处理。如果"Lower Transport PDU"是未分段的消息类型，则应按照第3.6.4.2节中的定义进行处理。如果"Lower Transport PDU"是传输控制PDU，则应根据3.6.5中定义的操作码字段的值对其进行处理。
### 3.5.5 友元队列
友元节点应为每个友元低功耗节点设置一个友元队列。友元队列存储低功耗节点的"Lower Transport PDU"。由于消息在友元队列中，则不应更改"Lower Transport PDU"的任何字段。

当朋友节点接收到目标地址为友元低功率节点的消息（即消息的目标地址是低功率节点的元素的单播地址或在朋友订阅列表中）并且TTL字段的值为2或更大时，TTL字段的值应减去1，并且消息应存储到朋友排队。

如果消息是分段访问消息或分段控制消息，则只有在成功地重新组合完整的"Lower Transport PDU"，并且友元节点已确认接收到所有分段之后，才应将消息存储到友元队列中。

如果友元队列已满，并且新消息需要存储，非友元更新消息，则应丢弃友元更新消息以外的最旧条目，以便为新消息腾出空间。

注意：一个实现可能必须丢弃多个消息，以便将新消息放入友元队列。

如果正在存储的消息是段确认消息，并且友元队列包含另一个段确认消息，该段确认消息具有相同的源地址和目标地址，并且具有相同的SeqAuth值，但具有较低的IV索引或序列号，则应丢弃较旧的段确认消息。

当友元节点进行安全更新时，例如通过接收有效的安全网络信标或通过改变密钥刷新阶段状态，它应将友元更新消息添加到友元队列中。当低功耗节点从友元队列请求消息时，应发送最旧的条目。一旦低功率节点确认该消息，则应丢弃该条目。如果友元节点使用友元轮询(Friend Poll)从低功耗节点轮询消息，并且该节点的友元队列为空，则友元节点应生成新的友元更新消息，并在发送响应之前将该消息添加到友元队列中，以便此好友更新消息可以作为对友元轮询(Friend Poll)消息的响应而发送。
## 3.6 上层传输层
上层传输层从访问层或内部生成的上层传输层控制消息获取访问有效负载，并将这些消息传输到对端上层传输层。对于来自访问层的消息，使用应用密钥对消息执行加密和身份验证。这允许接收上层传输层对接收到的消息进行身份验证。由上层传输层内部生成的传输控制消息仅在网络层加密和验证。
### 3.6.1 字节顺序
该层中的所有多个字节数值应以“大端”形式发送，如第3.1.1.1节所述。
### 3.6.2 上层传输访问PDU
当"NetWrok PDU"中的CTL字段为0时，"Upper Transport Access PDU"包含一个访问负载，称为上层传输访问PDU。

使用应用密钥或设备密钥对访问有效负载进行加密，并且将加密的访问有效负载和相关联的消息完整性检查值组合到"Upper Transport Access PDU"中。"Upper Transport Access PDU"字段如表3.15所示，如图3.15所示.
| 字段 | 字节 | 备注 |
| --- | --- | --- |
| 加密访问负载 | 1-380 | 这个是加密访问负载 |
| TransMIC | 4-8 | 访问负载的消息完整性检查值 |
*表3.15 上层传输访问PDU*
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_15%20Upper%20Transport%20Access%20PDU%20fields.png)
*图3.15 上层传输访问PDU格式*
#### 3.6.2.1 加密访问负载
访问负载由访问层提供。如果"TransMIC"字段是32位的，则访问有效负载的长度可以为（1~380）个字节。如果"TransMIC"字段是64位的，则访问有效负载的长度可以为（1~376）个字节。在上层传输层，此字段不透明，不能使用此字段中的任何信息。
#### 3.6.2.2 TransMIC
TransMic(The Message Integrity Check for Transport)是一个32位或64位的字段，对于分段消息，其中SEG被设置为1，“TransMIC”大小由“Lower Transport PDU”中SZMIC字段的值确定。对于未分段的消息，对于数据消息，"TransMIC"的大小为32位。
注意：控制消息没有“TransMIC”字段。
### 3.6.3 上层传输控制PDU   
当CTL位为1时，上层传输PDU包含一个传输控制消息。传输控制消息具有确定参数格式的7位操作码（Opcode）。此操作码字段不包含在参数字段中，但包含在下层传输层PDU未分段控制消息或分段控制消息的每个段中。

上层传输控制PDU没有在上层传输层进行身份验证，而是依赖于由网络层执行的身份验证。所有上层传输控制PDU使用64位NetMIC字段。

下层传输层可以将消息分割成较小的pdu，以便通过网络层传送。因此，建议保持传输控制PDU有效负载大小，如表3.16所示，其中的值表示取决于数据包数量的最大有用参数字段大小。
| 数据包数量 | 传输控制PDU的负载大小 |
| --- | --- |
| 1 | 11（未分段）|
| 1 | 8（分段） |
| 2 | 16 |
| 3 | 24 |
| n | 8 * n |
| 32 | 256 |
*表3.16 最大有用传输控制PDU的负载大小*
上层传输控制PDU的最大值为256个字节。
### 3.6.4 上层传输层行为
#### 3.6.4.1 发送一个访问负载
所有访问消息都在应用密钥或设备密钥的上下文中发送。应使用该应用密钥或设备密钥对访问有效负载进行加密，并且应将TransMIC字段设置为第3.8.6节中定义的消息完整性检查值。然后应按照第3.5.4.1节的规定处理上层传输PDU。

应为此消息分配一个序列号（SEQ），在下层传输层中分段的消息的上下文中，该SEQ字段对应于SeqAuth字段的24个最低比特，如第3.5.3.1章中定义，序列号用于由接收器对访问消息进行身份验证和解密的。

下层传输PDU的AKF字段和AID字段，应根据用于加密和验证上层传输PDU的应用密钥或设备密钥进行设置。如果使用了应用密钥，则AKF字段应设置为1，而AID字段应设置为应用密钥标识符（AID）。如果使用设备密钥，则AKF字段应设置为0，辅助字段应设置为0b000000。

上层传输层不应将上层传输PDU的新分段传输到给定的目标地址，直到到该目标地址的上一个传输PDU已完成或取消。
#### 3.6.4.2 接收一个上层传输PDU
在接收到上层传输访问PDU时，应解密访问有效负载，并根据所有已知的应用密钥或设备密钥对TransMIC进行身份验证。其中应用密钥和设备密钥是AKF字段和AID字段要匹配。如果上层传输访问PDU进行身份验证，并且已检查其是否存在重播攻击（请参阅第3.8.8节），则它将与此消息的上下文信息（如源地址、目标地址以及用于解密和身份验证的密钥）一起传递到访问层。

在接收到上层传输控制PDU时，PDU的目的地址应与该节点的元素的单播地址进行核对，如果匹配，则应处理该消息（见第3.6.6节）。

如果此节点支持友元特性，并且此功能已启用，则该节点与低功耗节点具有友元关系，并且此消息的目标地址是当前位于此低功耗节点的好友订阅列表中的地址，然后将消息存储在相应的友元队列中。
### 3.6.5 传输控制消息
传输控制消息可以使用单个未分段控制消息或分段控制消息序列来传输。每个消息都有一个7位操作码字段，用于确定参数字段的格式。每个传输控制信息应以尽可能少的下层传输PDU发送。操作码0x00消息在下层传输层终止，用于消息的分段和重组，不应被上层传输层发送。所有其他控制消息在上层传输层终止。
有关传输控制消息操作码的摘要，请参见表3.39。
#### 3.6.5.1  Friend Poll
"Friend Poll"消息由低功耗节点发送，以要求友元节点发送为低功耗节点存储的消息。
“Friend Poll”消息参数在表3.17中定义。
| 字段 | 大小(bits) | 备注 |
| --- | --- | --- |
| Padding | 7 | 0b0000000. 禁止所有其他值 |
| FSN | 1 | 友元序列号，用于确认已收到来自友元节点到低功耗节点的先前消息 |
*表3.17 "Friend Poll"消息参数*

1. 传输控制消息的操作码字段应设置为0x01。
2. FSN字段应设置为0或1，如第3.6.6.4.2节所定义。
3. 此消息应将“NetWrok PDU"的TTL字段设置为0。
4. 此消息应使用友元安全凭据发送。
#### 3.6.5.2  Friend Update
"Friend Update"消息由友元节点发送到低功耗节点，以通知低功耗节点网络的安全参数（见第3.6.6.1节）已更改或正在更改，或者友元队列为空。"Friend Update"消息参数在表3.18中定义。
| 字段 | 字节（bits） | 备注 |
| --- | --- | --- |
| Flags | 1 | 包含"IV Update"标志和"Key Refresh"标志 |
| IV Index | 4 | 友元节点已知的当前IV索引值 |
| MD | 1 | 指示友元队列是否为空 |
*表3.18 Friend Update 消息参数*

1. 传输控制消息的操作码字段应设置为0x02。
2. Flags字段是表3.19中定义的8位字段：

| 位 | 定义  |
| --- | --- |
| 0 | Key Refresh Flag，0: Not-In-Phase2 ，1: In-Phase2 |
| 1 | IV Update Flag，0: Normal operation，1: IV Update active |
| 2~7 | RFU|
*表3.19 Flags字段格式*

*  "Key Refresh Flag"指示密钥刷新过程是否正在进行（请参阅第3.10.4节).
*  "IV Update Flag"指示IV更新过程是否正在进行（请参阅第3.10.5节）。
 
3. IV Index字段包含当前的IV索引。
4. MD字段设置为指示友元队列是否为空，如表3.20所定义。
| 值 | 描述 |
| --- | --- |
| 0 | 友元队列为空 |
| 1 | 友元队列不为空 |
| 2-255 | 禁止 |
*表3.20 MD字段格式*
* 此消息应将"NetWork PDU"的TTL字段设置为0。
* 此消息应使用友元安全凭据发送。
#### 3.6.5.3  Friend Request
”Friend Request“消息由低功耗节点发送到“所有友元组"以开始查找好友。
”Friend Request“消息参数在表3.21中定义。友元节点应支持的条件

| 字段 |大小（bits）| 备注 |
| --- | --- | --- |
| Criteria | 1| 友元节点应支持的标准，为了参加友元谈判 |
| ReceiveDelay | 1 |低功耗节点请求的接收延迟 |
| PollTimeout | 3 | 低功耗节点设置的PollTimeout计时器的初始值 |
| PreviousAddress | 2 | 前一个朋友的主要元素的单播地址 |
| NumElements | 1 | 低功耗节点中的元素数 |
| LPNCounter | 2 | 低功耗节点已发送的Friend Request消息数 |
*表3.21 Friend Request 消息参数*

1. 传输控制消息的操作码字段应设置为0x03。
2. Criteria字段格式见表3.22。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_22%20Criteria%20field%20format%20.png)
*表3.22 Criteria字段格式*
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_16%20Criteria%20field%20format.png)
*图3.16 Criteria字段格式*

* RSSIFactor字段用于“Friedn Offer Delay”计算（见第3.6.6.3.1节），其值在表3.23中定义。

| 值 | RSSIFactor | 
| --- | --- |
| 0b00 | 1 |  
| 0b01 | 1.5 |  
| 0b10 | 2 |  
| 0b11 | 2.5 |
*表3.23 RSSIFactor字段值*

* ReceiveWindowFactor字段用于“Friedn Offer Delay”计算（见第3.6.6.3.1节），其值在表3.24中定义。

| 值 | ReceiveWindowFactor | 
| --- | --- |
| 0b00 | 1 |  
| 0b01 | 1.5 |  
| 0b10 | 2 |  
| 0b11 | 2.5 |
*表3.23 ReceiveWindowFactor字段值*

* MinQueueSizeLog字段定义为log2（N），其中N是友元节点可以存储在其友元队列中的最大尺寸下层传输pdu的最小数目，并且具有表3.25中定义的值之一。

| 值 | 描述 |
| --- | --- |
| 0b000 | 禁止 |
| 0b001 | 2 |
| 0b010 | 4 |
| 0b011 | 8 |
| 0b100 | 16 |
| 0b101 | 32 |
| 0b110 | 64 |
| 0b111 | 128 |
*表2.25 MinQueueSizeLog字段值*

3. ReceiveDelay字段应设置在表3.26规定的有效范围内。

| 值 | 描述 |  
| --- | --- | 
| 0x00-0x09 | 禁止 | 
| 0x0A–0xFF | 以1毫秒为单位的接收延迟  | 
*表3.26 ReceiveDelay字段值*

4. PollTimeout字段应设置在表3.27中定义的有效范围内。

| 值 | 描述 |  
| --- | --- | 
| 0x000000–0x000009 | 禁止 | 
| 0x00000A–0x34BBFF | 以100毫秒为单位  | 
| 0x34BC00–0xFFFFFF | 禁止 |
*表3.27 PollTimeout字段值*

5. PreviousAddress字段应设置为前一个友元的单播地址，如果之前没有建立友元关系，则应设置为未分配的地址。
6. NumElements字段应设置为低功耗节点上的元素数。友元节点使用该值计算单播地址范围。使用此消息的SRC地址分配给低功耗节点。
* 表3.28中定义了NumElements字段的值。

| 值 | 描述 |  
| --- | --- | 
| 0x00 | 禁止 | 
| 0x01–0xFF | 元素的数量  | 
*表3.28 NumElements字段值*

7.LPNCounter字段值设置为低功耗节点已发送的“Friend Request”消息数。此消息应将网络PDU的TTL字段设置为0。此消息应使用主安全凭据发送。

#### 3.6.5.4  Friend Offer
Friend Offer消息由Friend节点发送以提供友元关系 。Friend Offer消息参数定义见表3.29。

| 字段 | 大小（bits） | 备注 |
| --- | --- | --- |
| ReceiveWindow | 1 | 友元节点支持的接收窗口值 |
| QueueSize | 1 | 友元节点上可用的队列大小 |
| SubscriptionListSize | 1 | 低功耗节点的友元节点可以支持的订阅列表的大小 |
| RSSI | 1 | 由友元节点测量的RSSI |
| FriendCounter | 2 | 友元节点具有的Friend Offer消息数 |
*表3.29 Friend Offer 参数*

1. 传输控制消息的操作码字段应设置为0x04。
2. ReceiveWindow字段的值在表3.30中定义。

| 值 | 描述 |  
| --- | --- |
| 0x00 | 禁止 |  
| 0x01-0xFF | 接收窗口，单位为1毫秒 |  
*表3.30 ReceiveWindow字段值*

3. QueueSize字段包含友元节点可以为低功耗节点存储的消息数。
4. SubscriptionListSize字段包含友元节点可以为低功耗节点支持的订阅列表中的条目数。
5. RSSI字段包含一个有符号的8位值，并意思为以dBm为单位测量的接收信号强度的指示。这是发送Friend Request的友元节点测量的。如果信号强度指示不可用，则该值应为0x7F（127 dBm）
6. FriendCounter字段值设置为友元节点已发送的Friend Offer消息数。此消息应将网络PDU的TTL字段设置为0。此消息应使用主安全证书发送。

#### 3.6.5.5  Friend Clear
Friend Clear消息被发送到友元节点，以通知其删除友谊。Friend Clear消息参数在表3.31中定义。

| 字段 |大小（octets）| 备注 |
| --- | --- | --- |
| LPNAddress | 2 | 正在删除的低功耗节点的单播地址 |
| LPNCounter | 2 | 新关系的LPNCounter值 |
*表3.31 Friend Clear参数*

1. 传输控制报文的操作码字段应设置为0x05。
2. LPNAddress字段应设置为要删除的低功耗节点的单播地址。
3. LPNCounter字段应设置为用于建立关系的最新的Friend Reuest消息的LPNCounter值。

这是一条确认消息，发送节点希望收到“Friend Clear Comfirm”消息作为响应。如果友元节点未收到响应，则新友元节点应以双倍间隔（2、4、8、16秒等）重新发送消息，直到它收到响应或达到低功耗节点轮询超时。
此消息应使用主安全凭据发送。

#### 3.6.5.6  Friend Clear Comfirm
"Friend Clear Comfirm"消息由旧友元节点响应“Friend Clear”消息，以确认友元关系 已终止。如果收到的“Friend Clear”消息的TTL为0，则确认消息也应使用TTL为0。“Friend Clear Confirm”消息参数在表3.32中定义。

| 字段 | 大小（octets） | 备注 |
| --- | --- | --- |
| LPNAddress | 2 | 正在删除的低功耗节点的单播地址  |
| LPNCounter | 2 | 与"Friend Clear“消息的LPNCounter字段一致 |
*表3.32 “Friend Clear Confirm”消息参数*

1. 传输控制消息的操作码字段应设置为0x06。
2. LPNAddress字段应设置为已删除的低功耗节点的单播地址。
3. LPNCounter字段应设置为来自相应Friend Clear消息的LPNCounter值

只有在前一个友元关系的轮询超时内收到有效”Friend Clear“的消息时，才应发送该消息，但它应该为在该时间段内收到的每个有效的”Friend Clear“消息发送Friend Clear Confirm消息。如果”Friend Request“消息的LPNCounter字段（发起友谊的消息）的值减去"”Friend Clear“"消息的LPNCounter字段的值（模块65536）的结果在0到255之间（包括0到255），则”Friend Clear“消息被视为有效。
此消息应使用主安全凭据发送。

#### 3.6.5.7  Friend Subscription List Add
友元订阅列表添加（Friend Subscription List Add）消息由低功耗节点发送到友元节点，以指示要为其存储消息的组地址和虚拟地址的列表。
表3.33定义了好友订阅列表添加消息参数。

| 字段 | 大小（octets） | 备注 |
| --- | --- | --- |
| TransactionNumber | 1 | 识别事务的号码 |
| AddressList | 2 * N | 组地址和虚拟地址列表，其中N是此消息中的组地址和虚拟地址数 |

*表3.33 Friend Subscription List Add 消息参数*

1. 传输控制消息的操作码字段应设置为0x07。
2. TransactionNumber字段用于区分每个单独的事务（请参阅第3.6.6.4.3节）。
3. “AddressList”字段应包含要添加到友元订阅列表中的组地址和虚拟地址列表。注意：当此消息作为未分段的控制消息发送时，地址列表字段不能包含超过5个地址。

此消息应将"NetWork PDU"的TTL字段设置为0。此消息应使用友元安全凭据发送。

#### 3.6.5.8  Friend Subscription List Remove
友元订阅列表删（Friend Subscription List Remove）除消息由低功耗节点发送到友元节点，以指示要从友元订阅列表中删除的组地址和虚拟地址。
表3.34定义了友元订阅列表删除消息参数。

| 字段 | 大小（octets） | 备注 |
| --- | --- | --- |
| TransactionNumber | 1 | 识别事务的号码 |
| AddressList | 2 * N | 组地址和虚拟地址列表，其中N是此消息中的组地址和虚拟地址数 |
*表3.34 Friend Subscription List Remove消息参数*

1. 传输控制消息的操作码字段应设置为0x08。
2. TransactionNumber字段用于区分每个单独的事务（请参阅第3.6.6.4.3节）。
3. “AddressList”字段应包含要从好友订阅列表中删除的组地址和虚拟地址列表。注意：当此消息作为未分段的控制消息发送时，地址列表字段不能包含超过5个地址。

此消息应将"NetWork PDU"的TTL字段设置为0。此消息应使用友元安全凭据发送。

#### 3.6.5.9  Friend Subscription List Comfirm
友元订阅列表确认（Friend Subscription List Comfirm）消息由友元节点发送到低功耗节点，以响应”Friend Subscription List Add“消息或”Friend Subscription List Remove“消息。

好友订阅列表确认消息参数定义见表3.35。

| 字段 | 大小（obtets） | 备注 |
| --- | --- | --- |
| TransactionNumber | 1 | 识别事务的号码 |
*表3.35 Friend Subscription List Comfirm消息参数*
1. 传输控制消息的操作码字段应设置为0x09。
2. TransactionNumber字段用于区分每个单独的事务（请参阅第3.6.6.4.3节）。

此消息应将"NetWork PDU"的TTL字段设置为0。此消息应使用友元安全凭据发送。

#### 3.6.5.10 Heartbeat
Heartbeat消息由一个节点发送，让其他节点确定子网的拓扑结构。
Heartbeat消息参数在表3.36中定义。

| 字段 | 大小（bits） | 备注 |
| --- | --- | --- |
| RFU | 1 | 保留位 |
| InitTTL | 7 | 发送消息时使用的初始TTL |
| Features | 16 | 节点当前活动特征的位字段 |
*表3.36 Heartbeat消息参数*

1. 传输控制消息的操作码字段应设置为0x0A。
2. InitTTL字段包含发送消息时使用的初始TTL。InitTTL的值在表3.37中定义。

| 值 | 描述 |
| --- | --- |
| 0x00–0x7F | 发送消息时使用的初始TTL |
*表3.37 InitTTL值定义*
3. Features字段值包含一个bit字段，指示节点当前使用的特性，如第3.1节所定义。特性字段的格式见表3.38。
Bit Feature Notes
0 Relay Relay feature in use: 0 = False, 1 = True
1 Proxy Proxy feature in use: 0 = False, 1 = True
2 Friend Friend feature in use: 0 = False, 1 = True
3 Low Power Low Power feature in use: 0 = False, 1 = True
4–15 RFU Reserved for Future Use
| bit | 特性 | 备注 |
| --- | --- | --- |
| 0 | Relay | 0 = False, 1 = True |
| 1 | Proxy | 0 = False, 1 = True |
| 2 | Friend | 0 = False, 1 = True |
| 3 | Low Power  | 0 = False, 1 = True |
| 4-15 | RFU | 保留 |
*表3.38 Features字段值*

#### 3.6.5.11 操作码摘要
表3.39提供了传输控制消息使用的操作码摘要。

| 值 | 操作吗 | 备注 |
| --- | --- | --- |
| 0x00 | – | 为下层传输层预留 |
| 0x01 | Friend Poll  | 由低功耗节点发送到其友元节点,以请求其为低功耗节点存储的任何消息 |
| 0x02 | Friend Update | 由友元节点发送到低功耗节点,以通知其安全更新 |
| 0x03 | Friend Request | 由低功耗节点发送的所有友元固定组地址开始查找好友 |
| 0x04 | Friend Offer | 由友元节点发送到低功耗节点,以提供成为其朋友 |
| 0x05 | Friend Clear | 发送到友元节点，以通知前一个低功耗节点的友元节点，有关删除友元关系的信息 |
| 0x06 | Friend Clear Confirm | 从以前的友元节点发送到友元节点，以确认以前的友元关系已被删除 |
| 0x07 | Friend Subscription List Add| 发送到友元节点以将一个或多个地址添加到友元订阅列表 |
| 0x08 | Friend Subscription List Remove | 发送到友元节点以从友元订阅列表中删除一个或多个地址 |
| 0x09 | Friend Subscription List Confirm | 由友元节点发送，以确认友元订阅列表更新 |
| 0x0A | Heartbeat |由节点发送，以让其他节点确定子网的拓扑  |
| 0x0B-0xFF | RFU | 保留 |
*表3.39 传输控制消息的操作码摘要*
### 3.6.6 友元关系
友元节点可以存储低功耗节点的消息。
#### 3.6.6.1 功能概述
为了优化低功耗节点的功耗，采用轮询机制最小化低功耗节点的接收窗口。这允许低功耗节点何时可以接收消息 ，要根据接收来自友元节点的更新。友元关系定义了在低功耗节点和友元节点之间的友元关系期间静态的计时参数。
友元关系使用以下计时参数：
* ReceiveDelay
* ReceiveWindow
* PollTimeout

1. 接收延迟（ReceiveDelay）是低功耗节点发送请求和侦听响应之间的时间。此延迟允许友元节点准备响应的时间。
2. ReceiveWindow是低功耗节点侦听响应的时间。当低功耗节点从其友元节点接收到消息时，它可以停止监听其他消息。
请求可以是"Friend Poll"消息、"Friend Subscription List Add"消息或"Friend Subscription List Remove"消息。
"Friend Poll"消息的响应可以是"Friend Update"消息或存储的消息。
"Friend Subscription List Add"消息或"Friend Subscription List Remove"消息的响应是"Friend Subscription List Confirm"消息。
计时参数如图3.17所示。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_17%20Friendship%20timing%20parameters.png)
*图3.17 友元关系的计时参数*

3. PollTimeout计时器用于测量低功耗节点发送的两个连续请求之间的时间。如果在PollTimeout计时器过期之前Friend节点没有收到请求，则认为友元关系已终止。如图3.18所示。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_18%20%20Poll%20Timeout%20timer%20illustration.png)
*图3.18 PollTimeout计时器示例*

为了建立友元关系，支持低功耗功能的节点向allfriends地址发送”Friend Request"消息 。该消息由该节点无线电范围内支持友元特性的所有节点接收。
”Friend Request"消息包含许多参数，这些参数定义了此节点需要任何未来友元节点支持的需求。支持友元特性的每个节点，它可以支持通过向请求节点发送“Friend Offer”消息来响应”Friend Request"消息的要求。这些Offer消息还包括有关每个产品节点功能的附加信息。这允许低功耗节点确定它将接受哪些服务。

低功耗节点向其选择的友元节点发送“Friend Request”消息，然后，友元节点用“Friend Update”消息进行响应。此时，友谊建立起来，如图3.19所示。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_19%20Establishment%20of%20a%20friendship.png)
*图3.19 建立友元关系*
如果低功耗节点以前是另一个友元节点的朋友，则新友元节点通知旧友元节点它现在是该低功耗节点的当前朋友（请参阅第3.6.6.3.1节）。

建立友元关系后，友元节点存储低功耗节点的友元订阅列表，该列表是低功耗节点订阅的组和虚拟地址的集合。此列表允许友元节点存储低功耗节点订阅的消息。

给定网络密钥的“IV Index”、IV Update标志和密钥刷新标志的值称为安全参数。每当安全参数的至少一个值被更改时，新的安全参数称为安全更新。

友元节点存储低功耗节点的所有消息，，以及低功耗节点的在友元队列中最新安全更新。它们统称为存储消息。

为了获取存储的消息，低功耗节点发送Friend Poll消息，友元节点使用存储的消息进行回复。

存储在友元队列中的消息通过确认按顺序传递到低功耗节点。要启用此功能，请使用友元序列号。此值存储在低功耗节点上，并在每个Friend Poll消息中发送。当低功耗节点接收到响应“Friend Poll”的消息，并且此消息已使用友元安全凭据成功验证时，低功耗节点应更改此友元序列号。下次此低功耗节点轮询(POLL)时，友元节点可以发送下一条消息。如果对Friend Poll消息没有响应，那么低功耗节点不会更改友元序列号，友元节点可以确定它发送的最后一条消息没有收到，然后重新发送。
#### 3.6.6.2 友元安全
当友元节点和低功耗节点建立了友元关系时，从友元节点到低功耗节点的所有网络消息都是安全的，通过使用友元安全凭据创建的友元安全材料（请参阅第3.8.6.3.1节）。友元安全证书在低功耗建立过程中交换（见第3.6.6.4.1节）。“Friend Poll”，“Friend Update”，“Friend Subscription List Add/Remove/Confirm”消息，以及友元节点传递给低功耗节点的存储消息始终使用友元安全凭据进行加密。“Friend Clear”和“Friend Clear Confirm“消息始终使用主安全凭据加密。根据“发布友元凭据”标志的值（请参阅第4.2.2.4节），低功耗节点使用友元安全凭据或主安全凭据发送消息（请参阅第3.8.6.3.1节）。所有其他网络消息都使用主安全凭据发送。

图3.20说明了使用友元安全凭据（虚线）和主安全凭据（实线）保护的消息。

1. 低功耗节点首先使用主安全凭据发送Friend Request消息。
2. 友元节点以Friend Offer消息响应，再次使用主安全凭据。低功耗节点和友元节点都必须使用主安全凭据，因为两个设备都不与另一个设备处于友好状态，因此无法使用友元安全凭据。
3. 低功耗节点接受”friend Offer“，并使用友元安全凭据发送一个”Friend Poll“消息来确认这一点。
4. 友元节点将使用Friend Update消息对此做出响应。
5. 低功耗节点现在可以使用“Friend Subscription List Add”消息配置友元订阅列表，使用友元节点的“Friend Subscription List Add Confirm”消息确认。这两条消息都是使用友元安全凭据发送的。
6. 稍后，友元节点从另一个设备接收消息（InMsg）,这个消息需要传送到低功耗节点，所以它会将此消息存储在友元队列中。
7. 低功耗节点发送一个Friend Poll消息，该消息使用友元安全凭据进行保护，友元节点将用在友元队列中消息回复该消息。
8. 然后，低功耗节点决定发送两条消息：OutMsg1和OutMsg2。OutMsg1是使用友元安全凭据安全发送的，因此只有友元节点将接收和中继此消息。当友元节点中继Outmsg1时，将使用主安全凭据重新传输消息。OutMsg2使用主安全凭据安全发送，因此友元节点和低功耗节点范围内的任何其他中继节点都可以中继消息。OutMsg2在中继时，将使用主安全凭证重新传输。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_20%20Messages%20secured%20with%20friendship%20and%20master%20security%20credentials.png)
*图3.20 消息友元安全凭据和主安全凭据*

#### 3.6.6.3 友元特性
友元特性定义了三个必需的操作：

* 友元建立（Friend establishment）
* 友元消息传递 （Friend messaging）
* 友元管理 （Friend messaging）

友元节点发起的所有传输控制消息，应作为未分段的控制消息发送，SRC字段设置为友元节点的主要元素的单播地址

##### 3.6.6.3.1 Friend establishment
支持友元特性，并启用该功能，并接收“Friend Request”消息的节点（请参阅第3.6.5.3节），满足消息参数规定的最低要求的，应使用“Friend Offer”消息进行响应（见第3.6.5.4节）。

如果“Friend Request"消息的源地址是低功耗节点的地址，并且与当前友元节点处于友元关系中，那么友元节点还应考虑终止与该低功耗节点的现有友元关系。

发送Friend Offer消息时，目标地址应等于”Friend Request“消息的源地址，TTL值应设置为0。

节点应保留一个友元节点计数器（Friend counter），它是初始化为0。此值应在”Friend Offer“消息中发送，如果由于”Friend Offer“息而建立了友谊，则应用于派生友元安全材料。每次发送”Friend Offer“消息后，FriendCounter值应增加1。

从收到”Friend Request“消息到”Friend Offer“消息之间的时间称为”Friend Offer Delay“，该值根据RSSIFactor字段和ReceiveWindowFactor字段计算。

"Friend Offer Delay"允许低功耗节点从潜在友元节点接收Friend Offer消息，以确定所提供的”ReceiveWindow“有多大，以及信号质量有多重要。一些低功耗节点更喜欢”ReceiveWindow“值很小的友元节点，因此将“ReceiveWindowFactor”设置为比“RSSIFactor”更重要。这意味着，对于那些符合低功耗节点要求的节点，低功耗节点应该更快地收到来自朋友的“Friend Offer”，降低低功耗节点在寻找友元节点时的功耗。

局部延迟的计算公式如下：
Local Delay = ReceiveWindowFactor * ReceiveWindow - RSSIFactor * RSSI

* ReceiveWindowFactor是来自“Friend Request"消息的数字。
* ReceiveWindow是要在相应的”Friend Offer“消息中发送的值。
* RSSIFactor是来自“Friend Request"消息的数字。
* RSSI是在友元节点上接收到的“Friend Request"消息的接收信号强度。

如果本地延迟(Local Delay)值大于100，则应将Friend Offer Delay值（以毫秒为单位）设置为本地延迟值。否则，Friend Offer Delay设置为100毫秒。

如果节点在发送”Friend Offer“消息后1秒内,收到"Friend Poll"消息，则建立友元，并保存此"Friend Poll"消息的FSN字段值；否则，建立失败。

友元节点应在接收到来自低功耗节点的"Friend Poll"消息后,应在最小ReceiveDelay毫秒~最大(ReceiveDelay+ReceiveWindow)毫秒区间，使用“Friend upadte”消息进行响应。

图3.21展示了一个友元关系建立，其中多个启用了友元特性的节点接收相同的“Friend Request"消息。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_21%20Friend%20establishment%20example.png)
*图3.21 友元关系建立*
建立友元关系后，友元节点应将友元订阅列表初始化为空列表，并开始在友元队列中存储低功耗节点的消息。建立友元关系后，如果“Friend Request"消息的PreviousAddress字段包含一个有效的单播地址，该地址不是友元节点自己的单播地址，则友元节点应按照以下步骤开始向该单播地址发送朋友清除消息（见第3.6.5.5节）：
1. TTL应设置为最大有效值。
2. 友元关系一建立，第一个"Friend Clear"消息就要发出；同时，以1秒为周期启动"Friend Clear"消息重复计时器，"Friend Clear"过程计时器的启动周期应等于"Friend Poll"超时值的两倍。
3. 如果收到"Friend Clear Confirm"确认消息（见第3.6.5.6节）作为对朋友"Friend Clear"的响应，则应取消两个计时器并完成程序。
4. 如果"Friend Clear"重复计时器过期，则应发送新的"Friend Clear"消息，并重新启动计时器，重新启动的时间段应是前一个"Friend Clear"重复计时器时间段的两倍。例如，第一次过期后，周期应设置为2秒；下一次过期时，周期应设置为4秒，以此类推。
5. 如果朋友清除程序计时器过期，则应取消朋友清除重复计时器并完成该程序。

此过程的示例如图3.22所示。

一旦建立了友元关系, 友元节点应与友元消息中定义的低功耗节点通信（见第3.6.6.3.2节），并可按照友元管理（Friend Management）中的定义进行管理（见第3.6.6.3.3节）。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_22%20Friend%20Clear%20procedure%20example.png)
*图3.22 Friend Clear规程示例*

##### 3.6.6.3.2 Friend messaging
当友元节点从”友元低功耗节点“接收到“Friend Poll”消息时，其FSN字段值与上一个从低功耗节点接收到的“Friend Poll”消息的FSN字段值相同，友元节点应以其先前发送的完全相同的消息响应，除非该消息已被丢弃。如果先前发送的消息已被丢弃，则应发送友元队列中最旧的条目。

当友元节点从”友元低功耗节点“接“Friend Poll”消息时，低功耗节点的FSN字段值与来自低功耗节点的最后一个Friend Poll消息不同，然后它将从友元队列发送最旧的消息。

向低功耗节点发送存储的消息时，消息应保持不变。如果友元节点上的IV索引已经改变，例如，该节点现在正在使用新的IV索引发送，则发送到低功率节点的消息仍应在友元节点为这些消息接收的IV索引的上下文中发送。

注：上述要求意味着低功耗节点应至少每96小时收集一次所有存储的消息，否则友元节点可能会在低功耗节点接收到存储的消息之前丢弃这些消息。

从接收到低功耗节点发出的"Friend Poll"消息后，每条消息应在（ReceiveDelay ~ （ReceiveDelay+ReceiveWindow））毫秒数这个间隔发送。

如果在”PollTimeout“计时器过期之前，友元节点未收到"Friend Poll"、“Friend Subscription List Add"或"Friend Subscription List Remove"的消息，则友元关系终止，友元节点将丢弃友元队列中的所有条目。

从”Friend Subscription List Add“或”Friend Subscription List Remove“消息后，”Friend Subscription List Confirm“消息应在(ReceiveDelay~(ReceiveDelay+ReceiveWindow)毫秒数这个间隔发送消息。

当友元节点收到"Friend Clear"消息息，其中LPNAddress字段是友元低功耗节点，并且LPNCounter字段在（第3.6.5.6节）中定义的范围内时，友元节点将终止友元关系，友元节点应丢弃友元队列中的所有条目。

##### 3.6.6.3.3 Friend management
如果好友节点从低功耗节点接收到"Friend Subscription List Add"消息（参见第3.6.5.7节），则应将消息中包含的地址或地址列表添加到友元订阅列表中，并使用"Friend Subscription List Confirm"消息（参见第3.6.5.9节）响应，设置TransactionNumber字段的值添加与接收的好友订阅列表中的值相同的消息。

如果友元节点从低功耗节点接收到"Friend Subscription List Remove"消息（参见第3.6.5.8节），它应从友元订阅列表中删除消息中包含的地址或地址列表，并用Friend Subscription List Confirm"消息进行响应，将TransactionNumber字段的值设置为与接收的友元订阅列表中的删除消息中的值相同。
在接收到来自低功耗节点的"Friend Poll"消息后，友元节点应在 （ReceiveDelay~（ReceiveDelay+ReceiveWindow））时间间隔内，使用“Friend Update”消息进行响应。

#### 3.6.6.4 低功耗特性
支持低功耗特性的节点应支持三种强制操作：

* 低功耗建立 （Low Power establishment）
* 低功耗消息传递 （Low Power messaging）
* 低功耗管理 （Low Power management）

低功耗节点发出的所有传输控制消息, 应作为未分段控制消息发送，SRC字段设置为支持低功耗特性的节点的主要元素的单播地址。
##### 3.6.6.4.1 Low Power establishment
低功耗建立操作用于在支持低功耗特性的节点和支持友元特性的节点之间建立友元关系。此操作通过发送友元请求（Friend Request）消息启动。

发送友元请求（Friend Request）消息时，TTL字段设置为0，DST字段设置为所有友元（all-friends）地址。节点应保持低功耗节点计数器（LPNCounter），该计数器是初始化为0值。此值应在友元请求消息中发送，如果由于友元请求消息而建立了友谊，则此值将用于派生友谊安全材料。每次发送友元请求消息后，此值应增加1。LPNCounter可以包装。

从友元请求（Friend Request）经过100毫秒后，该节点应侦听潜在友元节点发送的友元邀请（Friend Offer）消息，侦听最多1秒，并且它可以选择其中一个友元节点来建立友元关系。低功耗节点可以接受接收到的友元邀请，或者继续侦听其他友元邀请消息以进行比较。

如果没有收到可接受的友元邀请（Friend Offer）消息，则节点可以发送新的友元请求（Friend Request）消息。两条连续的友元请求消息之间的时间间隔应大于1.1秒。

若要与已发送好友邀请（Friend Offer）消息的潜在好友建立友元关系，则该节点应将友元序号设置为零，并在收到好友邀请消息后的1秒内，向所选好友节点发送友元轮询（Friend Poll）消息。如果收到朋友更新消息作为响应，则建立友谊，支持低功耗特性的节点正在使用,并且支持友元特性的节点正在使用。

如果节点在多次尝试（例如，6次尝试）后未收到友元更新（Friend Upate）消息，则应重新启动低功耗的建立操作。低功耗建立操作的多次失败，可能表示低功耗节点不再具有有效的IV索引，它应启动IV索引恢复程序（见第3.10.6节）。一旦建立了友元关系，低功耗节点应与低功耗消息（见第3.6.6.4.2节）中定义的友元节点通信，并可管理低功耗管理（见第3.6.6.4.3节）中定义的友谊。

##### 3.6.6.4.2 Low Power messaging
低功耗消息传递操作由低功耗节点执行，以从友元节点接收存储的消息和安全更新。

该操作包括从低功耗节点到友元节点的异步请求和从友元节点到低功耗节点的定时响应。

在轮询超时计时器到期之前，与友元节点为好友的低功耗节点应该向友元节点发送友元轮询（（Friend Poll））消息。

在Friend Poll消息中，TTL字段应设置为0。

作为一般规则，低功耗节点应继续发送Friend Poll消息，直到它接收到MD字段设置为0的Friend Update消息。如图3.23所示。

低功耗节点可以通过发送友元清除(Friend Clear)消息来终止与好友的友谊。Friend Clear消息的TTL字段应为0。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_23%20Friend%20Update%20with%20security%20updates.png)
*图3.23 完全更新的友元更新消息*
FSN字段应设置为友元序列号的值。如果低功耗节点从友元节点接收到响应，并且响应不是最后一次接收到下层传输PDU（ Lower Transport PDU）时，应切换友元序列号。

如果低功耗节点在ReceiveWindow中未收到响应，则应重新发送Friend Poll消息。建议重新发送此消息3次，以确保可靠性和功耗之间的良好平衡。

如果在PollTimeout计时器过期之前，未收到对多个友元轮询（Friend Poll）消息的响应，则友元关系将终止。低功耗节点可以重复低功耗建立操作。如果低功耗节点接收到友元更新（Friend Upate）消息，则应使用与在"Secure Network beacon"中接收到的相同规则处理标志和IV索引字段（见第3.9.3.1、3.10.4和3.10.5节）.

##### 3.6.6.4.3 Low Power management
低功耗管理操作用于管理友元节点中的订阅列表。

低功耗节点可以向友元节点发送一个或多个"Friend Subscription List Add"消息，其中包含低功耗节点所订阅的组地址或虚拟地址的列表。当低功耗节点的订阅更改时，它可以随时发送此类消息。

低功耗节点可以向友元节点发送一个或多个"Friend Subscription List Remove"消息，其中包含组地址列表或不再订阅低功耗节点的虚拟地址列表。当低功耗节点的订阅更改时，它可以随时发送此类消息。

低功耗节点应以设置为TransactionNumber字段值为0x00开始。

应为每个新的"Friend Subscription List Add"消息或"Friend Subscription List Remove"消息增加TransactionNumber字段值，使TransactionNumber与"Friend Subscription List Confirm"消息的TransactionNumber字段匹配。

#### 3.6.6.5 分段和重组实例
第3.5.3.3节和第3.5.3.4节中定义的分段和重组行为,也适用于发送到低功耗节点和从低功耗节点发送的分段消息。

唯一的区别是，由于低功耗节点依赖于友元队列来处理所有传入消息，包括段和段确认，因此友元节点将确认低功耗节点的分段事务。

本节提供了在低功耗节点上进行分割和重新组装的两个示例。

##### 3.6.6.5.1 传入分段消息
图3.24中的消息序列图（MSC）是指向低功耗节点的分段消息的示例。友元节点单独执行重组并发送所需的确认，直到它接收到所有段，此时友元节点将这些段放置在友元队列中，以便将它们传递到低功耗节点。来自另一个源的未分段消息在事务处理过程中接收并独立处理。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_24%20Example%20of%20incoming%20segmented%20message%20directed%20to%20a%20Low%20Power%20node.png)
*图3.24 指向低功耗节点的传入分段消息示例*

##### 3.6.6.5.2 输出分段消息
图3.25所示的消息序列图(MSC)显示了低功耗节点发送的分段消息的示例。由于低功耗节点依赖于友元队列来接收所有传入消息，因此它还需要轮询确认。来自另一个源的未分段消息在事务处理过程中接收并独立处理。
![](https://github.com/fridyss/ble-mesh-note/blob/master/material/3_25%20Example%20of%20outgoing%20segmented%20message%20sent%20by%20a%20Low%20Power%20node.png)
*图3.25 低功耗节点发送的传出分段消息示例*
### 3.6.7 心跳
#### 3.6.7.1 功能概述
#### 3.6.7.2 发布心跳消息
#### 3.6.7.3 接收心跳消息
## 3.7 访问层
### 3.7.1 字节顺序
### 3.7.2 模型标识符
### 3.7.3 访问负载
#### 3.7.3.1 操作码
#### 3.7.3.2 应用参数
### 3.7.4 访问层行为
#### 3.7.4.1 发送一个访问消息
#### 3.7.4.2 收接一个访问消息
#### 3.7.4.3 安全注意事项
#### 3.7.4.4 消息错误程序
### 3.7.5 非应答和应答消息
#### 3.7.5.1 非应答消息
#### 3.7.5.2 应答消息
### 3.7.6 订阅和发布
#### 3.7.6.1 发布
##### 3.7.6.1.1 状态迁移
##### 3.7.6.1.2 状态更改发布
##### 3.7.6.1.3 定期发布
##### 3.7.6.1.4 发布重发
#### 3.7.6.2.5 订阅
### 3.7.7 消息序列图实例
#### 3.7.7.1 应答获取
#### 3.7.7.2 应答设置
#### 3.7.7.3 非应答设置
#### 3.7.7.4 周期性发布应答
## 3.8 Mesh安全
### 3.8.1 字节顺利
### 3.8.2 安全工具箱
#### 3.8.2.1 加密函数
#### 3.8.2.2 CMAC 函数
#### 3.8.2.3 CCM 函数
#### 3.8.2.4 s1 SAL 生成函数
#### 3.8.2.5 k1 推导函数
#### 3.8.2.6 k2 网络密钥数材推导函数
#### 3.8.2.7 k3 推导函数
#### 3.8.2.8 k4 推导函数
### 3.8.3 序列号
### 3.8.4 IV Index
### 3.8.5 Nonce
#### 3.8.5.1 网络 Nonce
#### 3.8.5.2 应用 Nonce
#### 3.8.5.3 设备 Nonce
#### 3.8.5.4 代理 Nonce
### 3.8.6 密钥
#### 3.8.6.1 设备密钥
#### 3.8.6.2 应用密钥
#### 3.8.6.3 网络密钥
##### 3.8.6.3.1 NID,Encryption Key, and Privacy Key
##### 3.8.6.3.2 Network ID
##### 3.8.6.3.3 IdentityKey
##### 3.8.6.3.4 BeaconKey
#### 3.8.6.4 全局密钥索引
### 3.8.7 消息安全
#### 3.8.7.1 上层传输认证和加密
#### 3.8.7.2 网络层认证和加密
#### 3.8.7.3 网络层模糊
### 3.8.8 消息重传保护
## 3.9 Mesh Beacon
### 3.9.1 字节顺序
### 3.9.2 未配置设备Beacon
### 3.9.3 安全网络Beacon
#### 3.9.3.1 安全网络Beacon行为
## 3.10 Mesh 网络管理
### 3.10.1 Mesh网络创建规程
### 3.10.2 临时访客
### 3.10.3 设备UUID
### 3.10.4 密钥刷新规程
#### 3.10.4.1 阶段1-新密钥的分配
#### 3.10.4.2 阶段2-切换密钥
#### 3.10.4.3 阶段3- 废除旧密钥
### 3.10.5 IV Update 规程
### 3.10.5.1 IV Update 测试模式
### 3.10.6 IV Index 恢复规程
### 3.10.7 节点消除规程
## 3.11 消息处理流