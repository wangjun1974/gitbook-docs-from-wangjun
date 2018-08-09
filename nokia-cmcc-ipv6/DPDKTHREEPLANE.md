# CBIS Dpdk 部署

## 适配 Intel 网卡 OVS-DPDK 的 overcloud-full image 

### 订制镜像

```
[root@localhost dpdk]# virt-customize -a overcloud-full.qcow2 --copy-in openvswitch-2.7.2-1.git20170719.el7fdp.x86_64.rpm:/root 
[   0.0] Examining the guest ...
[   1.9] Setting a random seed
[   1.9] Copying: openvswitch-2.7.2-1.git20170719.el7fdp.x86_64.rpm to /root
[   2.0] Finishing off

[root@localhost dpdk]# virt-customize -a overcloud-full.qcow2 --run-command 'rpm -e --nodeps openvswitch' 
[   0.0] Examining the guest ...
[   1.9] Setting a random seed
[   2.0] Running: rpm -e --nodeps openvswitch
[   3.0] Finishing off

[root@localhost dpdk]# virt-customize -a overcloud-full.qcow2 --run-command 'yum localinstall -y  /root/openvswitch-2.7.2-1.git20170719.el7fdp.x86_64.rpm' 
[   0.0] Examining the guest ...
[   2.0] Setting a random seed
[   2.0] Running: yum localinstall -y  /root/openvswitch-2.7.2-1.git20170719.el7fdp.x86_64.rpm
[   8.3] Finishing off

[root@localhost dpdk]# virt-customize -a overcloud-full.qcow2 --run-command 'rm -f /root/openvswitch-2.7.2-1.git20170719.el7fdp.x86_64.rpm' --selinux-relabel
[   0.0] Examining the guest ...
[   1.9] Setting a random seed
[   1.9] Running: rm -f /root/openvswitch-2.7.2-1.git20170719.el7fdp.x86_64.rpm
[   2.0] SELinux relabelling
[  53.9] Finishing off
```


### 备份 undercloud /home/stack/images 下的 overcloud-full.qcow2

```
cd /home/stack/images
cp overcloud-full.qcow2 overcloud-full.qcow2.orig
```


### 上传镜像到 undercloud /home/stack/images 目录
```
[root@localhost dpdk]# scp overcloud-full.qcow2 stack@192.167.40.3:~/images                                                                     
stack@192.167.40.3's password: 
overcloud-full.qcow2                                                                                            100% 2149MB 144.0MB/s   00:14 
```


### 更新镜像
```
source /home/stack/stackrc
[stack@undercloud images]$ openstack overcloud image upload --update-existing --image-path ~/images/
Image "overcloud-full-vmlinuz" is up-to-date, skipping.
Image "overcloud-full-initrd" is up-to-date, skipping.
Image "overcloud-full" was uploaded.
+--------------------------------------+----------------+-------------+------------+--------+
|                  ID                  |      Name      | Disk Format |    Size    | Status |
+--------------------------------------+----------------+-------------+------------+--------+
| a82d71d9-7093-4f6d-91c7-1a47ecbe9db3 | overcloud-full |    qcow2    | 2253062144 | active |
+--------------------------------------+----------------+-------------+------------+--------+
Image "bm-deploy-kernel" is up-to-date, skipping.
Image "bm-deploy-ramdisk" is up-to-date, skipping.
Image file "/httpboot/agent.kernel" is up-to-date, skipping.
Image file "/httpboot/agent.ramdisk" is up-to-date, skipping.
```


## 双平面 DPDK 模版改造

### 需修改的文件
|文件名|
|:----:|
|templates/network/role_networks/DpdkPerformanceCompute.yaml|
|templates/firstboot/cbis/dpdk/pre_reboot.yaml|

