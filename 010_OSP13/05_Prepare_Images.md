<!-- toc -->


### 准备镜像


#### 生成 local_registry_images.yaml 和 overcloud_images.yaml

```
#!/bin/bash
openstack overcloud container image prepare \
  -e /usr/share/openstack-tripleo-heat-templates/environments/ceph-ansible/ceph-ansible.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/collectd.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/congress.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/fluentd-client.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/ironic.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/sahara.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/ec2-api.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/barbican.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/octavia.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/manila.yaml \
  -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/sensu-client.yaml \
  --namespace=registry.access.redhat.com/rhosp13 \
  --push-destination=172.16.0.1:8787 \
  --prefix=openstack- \
  --tag-from-label {version}-{release} \
  --set ceph_namespace=registry.access.redhat.com/rhceph \
  --set ceph_image=rhceph-3-rhel7 \
  --output-env-file=/home/stack/templates/overcloud_images.yaml \
  --output-images-file /home/stack/local_registry_images.yaml
```


文件 `local_registry_images.yaml`
> 描述如何从远端 registry 获取镜像


文件 `overcloud_images.yaml`
> 描述如何向本地 registry 上传镜像


#### 下载镜像
> ** 注意：确认有足够的空间保存镜像。下载并导出所需空间大约需要20G **
```
cat local_registry_images.yaml | grep imagename | awk '{print $3}' | sed -e 's,^,docker pull ,g' | tee osp13-image-pull-20180726.sh
/usr/bin/nohup /bin/bash osp13-image-pull-20180726.sh & 
```

#### 保存镜像
脚本
```
[stack@undercloud ~]$ cat osp13-image-save-20180726.sh  
#!/bin/bash
cd ~
mkdir -p images
docker save $(docker images -q) -o /home/stack/images/osp13-images-20180726.tar 
```


执行脚本
```
/usr/bin/nohup /bin/bash -x osp13-image-save-20180726.sh &
```
> **注意：需要拷贝 osp13-images-20180726.tar 到离线 undercloud**


#### 生成镜像 Tag 列表
```
docker images  | sed '1d' | awk '{print $1 " " $2 " " $3}'  > images/osp13-images-20180726.list
```
> **注意：需要拷贝 osp13-images-20180726.list 文件到离线 undercloud**


#### 加载镜像
离线 undercloud docker engine 加载镜像


脚本
```
[stack@undercloud ~]$ cat osp13-image-load-20180726.sh 
docker load -i /home/stack/images/osp13-images-20180726.tar 
```


执行脚本
```
/usr/bin/nohup /bin/bash -x osp13-image-load-20180726.sh &
```


#### 镜像打 Tag
离线 undercloud docker engine 镜像打 Tag


脚本
```
[stack@undercloud ~]$ cat osp13-image-tag-20180726.sh 
LOCALREGISTRY="172.16.0.1:8787"

while read REPOSITORY TAG IMAGE_ID
do
        echo "== Tagging $REPOSITORY $TAG $IMAGE_ID =="
        docker tag "$IMAGE_ID" "$REPOSITORY:$TAG"
        docker tag "$IMAGE_ID" $( echo $REPOSITORY | sed -e "s,registry.access.redhat.com,$LOCALREGISTRY," )":$TAG"
done < /home/stack/images/osp13-images-20180726.list
```


执行
```
/usr/bin/nohup /bin/bash -x osp13-image-tag-20180726.sh &
```


#### 上传镜像
离线 undercloud 拷贝 overcloud_images.yaml 到 templates 目录下，生成脚本
```
cat templates/overcloud_images.yaml | grep Image | awk '{print $2}'   | sort -u | sed -e 's,^,docker push ,' | tee osp13-image-push-20180726.sh
```

执行
```
/usr/bin/nohup /bin/bash -x osp13-image-push-20180726.sh &
```


#### 文件 local_registry_images.yaml

