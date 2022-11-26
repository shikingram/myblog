---
title: "边缘计算kubeedge"   
author: "Kingram"  
date: 2022-11-26   
lastmod: 2022-11-26

tags: [  
"cloud-native",
]
---

## 引言
之前花时间学习了kubeedge相关知识，这篇文章作为整理，希望通过自己的语言能把什么是边缘计算和kueedge相关的技术描述清楚。

如果有理解错误的地方，欢迎指正。

## 什么是边缘计算
边缘计算是一种分布式计算框架，他指的是靠近数据源物理位置附近的计算，怎么理解靠近数据源物理位置，比如树莓派接一个温度传感器，温度传感器就是一个物理设备他产生数据，树莓派就是靠近这个设备的节点，称为边缘节点，在这个节点上进行的数据计算，就是字面上理解的边缘计算。

## 为什么需要边缘计算
边缘计算是一个比较贴近现实需求的技术，很多场景都需要这样的模式取代传统的云平台，云平台面临的很多问题是很棘手的，网络延迟高、海量设备接入、数据处理难、带宽不够等。举例两个场景：

### 制造业
现代化的制造工厂有很多生产车间，成百上千的设备状态需要监控，这些数据时时刻刻都在产生，传统的数据处理中心是比较难处理这么多设备的，在设备附近进行处理数据比通过网络传输到数据中心速度更快，对故障的防止和设备状态的监控效率更高。

### 停车场
在接触边缘计算后我首先想到了我之前所在公司的模式，做的是停车，每一辆车出停车场都需要在云端的数据处理中心进行黑名单判定等一系列的耗时操作后才会交易成功抬杆放行。效率很低。

这种场景其实更好地处理方式是在每个停车场都能进行这样的耗时计算，减少网络消耗，停车场设备之间是内网，甚至串口硬件传输，效率是非常高的。同时在升级软件设备、日志状态监控等方面都有了统一的解决方案。

