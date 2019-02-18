
# 利用K8S技术栈打造个人私有云（连载之：初章）


![iMac Pro](http://upload-images.jianshu.io/upload_images/9824247-f5551626a3c69a3d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



---

## 我的想法是什么

最近在学习Docker技术，相信Docker技术大家都有所了解，Docker类似于虚拟机（但与虚拟机又有本质不同），提供进程级别的隔离。我们可以利用Docker来方便地来做很多事情，比如搭建一个翻墙VPN、搞一个爬虫、弄一个私人博客，部署一个裸机上比较难以安装的环境等等……可以说几乎没有什么目的办不到，这简直是宅男老铁们的福利啊！

但话又说回来，单个Docker所能发挥的作用毕竟有限，也不便于批量管理，更满足不了各种量比较大的业务场景所需的高可用、弹性伸缩等特性，所以Docker得组集群来并赋予各种完善的调度机制才能发挥强大的技术优势。既然要组集群那就涉及诸如Docker的资源调度、管理等等一系列问题。Docker集群技术发展得很火热， 目前涉及Docker集群的三个主要的技术无外乎Docker Swarm、Kubernetes、Mesos三种主流方案。

Docker Swarm是Docker提供的原生集群技术，我只做过一些初步实践（[Docker Swarm集群初探](https://www.jianshu.com/p/3f3c9e0e3db5)），发现还比较容易上手，大家也可以自行去深入学习一下，我就不多说了。

Kubernetes（以下简称K8S）源自于Google，是一个为容器化应用提供自动部署、扩容和管理的开源项目，社区非常活跃，也是用得更加广泛的Docker集群技术。我最近也是花了一些时间在这上面进行学习，但由于缺少实际实践经验，总有点不痛不痒的感觉，所以没办法只能自己来创造一些实践，就想着用它来做出点什么出来。

好，背景介绍完了。那我到底想用我刚自学的Docker和Kubernetes来做一件什么事情呢？听我慢慢道来...

>**注：** 本文原载于  [**My Personal Blog：**](http://www.codesheep.cn)， [**CodeSheep · 程序羊**](http://www.codesheep.cn) ！

---

当下云主机可以说非常火热了，不知道大家是否用过BAT等一系列厂商旗下XX云所提供的云主机服务。我们只需要买一个云主机，然后就可以尽情地去上面干各种事情了，常见的比如建站、搭博客、部署服务甚至直接买一个windows云主机直接用于办公。

以某个云服务为例，来张图看看：

![某个云服务的控制台](http://upload-images.jianshu.io/upload_images/9824247-46f8ac0b0fbefb6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们就可以进去付费创建一个云主机自己使用，就像下面这样：

![实例化（创建）云主机](http://upload-images.jianshu.io/upload_images/9824247-5fd6ce3bb6d34e4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种服务如今如此地火热，我想这背后肯定少不了的就是容器技术和集群技术的加持，想到这里我想大家应该明白我这篇文章的主题了。是的，本篇文章及接下来的连载系列文章将详细讲述如何用k8s技术栈打造一个属于自己的私有云服务（取名为 **SheepCloud**，怎么样是不是很时髦...）。这样的话，我自己在家就可以申请创建很多云主机节点，然后自己想做啥就做啥，什么云计算、分布式实验统统不都可以免费进行了！

嗯，理想是好的，接下来还有一大堆事情要做呢...

---

## 我准备打造什么样形式的个人私有云

其实上面已经说过了，准备模仿那些云服务提供商的云主机功能，先在网页上申请创建云主机，创建成功后分配 **IP地址/子网号 + 用户名 + 密码** 给用户，这样用户就可以用用ssh方式连入分配到的具有独立IP的云主机中进行工作，这样就和那些服务商提供的云主机服务没有什么不同了。

所以首先得有前端页面，我自己用Vue.js写了一个Demo（目前还未跟后端联调），让大家有个感性的认识：

![SheepCloud控制台界面](http://upload-images.jianshu.io/upload_images/9824247-df82fba2398131fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

## 我准备如何来入手这个小项目

本来我的初衷就是想深入实践一下Docker和Kubernetes（以下简称K8S）是怎么玩的，但东西还真不少，总结了一下涉及的技术，可能还不止这些：
- Docker：不用多说，毕竟负责容器的落地，云主机本质上就是一个win或linux容器
- Kubernetes：管理Docker的集群技术，这里面是有很多kube的组件
- flannel：负责节点中容器间的通信以及私有云各个实例的IP地址规划
- etcd：分布式数据库，kubernetes和flannel都需要它
- SpringBt：驱动后端服务
- Vue.js：编写私有云前端WEB页面
…

我自己规划了一个基本路线来入手：

- 熟悉Docker
- 熟悉Kubernetes基本概念并搭建K8S集群
- K8S集群理解与练手实验
- 基础镜像制作与实验，能完成单个操作系统容器的手动管理
- K8S资源控制代码编写，能实现集群对容器资源的自动控制
- 私有云客户端WEB前端页面编写
- 前后端联调
- 总结输出

---

## 我准备输出哪些东西

准备输出系列连载文章，本篇文章是连载系列的第一篇

- [利用K8S技术栈打造个人私有云（连载之：初章） ](https://www.jianshu.com/p/9bc87b5380e8)
- [利用K8S技术栈打造个人私有云（连载之：K8S集群搭建）](https://www.jianshu.com/p/7d1fb03b8925)
- [利用K8S技术栈打造个人私有云（连载之：K8S环境理解和练手）](https://www.jianshu.com/p/5b0cd99e0332)
- [利用K8S技术栈打造个人私有云（连载之：基础镜像制作与实验）](https://www.jianshu.com/p/e38c05cf076a)
- [利用K8S技术栈打造个人私有云（连载之：资源控制研究）](https://www.jianshu.com/p/58a98e65074c)
- [利用K8S技术栈打造个人私有云（连载之：私有云客户端打造）](https://www.jianshu.com/p/a7cdb3ab4e11)

---

## 总结

学以致用这个词我近来感触颇深，学一门技术，如果不辅之以实践，真的很难深入其中。浮在表面不痛不痒地学习真心很不爽，没有实践，自己制造实践也要上！大家共勉

---

## 后记

> 由于能力有限，若有错误或者不当之处，还请大家批评指正，一起学习交流！

- [My Personal Blog](http://www.codesheep.cn/)
- [我的半年技术博客之路](https://www.jianshu.com/p/28ba53821450)

---


---