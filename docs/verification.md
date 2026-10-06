# 验证命令与验收

以下为预期结果和检查步骤，不是本轮真实设备输出。保存脱敏输出到 evidence/，记录镜像、时间、设备及测试端点。

## 分层检查
| 层次 | 命令 | 预期 |
|---|---|---|
| 版本/端口 | show version / show interface brief | 10.4(3)F，链路映射正确 |
| 功能 | show feature | ospf、bgp、nv overlay、SVI 等已开启 |
| Underlay | show ip ospf neighbors | 每节点两个邻居 FULL |
| 路由 | show ip route 10.255.1.33 | 远端 VTEP /32 可达（在111/222执行） |
| BGP | show bgp l2vpn evpn summary | 每节点两个 Established 邻居 |
| EVPN | show bgp l2vpn evpn | 主机学习后 Type-2；L2VNI 的 Type-3 |
| 隧道 | show nve interface nve1 / show nve peers | NVE1 Up，远端 VTEP peer Up |
| VNI | show nve vni | 10100/10200 L2 与 50000 L3 正常 |
| 映射 | show vlan brief / show vrf | VLAN 与 TENANT-A 存在 |
| 主机学习 | show mac address-table dynamic / show ip arp vrf TENANT-A | 本地/远端 MAC 与 ARP 合理 |
| VRF 转发 | show ip route vrf TENANT-A | 本地网段及学习到的主机路由 |
| L2 路由细节 | show l2route evpn mac all / show l2route evpn mac-ip all | 本地或远端主机来源正确 |

具体显示格式及命令可用性以虚拟镜像 CLI 为准，遇到语法差异使用上下文 ? 确认。

## 111 上的连通性探测
```text
ping 10.255.0.222 source 10.255.0.111
ping 10.255.0.33 source 10.255.0.111
ping 10.255.1.222 source 10.255.1.111
ping 10.255.1.33 source 10.255.1.111
show running-config interface nve1
show running-config bgp
```
另外两台按地址规划替换源/目的。大报文通过 ping 的 packet-size 与 df-bit 选项检查，先用 CLI ? 确认语法；仅小包成功不能证明 MTU 正确。

## 终端验收矩阵
1. 所有主机 ping 自己的 .1 网关，确认接入、SVI 与 ARP。
2. 三个 VLAN100 主机互 ping；三个 VLAN200 主机互 ping，确认跨 VTEP 同网段转发。
3. H111-100 → H222-200、H222-100 → H333-200、H333-100 → H111-200，并反向测试，确认跨 VTEP 跨子网 IRB。
4. H111-100 → H111-200，确认本地跨子网路由。
5. 保持终端流量，关闭 111–222 的一个 Underlay 接口；OSPF 收敛后经333恢复通信。记录收敛和丢包，随后 no shutdown 恢复原链路。
6. 全链路恢复后再次检查邻居和业务，然后保存配置。

## 当前验收记录
| 项目 | 状态 |
|---|---|
| 配置与文档静态一致性 | 提交前检查 |
| CLI 在目标镜像接受 | 待实机复测 |
| OSPF / BGP / NVE 状态 | 待实机复测 |
| 同网段、跨网段、MTU 与故障收敛 | 待实机复测 |

不要在未采集输出时填写“通过”。记录中同时保留失败现象、修复措施与复测结果。
