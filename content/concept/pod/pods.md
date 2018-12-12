---
date: 2018-12-08T09:00:00+08:00
title: Pods
menu:
  main:
    parent: "concept-pod"
weight: 311
description : "Kubernetes中的pod概念"
---

> 备注： 内容来自 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)

Pod是在Kubernetes中可以创建和管理的最小的可部署计算单元。

## Pod是什么?

pod是一个或多个容器（如Docker容器）的分组，具有共享存储/网络，以及如何运行容器的规范。pod的内容始终位于同一位置并共同调度，并在共享上下文中运行。pod模拟特定于应用程序的“逻辑主机” - 它包含一个或多个相对紧密耦合的应用程序容器 - 在容器世界之前，在同一物理或虚拟机上执行意味着在同一逻辑主机上执行。

虽然Kubernetes支持的容器运行时不止Docker，但Docker是最常见的运行时，它有助于用Docker术语描述pod。

pod的共享上下文是一组Linux命名空间，cgroup，以及其他可能的隔离方面 - 与隔离Docker容器相同的东西。在pod的上下文中，各个应用程序可能会应用进一步的子隔离。

pod中的容器共享IP地址和端口空间，并且可以通过 `localhost` 找到彼此。它们还可以使用诸如SystemV 信号量或 POSIX 共享内存等标准进程间通信方式相互通信。不同pod中的容器具有不同的IP地址，并且在没有特殊配置的情况下无法通过IPC进行通信 。这些容器通常通过Pod IP地址相互通信。

Pod中的应用程序还可以访问共享卷，共享卷被定义为pod的一部分，可以挂载到每个应用程序的文件系统中。

就Docker构造而言，pod被建模为一组具有共享命名空间和共享卷的Docker容器 。

与单个应用容器一样，pod被认为是相对短暂的（而不是持久的）实体。正如在pod的生命周期中所讨论的，创建pod，分配唯一ID（UID），并调度到它们保留的节点，直到终止（根据重启策略）或删除。如果节点终止，则在超时期限之后，将调度到该节点的Pod删除。给定的pod（由UID定义）不会“重新安排”到新节点; 相反，它可以被相同的pod替换，如果需要，甚至可以使用相同的名称，但是具有新的UID。

当某些东西被认为具有与pod相同的生命周期时，例如卷，这意味着只要该pod（具有该UID）存在就存在。如果由于任何原因删除了该pod，即使创建了相同的替换，相关的东西（例如卷）也会被销毁并重新创建。

![](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)

一个多容器pod，包含文件提取程序和Web服务器，该服务器使用持久卷在容器之间共享存储。

## Pod的动机

### 管理

Pod是多个合作进程模式的模型，形成了一个有凝聚力的服务单元。它们通过提供比组成pod的应用集合更高级别的抽象来简化应用的部署和管理。Pod用作部署，水平扩展和复制的单元。对容器中的容器自动处理共置（共同调度），共享命运（例如终止），协调复制，资源共享和依赖关系管理。

### 资源共享和沟通

Pod可以实现其成员之间的数据共享和通信。

pod中的应用程序都使用相同的网络命名空间（相同的IP和端口空间），因此可以相互“找到”并使用`localhost`进行通信。因此，pod中的应用程序必须协调它们对端口的使用。每个pod在平面共享网络空间中具有IP地址，该网络空间与网络中的其他物理计算机和pod完全通信。

对于pod中的应用程序容器，hostname 被设置为pod名称。

除了定义在pod中运行的应用程序容器之外，pod还指定了一组共享存储卷。卷使数据能够在容器重新启动后继续存在，并在容器内的应用程序之间共享。

## Pod的使用

Pod可用于托管垂直集成的应用程序堆栈（例如LAMP），但其主要动机是支持共址，共同管理的帮助程序，例如：

* 内容管理系统，文件和数据加载器，本地缓存管理器等。
* 日志和检查点备份，压缩，rotation，快照等
* 数据变更观察者，日志tailers，日志和监控适配器，事件发布者等。
* 代理，网桥和适配器
* 控制器，管理器，配置器和更新器

通常，单个pod不用于运行同一应用程序的多个实例。

## 考虑的替代方案

为什么不在一个（Docker）容器中运行多个程序？

1. 透明度。让pod内的容器对基础设施可见，使得基础设施能够为这些容器提供服务，例如进程管理和资源监视。这为用户提供了许多便利。
1. 解耦软件依赖关系。各个容器可以独立地进行版本控制，重建和重新部署。Kubernetes甚至可能有一天能支持单个容器的实时更新。
1. 便于使用。用户无需运行自己的流程管理器，担心信号和退出代码传播等。
1. 效率。由于基础设施承担更多责任，因此集装箱可以更轻量。

为什么不支持基于亲和力的容器协同调度？

这种方法可以提供协同定位，但不会提供pod的大部分好处，例如资源共享，IPC，保证命运共享和简化管理。

## Pod的耐久性

