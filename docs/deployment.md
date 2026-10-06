# 配置原理与部署

## 1. PNETLab 准备
使用合法获取的 NX-OSv 10.4(3)F 镜像；每台的 CPU、内存与虚拟网卡按镜像要求分配，不在仓库分发镜像。启动后执行 show version，等待接口就绪，按照拓扑接线。建议使用干净实验实例，保留控制台；完整配置是合并配置，不会自动清理旧 BGP/VRF/VNI。

三台分别打开对应 .cfg，进入 configure terminal 后粘贴。配置中无 configure terminal 或保存命令；验收通过后执行 copy running-config startup-config。

## 2. Underlay OSPF
feature ospf 启用功能；两条路由端口使用 /30、point-to-point、area 0 与相同 MTU。Loopback0 提供稳定的 BGP 会话源，Loopback1 提供 VTEP 地址。OSPF 只承担传输网络和 Loopback 可达性，不发布 TENANT-A 业务网段。上线 Overlay 前，必须从本机 Loopback0/1 到远端对应 Loopback 验证可达。

## 3. iBGP EVPN
feature bgp、nv overlay evpn 启用 EVPN 控制平面。三台 AS 65000，通过 Loopback0 建立全互联 iBGP，并在邻居 l2vpn evpn 地址族发送 standard 和 extended community。没有 RR：每台与另外两台直接建邻，避免 iBGP 不转发从其他 iBGP 邻居学到的路由所产生的遗漏。BGP Established 仅说明会话正常，仍需确认 EVPN 路由和 RT 导入。

## 4. VTEP 与 VLAN/VNI
feature nv overlay 与 vn-segment-vlan-based 启用 VXLAN 和映射。NVE1 使用 Loopback1，host-reachability protocol bgp；两个 L2VNI 使用 ingress-replication protocol bgp，动态建立 BUM 复制关系。VLAN100/200 分别映射 10100/10200，EVPN 节点配置 L2VNI 的 RD 与显式 RT。全节点使用一致业务映射。

## 5. Anycast Gateway 与 L3VNI
SVI100/200 放入 TENANT-A，并配置相同网关 IP 及 fabric forwarding mode anycast-gateway。先配置 vrf member，再配置 IP，以免接口迁移 VRF 时地址被清除。全局网关 MAC 必须一致。

传统 L3VNI 模式：TENANT-A 绑定 VNI50000，VLAN3000 映射 VNI50000；SVI3000 绑定 TENANT-A，配置 ip forward 和无 IP 的转发接口；NVE 上 member vni 50000 associate-vrf。两端的 L3VNI、VRF EVPN RT 必须匹配，支持对称 IRB。

基础实验依赖主机 MAC/IP 的 Type-2 与 BUM 的 Type-3 路由，不要求 Type-5 前缀发布。未配置 redistribute direct 或 advertise l2vpn evpn，不能把无 Type-5 视为失败。跨网段测试前让所有目标主机发送 ARP/流量，形成主机学习记录。

## 6. 接入和保存
Eth1/3 为 access VLAN100，Eth1/4 为 access VLAN200；无 trunk、vPC。PNETLab 主机配置静态 IP、掩码和网关后按 verification.md 验收。保存前确认没有误改管理面配置。若命令被拒绝，记录 show version 和准确报错，对照相应镜像支持；不要以忽略报错的方式继续粘贴。