文件 ~/templates/network/role_networks/DpdkPerformanceCompute.yaml 
```
--- DpdkPerformanceCompute.yaml.orig    2018-08-06 05:22:16.212150628 +0800
+++ DpdkPerformanceCompute.yaml 2018-08-07 06:09:14.516150628 +0800
@@ -166,7 +166,9 @@
               mtu: 9000
               name: tenant-bond
               ovs_extra:
+              - set interface dpdk0 type=dpdk options:dpdk-devargs=0000:08:00.0 
               - set interface dpdk0 mtu_request=$MTU
+              - set interface dpdk1 type=dpdk options:dpdk-devargs=0000:08:00.1 
               - set interface dpdk1 mtu_request=$MTU
               ovs_options: bond_mode=active-backup
               type: ovs_dpdk_bond

```

文件 templates/firstboot/cbis/dpdk/pre_reboot.yaml
```
--- /home/stack/templates/firstboot/cbis/dpdk/pre_reboot.yaml.orig      2018-08-06 09:08:45.826150628 +0800
+++ /home/stack/templates/firstboot/cbis/dpdk/pre_reboot.yaml   2018-08-06 09:54:26.181150628 +0800
@@ -119,7 +119,13 @@
             if [ $? -ne 0 ]; then
                 dpdk_reserved_memory=$socket_mem
             fi
+            log "Restart ovsdb service"
+            execute "systemctl restart ovsdb-server"
             execute "/bin/bash $cbis_artifacts_path/utils/preconfig_dpdk.sh --logfile $logfile --socket_mem $dpdk_reserved_memory"
```


## 三平面 DPDK 模版改造

### 需修改的文件
|文件名|
|:----:|
|templates/network/role_networks/Controller.yaml|
|templates/network/role_networks/DpdkPerformanceCompute.yaml|
|templates/firstboot/cbis/dpdk/pre_reboot.yaml（同上）|

