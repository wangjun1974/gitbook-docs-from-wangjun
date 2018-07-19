# RHV后续工作
------
### RHV与OVN，Neutron的状态更新


### RHV与存储的状态更新如Ceph


### RHV与Glance集成的状态更新


### cpuspeed是否能达到VMware环境下限速的目的

1. cpuspeed是什么

操作系统版本是什么版本



Clock scaling allows you to change the clock speed of the CPUs on the
    fly. This is a nice method to save battery power, because the lower
            the clock speed, the less power the CPU consumes.


什么是CPUFreq Governor
Most cpufreq drivers (except the intel_pstate and longrun) or even most
cpu frequency scaling algorithms only allow the CPU frequency to be set
to predefined fixed values.  In order to offer dynamic frequency
scaling, the cpufreq core must be able to tell these drivers of a
"target frequency". So these specific drivers will be transformed to
offer a "->target/target_index/fast_switch()" call instead of the
"->setpolicy()" call. For set_policy drivers, all stays the same,
though.

- [What Is A CPUFreq Governor](https://www.kernel.org/doc/Documentation/cpu-freq/governors.txt)


2. cpuspeed实现原理依赖什么



3. cpuspeed在VMware下是否能实现限速的目的
4. cpuspeed在VMware下如何实现限速
5. 除了cpuspeed还可以考虑如何实现cpu限速




### 如何实现VMware到Ovirt间的灾备

