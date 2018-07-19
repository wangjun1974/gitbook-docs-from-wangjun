# 航信rhv交流
1. rhv红帽的投入（研发团队人员数量或者分布之类的信息）以及社区状况
2. rhv最新版本特性简单介绍
3. rhv的roadmap的分享

## ovirt release status

### ovirt release schedule

Release|Start|GA
:-:|:--------:|:--------:
3.5|2014-03-27|2014-10-17
3.6|2014-10-17|2015-11-04
4.0|2015-11-04|2016-06-15
4.1|2016-06-15|2017-02-01
4.2|2017-02-01|2017-12-20
4.2+|2017-12-20|2018-06-21

### ovirt community status for travelsky

Project|Release|Commits|Commitors|Redhaters|Others 
:----------:|:-:|:--:|:-:|:-:|:-:
ovirt-engine|3.6|4494|92|87|5
ovirt-engine|4.1|2889|68|65|3
ovirt-engine|4.2|3536|84|76|8
vdsm|3.6|1684|81|66|15
vdsm|4.1|1438|54|50|4
vdsm|4.2|1727|55|50|5
ovirt-ansible|4.2|131|19|12|7

### 新特性介绍
- 基于标签的Affinity和Anti-Affinity
- RHV 4.1+包含Ansible Engine
- 与Ansible Engine和Ansible Tower集成
- 灾备
- 高性能虚拟机
- vGPU
- Metrics与Loging Collectd，ElasticSearch，Fluentd和Kibana
- 新用户界面
- 迁移策略
- Cockpit administration console

### Ovirt DR
There are four actions which the user can execute: 

actions|detail
:------|:-------
**Validate**|Validate the var file mapping which is used for failover and failback 
**Generate**|Generate the mapping var file based on the primary and secondary setup, to be used for failover and failback
**Failover**|Start a failover process to the target setup
**Failback**|Start a failback process from the target setup to the source setup

### High Performance VM
Furthermore, few required features essential for improving VM performance were not supported at all by oVirt (for example: using huge pages, IO Thread pinning and CPU cache layer 3) and new features as Headless VMs can now be leveraged to suggest one solution of the best recommended configuration according to VM usage requirements.

### reference
ID|URL
:-:|:------
1|[Automate your RHV Configuration with Ansible](https://rhelblog.redhat.com/2017/11/20/automate-your-rhv-configuration-with-ansible/)
2|[Cisco ACI and Red Hat Virtualization](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/kb/b_Cisco_ACI_Red_Hat_Virtualization.html)
3|[oVirt Disaster Recovery](https://github.com/oVirt/ovirt-ansible-disaster-recovery)
4|[High Performance VM](https://www.ovirt.org/develop/release-management/features/virt/high-performance-vm/)
5|[NVIDIA Grid vGPU Support in RHV](https://access.redhat.com/solutions/3221241)
6|[Red Hat Virtualization Reporting Evolution: Transitioning to Metrics Store](https://rhelblog.redhat.com/2017/05/23/red-hat-virtualization-reporting-evolution-transitioning-to-metrics-store/)
7|[Please Welcome Red Hat Virtualization 4.2](https://rhelblog.redhat.com/2018/05/15/please-welcome-red-hat-virtualization-4-2/)
8|[Label Based Affinity](https://www.redhat.com/en/blog/red-hat-virtualization-4-new-features-foundational-technology)
9|[Advanced Live Migration Policies](https://www.redhat.com/en/blog/red-hat-virtualization-4-new-features-foundational-technology)
10|[RHV Cluster Migration Policy in details](https://access.redhat.com/solutions/3143541)
11|[PatternFly](http://www.patternfly.org/)
12|[Red Hat Virtualization 4: Great new Features with the 10th Release – Comparison Matrix updated](https://www.whatmatrix.com/portal/red-hat-virtualization-4-greet-new-features-with-the-10th-release-comparison-matrix-updated/)
13|[Features by Release](https://www.ovirt.org/develop/release-management/features/)