文件templates/network/role_networks/Controller.yaml
```
description: 'Software Config to drive os-net-config with 2 bonded nics on a bridge
  with VLANs attached for the controller role.

  '
heat_template_version: 2015-04-30
outputs:
  OS::stack_id:
    description: The OsNetConfigImpl resource.
    value:
      get_resource: OsNetConfigImpl
parameters:
  AuxIpSubnet:
    default: ''
    description: IP address/subnet on the new vlan network
    type: string
  BondInterfaceOvsOptions:
    constraints:
    - allowed_pattern: ^((?!balance.tcp).)*$
      description: 'The balance-tcp bond mode is known to cause packet loss and

        should not be used in BondInterfaceOvsOptions.

        '
    default: bond_mode=active-backup
    description: The ovs_options string for the bond interface. Set things like lacp=active
      and/or bond_mode=balance-slb using this option.
    type: string
  ControlPlaneDefaultRoute:
    description: The default route of the control plane network.
    type: string
  ControlPlaneIp:
    default: ''
    description: IP address/subnet on the ctlplane network
    type: string
  ControlPlaneSubnetCidr:
    default: '24'
    description: The subnet CIDR of the control plane network.
    type: string
  DnsServers:
    default: []
    description: A list of DNS servers (2 max for some implementations) that will
      be added to resolv.conf.
    type: comma_delimited_list
  EC2MetadataIp:
    description: The IP address of the EC2 metadata server.
    type: string
  ExternalInterfaceDefaultRoute:
    default: 10.0.0.1
    description: default route for the external network
    type: string
  ExternalIpSubnet:
    default: ''
    description: IP address/subnet on the external network
    type: string
  ExternalNetworkVlanID:
    default: 10
    description: Vlan ID for the external network traffic.
    type: number
  InternalApiIpSubnet:
    default: ''
    description: IP address/subnet on the internal API network
    type: string
  InternalApiNetworkVlanID:
    default: 20
    description: Vlan ID for the internal_api network traffic.
    type: number
  ManagementInterfaceDefaultRoute:
    default: unset
    description: The default route of the management network.
    type: string
  ManagementIpSubnet:
    default: ''
    description: IP address/subnet on the management network
    type: string
  ManagementNetworkVlanID:
    default: 60
    description: Vlan ID for the management network traffic.
    type: number
  StorageIpSubnet:
    default: ''
    description: IP address/subnet on the storage network
    type: string
  StorageMgmtIpSubnet:
    default: ''
    description: IP address/subnet on the storage mgmt network
    type: string
  StorageMgmtNetworkVlanID:
    default: 40
    description: Vlan ID for the storage mgmt network traffic.
    type: number
  StorageNetworkVlanID:
    default: 30
    description: Vlan ID for the storage network traffic.
    type: number
  TenantIpSubnet:
    default: ''
    description: IP address/subnet on the tenant network
    type: string
  TenantNetworkVlanID:
    default: 50
    description: Vlan ID for the tenant network traffic.
    type: number
resources:
  OsNetConfigImpl:
    properties:
      config:
        os_net_config:
          network_config:
          - addresses:
            - ip_netmask:
                list_join:
                - '/'
                - - {get_param: ControlPlaneIp}
                  - {get_param: ControlPlaneSubnetCidr}
            dns_servers:
            - 192.167.40.2
            members:
            - members:
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: ens2f1
                primary: 'true'
                type: interface
                use_dhcp: false
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: ens2f0
                type: interface
                use_dhcp: false
              mtu: 9000
              name: infra-bond
              ovs_options: bond_mode=active-backup
              type: ovs_bond
            - addresses:
              - ip_netmask: {get_param: InternalApiIpSubnet}
              device: infra-bond
              mtu: 9000
              type: vlan
              use_dhcp: false
              vlan_id: 403
            - addresses:
              - ip_netmask: {get_param: ExternalIpSubnet}
              device: infra-bond
              mtu: 9000
              routes:
              - default: true
                next_hop: 192.167.40.254
              type: vlan
              use_dhcp: false
              vlan_id: 401
            mtu: 9000
            name: br-all
            routes:
            - ip_netmask: 169.254.169.254/32
              next_hop: 172.31.255.1
            type: ovs_bridge
            use_dhcp: false
          - members:
            - members:
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: ens3f1
                primary: 'true'
                type: interface
                use_dhcp: false
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: ens3f0
                type: interface
                use_dhcp: false
              mtu: 9000
              name: tenant-bond
              ovs_options: bond_mode=active-backup
              type: ovs_bond
            - addresses:
              - ip_netmask: {get_param: TenantIpSubnet}
              device: tenant-bond
              mtu: 9000
              type: vlan
              use_dhcp: false
              vlan_id: 406
            mtu: 9000
            name: br-ex
            type: ovs_bridge
            use_dhcp: false
          - members:
            - members:
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: eno49
                primary: 'true'
                type: interface
                use_dhcp: false
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: eno50
                type: interface
                use_dhcp: false
              mtu: 9000
              name: storage-bond
              ovs_options: bond_mode=active-backup
              type: ovs_bond
            - addresses:
              - ip_netmask: {get_param: StorageIpSubnet}
              device: storage-bond
              mtu: 9000
              type: vlan
              use_dhcp: false
              vlan_id: 404
            - addresses:
              - ip_netmask: {get_param: StorageMgmtIpSubnet}
              device: storage-bond
              mtu: 9000
              type: vlan
              use_dhcp: false
              vlan_id: 405
            mtu: 9000
            name: br-storage
            type: ovs_bridge
            use_dhcp: false
          - name: br-phys-3
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet3-patch
            peer: physnet3-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-3
            name: physnet3-br-ex-patch
            peer: br-ex-physnet3-patch
            type: ovs_patch_port
          - name: br-phys-2
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet2-patch
            peer: physnet2-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-2
            name: physnet2-br-ex-patch
            peer: br-ex-physnet2-patch
            type: ovs_patch_port
          - name: br-phys-1
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet1-patch
            peer: physnet1-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-1
            name: physnet1-br-ex-patch
            peer: br-ex-physnet1-patch
            type: ovs_patch_port
          - name: br-phys-0
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet0-patch
            peer: physnet0-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-0
            name: physnet0-br-ex-patch
            peer: br-ex-physnet0-patch
            type: ovs_patch_port
          - name: br-phys-7
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet7-patch
            peer: physnet7-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-7
            name: physnet7-br-ex-patch
            peer: br-ex-physnet7-patch
            type: ovs_patch_port
          - name: br-phys-6
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet6-patch
            peer: physnet6-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-6
            name: physnet6-br-ex-patch
            peer: br-ex-physnet6-patch
            type: ovs_patch_port
          - name: br-phys-5
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet5-patch
            peer: physnet5-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-5
            name: physnet5-br-ex-patch
            peer: br-ex-physnet5-patch
            type: ovs_patch_port
          - name: br-phys-4
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet4-patch
            peer: physnet4-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-4
            name: physnet4-br-ex-patch
            peer: br-ex-physnet4-patch
            type: ovs_patch_port
          - name: br-phys-8
            type: ovs_bridge
          - bridge_name: br-ex
            name: br-ex-physnet8-patch
            peer: physnet8-br-ex-patch
            type: ovs_patch_port
          - bridge_name: br-phys-8
            name: physnet8-br-ex-patch
            peer: br-ex-physnet8-patch
            type: ovs_patch_port
      group: os-apply-config
    type: OS::Heat::StructuredConfig
```

