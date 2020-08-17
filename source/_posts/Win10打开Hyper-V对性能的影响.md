---
title: Win10打开Hyper-V对性能的影响
date: 2020-08-01 15:46:49
tags:
---
## 引言
打算在家里电脑安装Docker，就顺便研究了一下Docker在Windows上运行的原理。其实是用Win10自带的Hyper-V新建了一台虚拟机，`Dockerd`运行在虚拟机上面 (切~~)。

顺便了解了一下现在主流的虚拟化方案，主要有三个：Vmware, VirtualBox, Hyper-V。
Vmware作为最老牌的虚拟机，对老机器兼容比较好，但个人感觉运行效率有点低，而且还要付费。
VirtualBox是开源软件，没用过，不做评价。
Hyper-V是Microsoft力推的虚拟机软件，Win10专业版自带，运行效率比较高。

但批评Hyper-V的人也有，有人说Hyper-V会将Host也虚拟化，造成Host性能损失。 ！！！ 这能忍？于是我赶紧测试了一下。

## 测试
根据网络上说法，在CPU方面会造成约4%的性能损失，这我倒是不在意。

关键在于Hyper-V会造成内存性能下降，以下是测试结果（均进行两次实验）：
#### 开启Hyper-V
![开启Hyper-V 1](/images/cachemem-hyper-v-V2019.png)
![开启Hpyer-V 2](/images/cachemem-hyper-v-V2020.png)
#### 关闭Hyper-V
![关闭Hyper-V 1](/images/cachemem-V2020-1.png)
![关闭Hyper-V 2](/images/cachemem-V2020-2.png)

可以看到开启Hyper-V后确实会对性能造成一定的影响，尤其是L2和L3缓存。

同时，Vmware不会对系统性能造成影响（废话）
![Vmware无影响](/images/cachemem-started-Vmware-V2020.png)