```
container_images:
- imagename: registry.access.redhat.com/rhosp13/openstack-aodh-api:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-aodh-evaluator:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-aodh-listener:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-aodh-notifier:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-barbican-api:13.0-41
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-barbican-keystone-listener:13.0-43
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-barbican-worker:13.0-43
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-ceilometer-central:13.0-44
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-ceilometer-compute:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-ceilometer-notification:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-cinder-api:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-cinder-scheduler:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-cinder-volume:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-collectd:13.0-41
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-cron:13.0-51
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-ec2-api:13.0-49
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-glance-api:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-gnocchi-api:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-gnocchi-metricd:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-gnocchi-statsd:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-haproxy:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-heat-api-cfn:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-heat-api:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-heat-engine:13.0-44
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-horizon:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-ironic-api:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-ironic-conductor:13.0-44
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-ironic-pxe:13.0-40
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-iscsid:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-keystone:13.0-44
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-manila-api:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-manila-scheduler:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-manila-share:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-mariadb:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-memcached:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-neutron-dhcp-agent:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-neutron-l3-agent:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-neutron-metadata-agent:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-neutron-openvswitch-agent:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-neutron-server:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-api:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-compute-ironic:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-compute:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-conductor:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-consoleauth:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-libvirt:13.0-52
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-novncproxy:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-placement-api:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-nova-scheduler:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-octavia-api:13.0-43
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-octavia-health-manager:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-octavia-housekeeping:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-octavia-worker:13.0-44
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-panko-api:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-rabbitmq:13.0-47
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-redis:13.0-49
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-sahara-api:13.0-44
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-sahara-engine:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-sensu-client:13.0-43
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-swift-account:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-swift-container:13.0-48
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-swift-object:13.0-45
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhosp13/openstack-swift-proxy-server:13.0-46
  push_destination: 172.16.0.1:8787
- imagename: registry.access.redhat.com/rhceph/rhceph-3-rhel7:3-11
  push_destination: 172.16.0.1:8787
```


