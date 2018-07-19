# 方案讨论

## 用户需求
安全，高效

## 结论

Cisco建议：
1. 模块化
2. 松耦合
3. 多云的优点：去中心化，业务在云之间平滑迁移，多云的缺点：复杂，碎片化，不好控制
4. 多云如何与基础架构之间的衔接，不推荐紧耦合，推荐1：独立编排的松耦合方式，优点：简单，底层管理优化，可提供Underlay流量工程；缺点：xxx；
5. Cisco推荐由ACI管理Underlay和Overlay，云平台使用类似macvlan/ipvlan之类的技术，优点：性能好，松耦合，高SDN虚拟化/容器化可视化，疑问：纯开源插件支持提供者
6. Cisco ACI EPG EndPoint Group
7. Cisco **网络为中心的策略模型**，传统应用可按网络分区按照安全域划分，类似vlan组概念
8. Cisco 应用为中心的策略模型，新型应用可按应用模块，服务力度划分
9. Cisco 的 ACI 网络层级 Tenant -> VRF -> Bridge Domain -> EPG/Subnets Container, EPG不能跨Bridge Domain，在一个Bridge Domain里可以有多个EPG，在1个Bridge里的EPG可以跨多个Subnet，EPG与EPG间默认不通，EPG与EPG间的联通需要引入新的策略contract；跨Tenant的contract的设计
10. Application Profile是EPG的集合
11. Application Profile包含的EPG决定应用的健康分值

VMware与ACI实现原理
- [ ] VMware vSphere 5.5+
- [ ] Cisco Leaf + VMware Distribute vSwitch
- [ ] 3rd Party Leaf + Cisco AVS Edge 

物理层实现原理
- [ ] Leaf物理交换机是Cisco
- [ ] 设置以ACI作为网关的EP
- [ ] 设置以防火墙作为网关的EP
- [ ] 设置以VRF为边界，以防火墙子端口为网关的EP
- [ ] 负载均衡的流量引导方式为Full NAT
- [ ] 对LVS DR的支持
- [ ] 流量镜像
- [ ] 纳管，导流
- [ ] 防火墙放在VRF间，负载均衡用Full NAT，IDS/SPAN


等级保护的要求
- [ ] WAF，IPS
- [ ] SDN相当于防火墙物理旁路逻辑串接的要求
- [ ] 在1个Fabric下防火墙旁路相当于安全资源池概念
- [ ] 灾备，两地三中心，不建议跨地域防火墙集群
- [ ] 全网漏扫与SDN的关系，到每个EPG内漏扫
- [ ] SDN北向RESTAPI接口