Pod不应被视为耐用实体。它们将无法在调度故障，节点故障或其他逐出（例如由于缺乏资源或节点维护的情况下）中存活。

通常，用户不需要直接创建pod。他们应该几乎总是使用控制器，即使是单例pod，例如，Deployment 控制器提供集群范围的自我修复，以及复制和部署管理。像 StatefulSet 这样的控制器 也可以为有状态的pod提供支持。

使用API集合作为主要的面向用户的原语在集群调度系统中相对常见，包括Borg，Marathon，Aurora和Tupperware。

Pod作为原语公开，以便于：

* 调度程序和控制器的可插拔性
* 支持pod级操作，无需通过控制器API“代理”它们
* pod寿命与控制器生命周期的分离，例如bootstrapping
* 控制器和服务的分离 - 端点控制器只是监视pod
* 具有集群级功能的Kubelet级功能的清晰组合 - Kubelet实际上是“pod控制器”
* 高可用性应用程序，它们将期望在终止之前，并且肯定在删除之前更换pod，例如在计划逐出或镜像预取的情况下。

## Pod的创建

原文没有讲pod的创建，补充一个图，pod 创建的流程：

![](/Users/aoxiaojian/work/code/learning/learning-kubernetes/content/concept/pod/images/pod-start.png)

图片来源：[Core Kubernetes: Jazz Improv over Orchestration](https://blog.heptio.com/core-kubernetes-jazz-improv-over-orchestration-a7903ea92ca)，详细内容可见 [Kubernetes: Lifecycle of a Pod](https://dzone.com/articles/kubernetes-lifecycle-of-a-pod)

## Pod的终止

因为pod代表集群中节点上的正在运行的进程，所以允许这些进程在不再需要时优雅地终止非常重要（使用KILL信号粗暴杀死并且没有机会清理）。用户应该能够请求删除并知道进程何时终止，但也能够确保删除最终完成。当用户请求删除pod时，系统会在允许pod被强制终止之前记录预期的宽限期，并将TERM信号发送到每个容器中的主进程。宽限期到期后，KILL信号将发送到这些进程，然后从API server中删除该pod。如果在等待进程终止时，重新启动了Kubelet或容器管理器，终止会使用完全宽限期重试。

示例流程：

1. 用户发送删除Pod的命令，默认宽限期（30s）
1. API server中的Pod随着时间的推移而更新，其中Pod在宽限期内被视为“死”。
1. 在客户端命令中列出时，Pod显示为“Terminating”
1. （与3同时）当Kubelet发现Pod已被标记为terminating，因为已经设置了2中的时间，它开始pod关闭过程。
	1. 如果pod已定义了 preStop 钩子，则会在pod中调用它。如果 preStop 钩子在宽限期到期后仍在运行，则以小的延长的宽限期（2秒）调用步骤2。
	1. 向Pod中的进程发送TERM信号。
1. （与3同时）从服务的端点列表中删除pod，并且不再被视为副本控制器的运行中的pod集合的一部分。缓慢关闭的pod无法继续为流量提供服务，因为负载均衡器（如service proxy）会将其从循环中移除。
1. 当宽限期到期时，仍然在Pod中运行的任何进程都将被 SIGKILL 杀死。
1. Kubelet将通过设置宽限期0（立即删除）完成删除API server上的Pod。Pod从API中消失，不再从客户端可见。

默认情况下，所有删除在30秒内都是优雅的。`kubectl delete` 命令支持 `--grace-period=<seconds>` 属性，允许用户覆盖默认值并指定其自己的值。值`0` 强制删除 pod。在kubectl 版本 >=1.5 时，必须和 `--grace-period=0` 一起指定一个额外的标志 `--force` ，以执行强制删除。

补充一个pod 终止的图：

![](images/pod-terminate.jpg)

图片来源：[Kubernetes: Lifecycle of a Pod](https://dzone.com/articles/kubernetes-lifecycle-of-a-pod)

### 强制删除pod

强制删除pod被定义为立即从集群状态和 etcd 中删除 pod。当执行强制删除时，api server 不会等待来自 kubelet 的确认（确认该pod已在其运行的节点上终止）。它会立即删除 API 中的 pod，以便可以使用相同的名称创建新的 pod。在节点上，设置为立即终止的 pod 在被强制终止之前仍将被给予一个小的宽限期。

强制删除可能对某些 pod 有潜在危险，应谨慎执行。

## Pod容器的特权模式

从Kubernetes v1.1开始，pod中的任何容器都可以使用容器规范中 `SecurityContext` 上的 `privileged` 标志启用特权模式。这对于想要使用Linux功能的容器非常有用，例如操作网络堆栈和访问设备。容器内的进程获得与容器外部进程可用的几乎相同的权限。使用特权模式，可以更容易编写网络和卷插件做为独立的pod，而不需要编译到kubelet中。

## API 对象

Pod是Kubernetes REST API中的顶级资源。