## kubeedge是什么
了解了边缘计算的概念后再看看[kubeedge](https://kubeedge.io/en/docs/)是怎么解决这样的边缘场景问题的。

kubeedge是基于kubernetes容器编排调度能力的开源系统，是华为捐献给CNCF的第一个开源项目，有个和他同样处理边缘问题的项目是K3S，kubeedge解决的是云边协同、大规模、轻量化、标准化等问题。

## kubeedge的架构简介
![kubeedge_arch](/img/cloud_native/kubeedge/kubeedge_arch.png)

这个图是kubeedge架构图，上下看，分为3层

### cloud 云端层
云端是k8s集群，同时运行控制核心组件，边缘节点加入k8s集群后在云端就可以统一控制

### edge 边缘层
边缘层运行的组件主要负责pod生命周期管理、不同协议的消息处理、数据存储、设备交互等功能

### device 设备层
连接的各种设备

## kubeedge各个组件功能
### cloud
云端主要有三个组件

**cloudhub**

这是云端的一个模块，他主要负责controller组件和edge边的交互，对边缘支持两种协议web-socket和[QUIC](https://quicwg.org/ops-drafts/draft-ietf-quic-applicability.html)
edge侧可以选择其中一种，edge侧和他对标的就是edgehub

**edgecontroller**

这个模块是k8s和边缘节点的交互桥梁，当然在edgecontroller和边缘节点之间还有个cloudhub

该组件两个控制器：下行控制、上行控制

![downstream](/img/cloud_native/kubeedge/DownstreamController.png)

下行控制主要负责增删改查运行在边缘侧的pod,configmap,secret等资源，同时创建管理这些资源的控制器，和定位资源到节点

![downstream](/img/cloud_native/kubeedge/UpstreamController.png)

上行控制主要负责提供这些资源的详细状态信息，user可以通过k8s集群查询触发控制

**devicecontroller**

设备控制模块，同样和edgecontroller一样分了上行控制器和下行控制器，他通过k8s的crd（devicemodel/device）资源描述设备。

devicemodel：这是设备模板资源，描述设备暴露的属性和权限

简单的例子：
```yaml
apiVersion: devices.kubeedge.io/v1alpha1
kind: DeviceModel
metadata:
  labels:
    description: 'TI Simplelink SensorTag Device Model'
    manufacturer: 'Texas Instruments'
    model: CC2650
  name: sensor-tag-model
spec:
  properties:
  - name: temperature
    description: temperature in degree celsius
    type:
      int:
        accessMode: ReadOnly
        maximum: 100
        unit: Degree Celsius
  - name: temperature-enable
    description: enable data collection of temperature sensor
    type:
      string:
        accessMode: ReadWrite
        defaultValue: OFF
  propertyVisitors:
  - propertyName: temperature
    modbus:
      register: CoilRegister
      offset: 2
      limit: 1
      scale: 1.0
      isSwap: true
      isRegisterSwap: true
  - propertyName: temperature-enable
    modbus:
      register: DiscreteInputRegister
      offset: 3
      limit: 1
      scale: 1.0
      isSwap: true
      isRegisterSwap: true
```
这个简单的模板描述了两个属性，一个是传感器`temperature`另一个是开关`temperature_enable`,传感器设置了温度最大值为100，开关控制数据是否上报

propertyVisitors描述了这两个属性的访问，这里用的是 `modbus`协议

device: 这是一个具体的设备实例，这个描述是要用到模板资源的

简单的例子：
```yaml
apiVersion: devices.kubeedge.io/v1alpha1
kind: Device
metadata:
  name: sensor-tag01
  labels:
    description: 'TI Simplelink SensorTag 2.0 with Bluetooth 4.0'
    manufacturer: 'Texas Instruments'
    model: CC2650
spec:
  deviceModelRef:
    # 这里引用了模板
    name: sensor-tag-model
  protocol:
    modbus:
      rtu:
        serialPort: '1'
        baudRate: 115200
        dataBits: 8
        parity: even
        stopBits: 1
        slaveID: 1
  nodeSelector:
    nodeSelectorTerms:
    - matchExpressions:
      - key: ''
        operator: In
        values:
        - node1
status:
  twins:
    - propertyName: temperature-enable
      reported:
        metadata:
          timestamp: '1550049403598'
          type: string
        value: OFF
      desired:
        metadata:
          timestamp: '1550049403598'
          type: string
        value: OFF
```

下行控制

![device-update-cloud-edge](/img/cloud_native/kubeedge/device-updates-cloud-edge.png)

这个下行控制其实就是描述了通过控制device资源来控制设备的流程，图中有个关键的组件是devicetwin，这里保存了设备的完整信息，和云端保持同步。

上行控制

![device-updates-edge-cloud](/img/cloud_native/kubeedge/device-updates-edge-cloud.png)

上行控制同样的涉及到边缘侧的几个组件，其实这两张图就能大概看出了devicetwin和eventbus的功能：devicetwin保存设备信息，eventbus是devicetwin和mapper的桥梁

### edge

**edgehub**

主要负责和cloudhub交互，同时保持心跳，从架构图中可以看到同时和边缘节点内部组件交互，分别是matamanager、devicetwin、eventbus、servicebus

**devicetwin**

很重要的一个组件，负责同步数据库数据以及子模块管理，devicetwin分为四个子模块

- Membership Module: 管理设备和节点的关系
- Twin Moudle: 设备孪生属性的更新同步
- Communication Moudle: 和其他组件通讯
- Device Moudle: 设备静态属性属性更新同步

这里涉及到两个属性，孪生属性和静态属性，静态属性指的是一些不会改变的源数据，如mac、序列号、标识符等，孪生属性指的是设备的动态状态，如灯的开关

**eventbus**

这个组件是负责组件和应用之间mqtt消息收发，订阅如下topic
```
- $hw/events/upload/#
- SYS/dis/upload_records
- SYS/dis/upload_records/+
- $hw/event/node/+/membership/get
- $hw/event/node/+/membership/get/+
- $hw/events/device/+/state/update
- $hw/events/device/+/state/update/+
- $hw/event/device/+/twin/+
```

**servicebus**

和eventbus类似，他负责组件和应用之间的http消息收发

**metamanager**

这个组件是edged和edgehub之间的处理器，其实就是对数据库元数据进行增删改查（本机维护了一个SQLite数据库）

增

![insert](/img/cloud_native/kubeedge/meta-insert.png)

改

![update](/img/cloud_native/kubeedge/meta-update.png)

**edged**

edged看做是一个简化版的kubelet负责pod生命周期的管理，并实现了和CRI，Volume，ConfigMap，Secret等功能的对接。

**mapper**

图中最后一个组件是mapper, 这是设备驱动统一管理引擎，类似于CRI和kubernetes，CRI是kubernetes底层接口和容器引擎打交道，而mapper是开放接口和方便不同设备接入kubeedge,目前实现的类型有
- bluetooth mapper
- modbus mapper
- Opcua mapper

其他类型需要自己实现，[指南](https://github.com/kubeedge/mappers-go/blob/main/docs/UserGuideofCustomizedMapper.md)

## 云边消息交互图
![云边消息](/img/cloud_native/kubeedge/router.png)

## 总结

本文主要对边缘计算概念和kubeedge组件做了简单的知识总结，详细的使用和搭建还有很多内容需要学习

## 参考资料
- https://www.redhat.com/zh/topics/edge-computing/what-is-edge-computing
- https://kubeedge.io/en/docs/
- https://zhuanlan.zhihu.com/p/350335104