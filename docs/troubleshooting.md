# 常见故障排查

顺序：端口 → Underlay → BGP EVPN → VNI/RT → 主机与网关 → MTU/虚拟平台。

| 现象 | 优先检查 | 修复方向 |
|---|---|---|
| OSPF 无邻居 | 接线、no switchport、IP/掩码、area、shutdown | 两端路由模式与同一 /30，启用接口 |
| OSPF 卡 EXSTART/EXCHANGE | 两端 MTU、OSPF 网络类型 | 统一 MTU9216 和 point-to-point；不要用忽略 MTU 掩盖问题 |
| BGP Idle/Active | 远端 Lo0 路由、remote-as、update-source | 先修复 OSPF，源为 Lo0，AS65000 |
| BGP 正常但无 EVPN 路由 | 地址族、send-community extended、主机学习 | 开启 l2vpn evpn，发送扩展团体，触发主机 ARP |
| 有 EVPN 路由但不导入 | RD/RT、VNI、VRF | 同业务 RT 一致，RD 独立；核对 L2 与 L3 RT |
| NVE Down | feature、Lo1 状态、source-interface、VNI | 启用 NVE，确保 Lo1 存在且有远端路由 |
| NVE peer 少于预期 | Type-3、L2VNI 状态、Lo1 可达 | 核对 ingress replication、EVPN导入和活跃业务VLAN |
| 同 VLAN 不通 | access VLAN、MAC学习、VN-segment | 核对两端相同 VLAN/VNI 和终端掩码 |
| 网关不通或漂移 | SVI状态、VRF、Anycast IP/MAC | 三台网关IP/MAC一致，主机IP唯一；确认有活动接入端口 |
| 同网段通而跨网段不通 | L3VNI、associate-vrf、SVI3000、VRF RT | 核对 VNI50000、ip forward、VRF归属及远端主机ARP |
| 小包通大包不通 | VXLAN封装开销、每跳MTU | 检查路由口、PNETLab虚拟交换和宿主机路径 |
| CLI 命令拒绝或数据面异常 | show version、资源、虚拟网卡 | 对照镜像实际支持，保留报错；重启前先保存诊断 |

## 建议采集
show version、show interface brief、show ip ospf neighbors、show ip route、show bgp l2vpn evpn summary、show bgp l2vpn evpn、show nve peers、show nve vni、show mac address-table、show ip arp vrf TENANT-A 和相关 running-config。

不要仅依据 BGP Established 或 NVE Up 宣布业务通过。避免第一步就 clear bgp、重启或清空配置；先定位失败层次，每次只改变一个因素并复测。对外提交前删除账号、密码、管理地址及与实验无关的标识。