文件templates/network/role_networks/DpdkPerformanceCompute.yaml
```
description: 'Software Config to drive os-net-config with 2 bonded nics on a bridge
  with VLANs attached for the controller role.

  '
heat_template_version: 2015-04-30
outputs:
  OS::stack_id:
    description: The OsNetConfigImpl resource.
    value:
      get_resource: OsNetConfigImpl
parameters:
  AuxIpSubnet:
    default: ''
    description: IP address/subnet on the new vlan network
    type: string
  BondInterfaceOvsOptions:
    constraints:
    - allowed_pattern: ^((?!balance.tcp).)*$
      description: 'The balance-tcp bond mode is known to cause packet loss and

        should not be used in BondInterfaceOvsOptions.

        '
    default: bond_mode=active-backup
    description: The ovs_options string for the bond interface. Set things like lacp=active
      and/or bond_mode=balance-slb using this option.
    type: string
  ControlPlaneDefaultRoute:
    description: The default route of the control plane network.
    type: string
  ControlPlaneIp:
    default: ''
    description: IP address/subnet on the ctlplane network
    type: string
  ControlPlaneSubnetCidr:
    default: '24'
    description: The subnet CIDR of the control plane network.
    type: string
  DnsServers:
    default: []
    description: A list of DNS servers (2 max for some implementations) that will
      be added to resolv.conf.
    type: comma_delimited_list
  EC2MetadataIp:
    description: The IP address of the EC2 metadata server.
    type: string
  ExternalInterfaceDefaultRoute:
    default: 10.0.0.1
    description: default route for the external network
    type: string
  ExternalIpSubnet:
    default: ''
    description: IP address/subnet on the external network
    type: string
  ExternalNetworkVlanID:
    default: 10
    description: Vlan ID for the external network traffic.
    type: number
  InternalApiIpSubnet:
    default: ''
    description: IP address/subnet on the internal API network
    type: string
  InternalApiNetworkVlanID:
    default: 20
    description: Vlan ID for the internal_api network traffic.
    type: number
  ManagementInterfaceDefaultRoute:
    default: unset
    description: The default route of the management network.
    type: string
  ManagementIpSubnet:
    default: ''
    description: IP address/subnet on the management network
    type: string
  ManagementNetworkVlanID:
    default: 60
    description: Vlan ID for the management network traffic.
    type: number
  StorageIpSubnet:
    default: ''
    description: IP address/subnet on the storage network
    type: string
  StorageMgmtIpSubnet:
    default: ''
    description: IP address/subnet on the storage mgmt network
    type: string
  StorageMgmtNetworkVlanID:
    default: 40
    description: Vlan ID for the storage mgmt network traffic.
    type: number
  StorageNetworkVlanID:
    default: 30
    description: Vlan ID for the storage network traffic.
    type: number
  TenantIpSubnet:
    default: ''
    description: IP address/subnet on the tenant network
    type: string
  TenantNetworkVlanID:
    default: 50
    description: Vlan ID for the tenant network traffic.
    type: number
resources:
  OsNetConfigImpl:
    properties:
      config:
        os_net_config:
          network_config:
          - addresses:
            - ip_netmask:
                list_join:
                - '/'
                - - {get_param: ControlPlaneIp}
                  - {get_param: ControlPlaneSubnetCidr}
            dns_servers:
            - 192.167.40.2
            members:
            - bonding_options: mode=active-backup
              members:
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: ens2f1
                primary: 'true'
                type: interface
                use_dhcp: false
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: ens2f0
                type: interface
                use_dhcp: false
              mtu: 9000
              name: infra-bond
              type: linux_bond
            mtu: 9000
            name: br-all
            routes:
            - ip_netmask: 169.254.169.254/32
              next_hop: 172.31.255.1
            type: linux_bridge
            use_dhcp: false
          - members:
            - bonding_options: mode=active-backup
              members:
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: eno49
                primary: 'true'
                type: interface
                use_dhcp: false
              - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G $DEVICE
                  rx 2048 tx 2048
                mtu: 9000
                name: eno50
                type: interface
                use_dhcp: false
              mtu: 9000
              name: storage-bond
              type: linux_bond
            mtu: 9000
            name: br-storage
            type: linux_bridge
            use_dhcp: false
          - addresses:
            - ip_netmask: {get_param: TenantIpSubnet}
            members:
            - members:
              - members:
                - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G
                    $DEVICE rx 2048 tx 2048
                  mtu: 9000
                  name: ens3f1
                  primary: 'true'
                  type: interface
                  use_dhcp: false
                name: dpdk0
                type: ovs_dpdk_port
              - members:
                - ethtool_opts: --config-ntuple $DEVICE rx-flow-hash udp4 sdfn; -G
                    $DEVICE rx 2048 tx 2048
                  mtu: 9000
                  name: ens3f0
                  type: interface
                  use_dhcp: false
                name: dpdk1
                type: ovs_dpdk_port
              mtu: 9000
              name: tenant-bond
              ovs_extra:
              - set interface dpdk0 type=dpdk options:dpdk-devargs=0000:08:00.0 
              - set interface dpdk0 mtu_request=$MTU
              - set interface dpdk1 type=dpdk options:dpdk-devargs=0000:08:00.1 
              - set interface dpdk1 mtu_request=$MTU
              ovs_options: bond_mode=active-backup
              type: ovs_dpdk_bond
            name: br-ex
            ovs_extra:
            - str_replace:
                params:
                  _VLAN_TAG_: {get_param: TenantNetworkVlanID}
                template: set port $DEVICE tag=_VLAN_TAG_
            type: ovs_user_bridge
            use_dhcp: false
          - addresses:
            - ip_netmask: {get_param: InternalApiIpSubnet}
            device: infra-bond
            mtu: 9000
            type: vlan
            use_dhcp: false
            vlan_id: 403
          - addresses:
            - ip_netmask: {get_param: StorageIpSubnet}
            device: storage-bond
            mtu: 9000
            type: vlan
            use_dhcp: false
            vlan_id: 404
          - addresses:
            - ip_netmask: {get_param: StorageMgmtIpSubnet}
            device: storage-bond
            mtu: 9000
            type: vlan
            use_dhcp: false
            vlan_id: 405
      group: os-apply-config
    type: OS::Heat::StructuredConfig
```


### 模版

#### DPDK 双平面
20180809 链接:https://pan.baidu.com/s/1v-7wYf5vaMHm7FXQ83kmWQ  密码:mhab
20180808 链接:https://pan.baidu.com/s/1NDGM-PvxR-jkBvdqZvobAQ  密码:9mv0

#### DPDK 三平面
20180809 链接:https://pan.baidu.com/s/1uccWFhlLu9QojJ0UhWvisQ  密码:v2yk


### ChangeLog
* 20180809 DPDK 三平面成功部署
* 20180808 DPDK 双平面成功部署

