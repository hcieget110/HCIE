# 面试与复习提纲

- **Underlay 和 Overlay 各解决什么？** OSPF 让 VTEP 地址可达；BGP EVPN 分发业务 MAC/IP 与隧道成员信息；VXLAN 数据包由 Underlay 传输。
- **为什么有两个 Loopback？** 本实验分开 BGP 身份和 NVE 封装端点，便于定位与扩展；两者都必须被 Underlay 学到。
- **VLAN 与 VNI？** VLAN 为本地二层接入标识；L2VNI 标识覆盖网络的二层广播域。不同交换机需要一致业务映射。
- **RD 与 RT？** RD 使路由唯一；RT 决定导入/导出策略。不能仅因 RD 相同或不同判断业务能否互通。
- **Type-2、Type-3、Type-5？** 分别关注 MAC/IP 主机、IMET/BUM 成员、IP前缀。此实验不发布 Type-5。
- **BUM 如何复制？** 使用 BGP ingress replication，源VTEP向远端成员逐一复制；规模增大时复制成本增加。
- **Anycast Gateway 为什么同时需要相同IP与MAC？** 主机可在任意接入VTEP使用同一默认网关，保持一致的网关标识。
- **L3VNI 的作用？** 对称 IRB 中承载 VRF 内的三层转发；本实验50000连接TENANT-A，两端需具备正确VRF和RT。
- **为什么不用 RR？** 三台可直接全互联，邻居少且容易理解。扩展到大量Leaf时可引入Spine/RR；RR通常无需充当业务VTEP。
- **怎样证明实验成功？** 从邻居、路由、主机学习到同子网与跨子网流量逐层举证，补充MTU及链路故障恢复测试。

练习：画出 H111-100 到 H222-200 的入站查网关、VRF查表、VXLAN封装、远端解封装和主机交付流程，并解释每一步需要哪条控制平面信息。