#### 文件overcloud_images.yaml
```
# the following on 2018-07-27T09:38:22.170303
#
#   openstack overcloud container image prepare -e /usr/share/openstack-tripleo-heat-templates/environments/ceph-ansible/ceph-ansible.yaml -e /usr/sha
re/openstack-tripleo-heat-templates/environments/services-docker/collectd.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/services-do
cker/congress.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/fluentd-client.yaml -e /usr/share/openstack-tripleo-hea
t-templates/environments/services-docker/ironic.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/sahara.yaml -e /usr/s
hare/openstack-tripleo-heat-templates/environments/services-docker/ec2-api.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/services-d
ocker/barbican.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/octavia.yaml -e /usr/share/openstack-tripleo-heat-temp
lates/environments/services-docker/manila.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/sensu-client.yaml --namespa
ce=registry.access.redhat.com/rhosp13 --push-destination=172.16.0.1:8787 --prefix=openstack- --tag-from-label {version}-{release} --set ceph_namespace
=registry.access.redhat.com/rhceph --set ceph_image=rhceph-3-rhel7 --output-env-file=/home/stack/templates/overcloud_images.yaml --output-images-file 
/home/stack/local_registry_images.yaml
#

parameter_defaults:
  DockerAodhApiImage: 172.16.0.1:8787/rhosp13/openstack-aodh-api:13.0-47
  DockerAodhConfigImage: 172.16.0.1:8787/rhosp13/openstack-aodh-api:13.0-47
  DockerAodhEvaluatorImage: 172.16.0.1:8787/rhosp13/openstack-aodh-evaluator:13.0-46
  DockerAodhListenerImage: 172.16.0.1:8787/rhosp13/openstack-aodh-listener:13.0-46
  DockerAodhNotifierImage: 172.16.0.1:8787/rhosp13/openstack-aodh-notifier:13.0-46
  DockerBarbicanApiImage: 172.16.0.1:8787/rhosp13/openstack-barbican-api:13.0-41
  DockerBarbicanConfigImage: 172.16.0.1:8787/rhosp13/openstack-barbican-api:13.0-41
  DockerBarbicanKeystoneListenerConfigImage: 172.16.0.1:8787/rhosp13/openstack-barbican-keystone-listener:13.0-43
  DockerBarbicanKeystoneListenerImage: 172.16.0.1:8787/rhosp13/openstack-barbican-keystone-listener:13.0-43
  DockerBarbicanWorkerConfigImage: 172.16.0.1:8787/rhosp13/openstack-barbican-worker:13.0-43
  DockerBarbicanWorkerImage: 172.16.0.1:8787/rhosp13/openstack-barbican-worker:13.0-43
  DockerCeilometerCentralImage: 172.16.0.1:8787/rhosp13/openstack-ceilometer-central:13.0-44
  DockerCeilometerComputeImage: 172.16.0.1:8787/rhosp13/openstack-ceilometer-compute:13.0-46
  DockerCeilometerConfigImage: 172.16.0.1:8787/rhosp13/openstack-ceilometer-central:13.0-44
  DockerCeilometerNotificationImage: 172.16.0.1:8787/rhosp13/openstack-ceilometer-notification:13.0-45
  DockerCephDaemonImage: 172.16.0.1:8787/rhceph/rhceph-3-rhel7:3-11
  DockerCinderApiImage: 172.16.0.1:8787/rhosp13/openstack-cinder-api:13.0-46
  DockerCinderConfigImage: 172.16.0.1:8787/rhosp13/openstack-cinder-api:13.0-46
  DockerCinderSchedulerImage: 172.16.0.1:8787/rhosp13/openstack-cinder-scheduler:13.0-45
  DockerCinderVolumeImage: 172.16.0.1:8787/rhosp13/openstack-cinder-volume:13.0-45
  DockerClustercheckConfigImage: 172.16.0.1:8787/rhosp13/openstack-mariadb:13.0-47
  DockerClustercheckImage: 172.16.0.1:8787/rhosp13/openstack-mariadb:13.0-47
  DockerCollectdConfigImage: 172.16.0.1:8787/rhosp13/openstack-collectd:13.0-41
  DockerCollectdImage: 172.16.0.1:8787/rhosp13/openstack-collectd:13.0-41
  DockerCrondConfigImage: 172.16.0.1:8787/rhosp13/openstack-cron:13.0-51
  DockerCrondImage: 172.16.0.1:8787/rhosp13/openstack-cron:13.0-51
  DockerEc2ApiConfigImage: 172.16.0.1:8787/rhosp13/openstack-ec2-api:13.0-49
  DockerEc2ApiImage: 172.16.0.1:8787/rhosp13/openstack-ec2-api:13.0-49
  DockerGlanceApiConfigImage: 172.16.0.1:8787/rhosp13/openstack-glance-api:13.0-47
  DockerGlanceApiImage: 172.16.0.1:8787/rhosp13/openstack-glance-api:13.0-47
  DockerGnocchiApiImage: 172.16.0.1:8787/rhosp13/openstack-gnocchi-api:13.0-45
  DockerGnocchiConfigImage: 172.16.0.1:8787/rhosp13/openstack-gnocchi-api:13.0-45
  DockerGnocchiMetricdImage: 172.16.0.1:8787/rhosp13/openstack-gnocchi-metricd:13.0-46
  DockerGnocchiStatsdImage: 172.16.0.1:8787/rhosp13/openstack-gnocchi-statsd:13.0-46
  DockerHAProxyConfigImage: 172.16.0.1:8787/rhosp13/openstack-haproxy:13.0-47
  DockerHAProxyImage: 172.16.0.1:8787/rhosp13/openstack-haproxy:13.0-47
  DockerHeatApiCfnConfigImage: 172.16.0.1:8787/rhosp13/openstack-heat-api-cfn:13.0-45
  DockerHeatApiCfnImage: 172.16.0.1:8787/rhosp13/openstack-heat-api-cfn:13.0-45
  DockerHeatApiConfigImage: 172.16.0.1:8787/rhosp13/openstack-heat-api:13.0-46
  DockerHeatApiImage: 172.16.0.1:8787/rhosp13/openstack-heat-api:13.0-46
  DockerHeatConfigImage: 172.16.0.1:8787/rhosp13/openstack-heat-api:13.0-46
  DockerHeatEngineImage: 172.16.0.1:8787/rhosp13/openstack-heat-engine:13.0-44
  DockerHorizonConfigImage: 172.16.0.1:8787/rhosp13/openstack-horizon:13.0-45
  DockerHorizonImage: 172.16.0.1:8787/rhosp13/openstack-horizon:13.0-45
  DockerInsecureRegistryAddress:
  - 172.16.0.1:8787
  DockerIronicApiConfigImage: 172.16.0.1:8787/rhosp13/openstack-ironic-api:13.0-46
  DockerIronicApiImage: 172.16.0.1:8787/rhosp13/openstack-ironic-api:13.0-46
  DockerIronicConductorImage: 172.16.0.1:8787/rhosp13/openstack-ironic-conductor:13.0-44
  DockerIronicConfigImage: 172.16.0.1:8787/rhosp13/openstack-ironic-pxe:13.0-40
  DockerIronicPxeImage: 172.16.0.1:8787/rhosp13/openstack-ironic-pxe:13.0-40
  DockerIscsidConfigImage: 172.16.0.1:8787/rhosp13/openstack-iscsid:13.0-45
  DockerIscsidImage: 172.16.0.1:8787/rhosp13/openstack-iscsid:13.0-45
  DockerKeystoneConfigImage: 172.16.0.1:8787/rhosp13/openstack-keystone:13.0-44
  DockerKeystoneImage: 172.16.0.1:8787/rhosp13/openstack-keystone:13.0-44
  DockerManilaApiImage: 172.16.0.1:8787/rhosp13/openstack-manila-api:13.0-47
  DockerManilaConfigImage: 172.16.0.1:8787/rhosp13/openstack-manila-api:13.0-47
  DockerManilaSchedulerImage: 172.16.0.1:8787/rhosp13/openstack-manila-scheduler:13.0-46
  DockerManilaShareImage: 172.16.0.1:8787/rhosp13/openstack-manila-share:13.0-45
  DockerMemcachedConfigImage: 172.16.0.1:8787/rhosp13/openstack-memcached:13.0-46
  DockerMemcachedImage: 172.16.0.1:8787/rhosp13/openstack-memcached:13.0-46
  DockerMysqlClientConfigImage: 172.16.0.1:8787/rhosp13/openstack-mariadb:13.0-47
  DockerMysqlConfigImage: 172.16.0.1:8787/rhosp13/openstack-mariadb:13.0-47
  DockerMysqlImage: 172.16.0.1:8787/rhosp13/openstack-mariadb:13.0-47
  DockerNeutronApiImage: 172.16.0.1:8787/rhosp13/openstack-neutron-server:13.0-48
  DockerNeutronConfigImage: 172.16.0.1:8787/rhosp13/openstack-neutron-server:13.0-48
  DockerNeutronDHCPImage: 172.16.0.1:8787/rhosp13/openstack-neutron-dhcp-agent:13.0-48
  DockerNeutronL3AgentImage: 172.16.0.1:8787/rhosp13/openstack-neutron-l3-agent:13.0-47
  DockerNeutronMetadataImage: 172.16.0.1:8787/rhosp13/openstack-neutron-metadata-agent:13.0-48
  DockerNovaApiImage: 172.16.0.1:8787/rhosp13/openstack-nova-api:13.0-48
  DockerNovaComputeImage: 172.16.0.1:8787/rhosp13/openstack-nova-compute:13.0-48
  DockerNovaComputeIronicImage: 172.16.0.1:8787/rhosp13/openstack-nova-compute-ironic:13.0-48
  DockerNovaConductorImage: 172.16.0.1:8787/rhosp13/openstack-nova-conductor:13.0-48
  DockerNovaConfigImage: 172.16.0.1:8787/rhosp13/openstack-nova-api:13.0-48
  DockerNovaConsoleauthImage: 172.16.0.1:8787/rhosp13/openstack-nova-consoleauth:13.0-47
  DockerNovaLibvirtConfigImage: 172.16.0.1:8787/rhosp13/openstack-nova-compute:13.0-48
  DockerNovaLibvirtImage: 172.16.0.1:8787/rhosp13/openstack-nova-libvirt:13.0-52
  DockerNovaMetadataImage: 172.16.0.1:8787/rhosp13/openstack-nova-api:13.0-48
  DockerNovaPlacementConfigImage: 172.16.0.1:8787/rhosp13/openstack-nova-placement-api:13.0-48
  DockerNovaPlacementImage: 172.16.0.1:8787/rhosp13/openstack-nova-placement-api:13.0-48
  DockerNovaSchedulerImage: 172.16.0.1:8787/rhosp13/openstack-nova-scheduler:13.0-48
  DockerNovaVncProxyImage: 172.16.0.1:8787/rhosp13/openstack-nova-novncproxy:13.0-48
  DockerOctaviaApiImage: 172.16.0.1:8787/rhosp13/openstack-octavia-api:13.0-43
  DockerOctaviaConfigImage: 172.16.0.1:8787/rhosp13/openstack-octavia-api:13.0-43
  DockerOctaviaHealthManagerImage: 172.16.0.1:8787/rhosp13/openstack-octavia-health-manager:13.0-45
  DockerOctaviaHousekeepingImage: 172.16.0.1:8787/rhosp13/openstack-octavia-housekeeping:13.0-45
  DockerOctaviaWorkerImage: 172.16.0.1:8787/rhosp13/openstack-octavia-worker:13.0-44
  DockerOpenvswitchImage: 172.16.0.1:8787/rhosp13/openstack-neutron-openvswitch-agent:13.0-48
  DockerPankoApiImage: 172.16.0.1:8787/rhosp13/openstack-panko-api:13.0-47
  DockerPankoConfigImage: 172.16.0.1:8787/rhosp13/openstack-panko-api:13.0-47
  DockerRabbitmqConfigImage: 172.16.0.1:8787/rhosp13/openstack-rabbitmq:13.0-47
  DockerRabbitmqImage: 172.16.0.1:8787/rhosp13/openstack-rabbitmq:13.0-47
  DockerRedisConfigImage: 172.16.0.1:8787/rhosp13/openstack-redis:13.0-49
  DockerRedisImage: 172.16.0.1:8787/rhosp13/openstack-redis:13.0-49
  DockerSaharaApiImage: 172.16.0.1:8787/rhosp13/openstack-sahara-api:13.0-44
  DockerSaharaConfigImage: 172.16.0.1:8787/rhosp13/openstack-sahara-api:13.0-44
  DockerSaharaEngineImage: 172.16.0.1:8787/rhosp13/openstack-sahara-engine:13.0-45
  DockerSensuClientImage: 172.16.0.1:8787/rhosp13/openstack-sensu-client:13.0-43
  DockerSensuConfigImage: 172.16.0.1:8787/rhosp13/openstack-sensu-client:13.0-43
  DockerSwiftAccountImage: 172.16.0.1:8787/rhosp13/openstack-swift-account:13.0-46
  DockerSwiftConfigImage: 172.16.0.1:8787/rhosp13/openstack-swift-proxy-server:13.0-46
  DockerSwiftContainerImage: 172.16.0.1:8787/rhosp13/openstack-swift-container:13.0-48
  DockerSwiftObjectImage: 172.16.0.1:8787/rhosp13/openstack-swift-object:13.0-45
  DockerSwiftProxyImage: 172.16.0.1:8787/rhosp13/openstack-swift-proxy-server:13.0-46
```

