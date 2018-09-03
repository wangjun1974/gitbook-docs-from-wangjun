## CBIS 18.5 为计算节点添加KVM Realtime功能

### 相关文件
[下载链接](https://pan.baidu.com/s/1y63huaVbHF_y4j2Bv0vEpA)
下载密码:m9yq

### 如何安装
#### 1. 首先下载cbis-18.5-kvmrt-20180829.tar.gz文件并上传至undercloud

#### 2. 登录undercloud，以root用户身份在根目录解压cbis-18.5-kvmrt-20180829.tar.gz文件

```
cd /
tar zxf /tmp/cbis-18.5-kvmrt-20180829.tar.gz
```

#### 3. 登录undercloud, 以stack用户身份针对计算节点执行以下命令为计算节点添加KVM Realtime功能

以overcloud-OvsCompute-0为例
```
cd /usr/share/cbis/cbis-ansible/post-install
ansible-playbook --limit overcloud-OvsCompute-0 configure_kvmrt_overcloud.yml
```

*注意*
* 建议首先指定某个计算节点进行功能，性能和网元验证
