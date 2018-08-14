# CBIS Horizon 不显示 Heat User Password

## 在所有控制节点上

### 修改配置文件

**修改/etc/openstack-dashboard/local_settings**
```
OPENSTACK_HEAT_STACK = {
   'enable_user_pass': False,
}
```

### 重启服务

```
systemctl restart httpd
```


### ChangeLog
* 20180814 通过设置让 Horizon 不显示Heat Stack Password

