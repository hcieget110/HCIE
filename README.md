# HCIE · VXLAN / EVPN 实验与复习

面向 PNETLab / NX-OSv 10.4(3)F 的 111、222、333 三节点实验资料。首版覆盖 OSPF Underlay、iBGP EVPN、VTEP、VLAN/VNI、分布式 Anycast Gateway 与故障定位。

> 本仓库首版为可复用配置骨架。设备编号沿用 111/222/333；当前会话未提供原实验 running-config、接线和输出，因此地址与接口采用本文统一规划，并非原实验配置的逐字归档。尚未在本轮连接 PNETLab 实机复测；不要把预期输出当作已通过记录。

## 目录与阅读顺序
- [拓扑、地址与接线](docs/topology-addressing.md)
- [配置原理与部署步骤](docs/deployment.md)
- [验证与验收记录](docs/verification.md)
- [常见故障排查](docs/troubleshooting.md)
- [面试复习](docs/interview.md)
- [111 配置](configs/nxosv-10.4-3F/111.cfg)
- [222 配置](configs/nxosv-10.4-3F/222.cfg)
- [333 配置](configs/nxosv-10.4-3F/333.cfg)

## 实验范围
三台交换机均为 VTEP，Underlay 为 OSPF area 0；Overlay 为 AS 65000 内 Loopback0 全互联 iBGP EVPN，无 RR。L2VNI 10100/10200，TENANT-A 的 L3VNI 50000；使用 BGP ingress replication，无需 PIM。无 vPC、外部网络或默认路由注入。

推荐先验证 Loopback 可达，再检查 EVPN 邻居，最后接入终端验证同网段和跨网段通信。粘贴前核对 PNETLab 接口映射、镜像版本及 MTU；配置不含管理 IP、账号、密码或镜像文件。

## 扩展约定
每个实验按 docs/ 文档与 configs/<平台版本>/ 配置组织。改动地址、VNI 或端口时同步修改规划、三台配置和验收用例；保存脱敏输出到 evidence/。后续可增加 Spine/RR、vPC、Type-5 与外部路由实验。

## 参考
- [Cisco NX-OS 10.4(x) VXLAN BGP EVPN 配置指南](https://www.cisco.com/c/en/us/td/docs/dcn/nx-os/nexus9000/104x/configuration/vxlan/cisco-nexus-9000-series-nx-os-vxlan-configuration-guide-release-104x/m_configuring_vxlan_bgp_evpn.html)
- [Cisco NX-OS 10.4(x) VXLAN 配置指南](https://www.cisco.com/c/en/us/td/docs/dcn/nx-os/nexus9000/104x/configuration/vxlan/cisco-nexus-9000-series-nx-os-vxlan-configuration-guide-release-104x/m_configuring_vxlan_93x.html)

版本：首版，2026-10-06。官方指南针对 Nexus 平台；虚拟镜像的命令支持与转发能力以实际镜像测试为准。
