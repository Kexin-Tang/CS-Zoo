# OSI七层模型
<img src="https://static.runoob.com/images/mix/v2-854e3df8ea850c977c30cb1deb1f64db_r.jpg" alt="OSI七层模型">

| 名称 | 数据格式 | 作用 | 常见协议/设备 |
| :----: | :----: | :----: | :----: |
|  应用层 | N/A | 控制应用程序 | ssh HTTP ftp DNS|
|  表示层 | N/A | 格式化数据，比如提供加密| ASCII JPEG MP4 AVI PNG|
|  会话层 | N/A | 控制会话，建立管理终止应用程序会话 | SQL AHP PHP JSP|
|  传输层 | fragment(段) | 提供可靠和尽力而为的传输,维护端到端通信 | TCP UDP|
|  网络层 | package(包) | 路由选择与IP寻址 | IP 路由器|
|数据链路层 | frame(帧) | 定义如何格式化数据(帧定界)，支持错误检测 | 交换机|
|  物理层 | bit(比特流) | 定义物理传输媒介的信息 | HUB集线器|
