

=======================================
Zabbix
=======================================

Zabbix组件
Zabbix Server：负责接收agent发送的报告信息的核心组件，所有配置、统计数据及操作数据均由其组织进行
Database Storage：专用于存储所有配置信息，以及有zabbix收集的数据
Web interface（frontend）：zabbix的GUI接口，通常与server运行在同一台机器上
Proxy：可选组件，常用于分布式监控环境中，代理Server收集部分被监控数据并统一发往Server端
Agent：部署在被监控主机上，负责收集本地数据并发往Server端或者Proxy端
