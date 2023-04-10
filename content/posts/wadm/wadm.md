---
title: "wadm oam数据流分析调研"   
author: "Kingram"  
date: 2022-12-14   
lastmod: 2022-12-14

tags: [  
    "oam",
    "wadm"
]
---

# wasmcloud架构简图
![](/img/wadm/1.png)
# wadm简介
## wadm是什么？
WasmCloud主机运行时（OTP）提供了[Actors](https://wasmcloud.dev/reference/host-runtime/actors/) 和 [Capabilities](https://wasmcloud.dev/reference/host-runtime/capabilities) 的托管和调度，wadm是一个控制平面，用来管理一系列的部署在主机运行时lattice中的应用，并监听当前应用状态。
## wadm提供的能力：

- 管理应用规范，应用规范遵循[OAM | Open Application Model Specification](https://oam.dev/)规范
- 观察应用状态，实时观察当前的应用所处状态
- 补偿措施，当需要时，向lattice 控制器接口发布指令以达到当前状态和期望状态相匹配
## wadm api-server
wadm通过NATS提供API服务，API提供了一系列的操作接口，包括部署应用，查看状态，取消部署等

- api的topic遵循以下格式

`wadm.api.{lattice-id}.{category}.{operation}.{object}`

- topic的请求和响应使用JSON编码
- api事件和应用的实时状态事件通过[CloudEvents](https://cloudevents.io/)规范发布，如果消费者需要监听可以通过以下topic

`wadm.evt.{lattice-id}`
### 获取model列表
> wadm.api.{lattice}.model.list

```json
None
```
### 获取单个model详情
> wadm.api.{lattice}.model.get.{name}

```json
{
  "version":"0.0.1"
}
```
### 发布model
> wadm.api.{lattice}.model.put

```json
 JSON serialization of OAM model
```
### 删除model
> wadm.api.{lattice}.model.del.{name}

```json
{
  "version":"0.0.0.1",
  "undeploy":"true",
  "delete_all":"true"
}
```
### 查询model版本历史
> wadm.api.{lattice}.model.versions.{name}

```json
None
```
### 部署model
> wadm.api.{lattice}.model.deploy.{name}

```json
{
  "version":"0.0.1"
}
```
### 取消部署model
> wadm.api.{lattice}.model.undeploy.{name}

```json
{
  "destructive":"true"
}
```
# OAM字段说明
![](/img/wadm/2.png)
上图为oam模型基本框架，目前wadm项目版本为`0.3.0`，已实现的`component` 和`traits`的类型及配置如下：
![](/img/wadm/3.png)
**explain** ： [petclinic](https://github.com/wasmCloud/wadm/blob/main/oam/petclinic.yaml)
# Wadm oam deploy 数据流分析
## API时序图
![](/img/wadm/4.png)
上图展示用户在调用部分api时wadm内部数据时序图，wadm依赖`redis`作为OAM数据模型的存储中间件，redis中存储了各个应用的不同版本模型，wadm与HostCore之间也借助于NATS通信。（图中步骤2.4）
应用的状态变化事件都遵循CloudEvents规范发布到NATS中指定topic（图中步骤1.4、2.5）
## Wadm与HostCore数据通信
![](/img/wadm/5.png)
如上图所示，wadm维护了一些进程和`HostCore`之间进行通信，当用户执行`deploy`动作时，会首先获取该应用对应的[deploy_monitor进程（不存在则创建）](https://github.com/wasmCloud/wadm/blob/ec3888120148c94e33ce76251d0cb735641ea6c5/wadm/lib/wadm/api/api_server.ex#L253-L264)在短暂延迟后执行reconcile，将OAM模型资源转换为对应的actor、capability列表，并推送至NATS，在创建deployment_monitor时同时[启动lattice_supervisor进程（不存在则创建）](https://github.com/wasmCloud/wadm/blob/ec3888120148c94e33ce76251d0cb735641ea6c5/wadm/lib/wadm/deployments/deployment_monitor.ex#L83-L89)监控Lattice状态变化，并处理相关消息。
各个进程的详细介绍如下：
### deploy_monitor

- 订阅`deployments:#{lattice_id}`处理状态有变动的`Lattice`对象
- 启动时将该应用的OAM模型转换为具体的`actor`、`capability`等对象列表，发送至`wasmbus.ctl.{id}.cmd.{host_id}.la`主题 ，`HostCore`订阅该主题消费消息，启动对应的`Lattice`
### lattice_supervisor
`lattice_supervisor`维护了[三个内部进程](https://github.com/wasmCloud/wadm/blob/ec3888120148c94e33ce76251d0cb735641ea6c5/wadm/lib/wadm/lattice_supervisor.ex#L62-L83)，分别为`connectionSupervisor`、`cosumerSupervisor`、`LatticeStateMonitor`相关功能如下：
#### 1、链接控制connectionSupervisor 
负责连接NATS服务，读取`config_plan`中配置
#### 2、消费控制cosumerSupervisor  

- 负责消费NATS中的消息，订阅如下topic
   - `*.wasmbus.evt.*`
   - `wasmbus.evt.*`
- 广播收到的消息，广播到topic`lattice:#{lattice_id}`
#### 3、状态监控LatticeStateMonitor
订阅`lattice:#{lattice_id}`并将收到的消息广播给`deployments:#{lattice_id}`
## 数据结构
### Lattice
Lattice数据结构体承担上下文结构，作为HostCore和wadm之间的状态变化数据流转，这是一个可复用的组件，仓库地址：
字段简介：
```erlang
  defstruct [
    :id, // Lattice_id
    :actors, // actor信息列表
    :providers, //provider信息列表
    :hosts, // Host主机信息
    :linkdefs,// 链接
    :refmap,// imageURL/OCI地址
    :instance_tracking,//实例创建时间
    :parameters, //decay时间周期
    :invocation_log,//调度日志
    :claims //claims信息
  ]
```
# WADM 程序代码启动流程
## Application
wadm start 启动Application代码块，调用Supervisor模块
```erlang
defmodule Wadm.Application do
  use Application

  def start(_type, _args) do
    Wadm.Supervisor.start_link(name: Wadm.Supervisor)
  end
end
```
## Supervisor
Supervisor init启动相关依赖组件

- Wadm.ClusterSupervisor 提供集群能力
- Wadm.HordeSupervisor 动态控制进程启动
- Wadm.HordeRegistry 进程控制注册
- Wadm.PubSub 订阅广播主题组件
- Gnat.ConnectionSupervisor 链接NATS
- Redix redis操作
- Wadm.Api.ApiServer 消费NATS
```erlang
    children = [
      {Cluster.Supervisor, [topologies, [name: Wadm.ClusterSupervisor]]},
      {Horde.DynamicSupervisor,
       [name: Wadm.HordeSupervisor, strategy: :one_for_one, members: :auto]},
      {Horde.Registry, [name: Wadm.HordeRegistry, keys: :unique, members: :auto]},
      {Phoenix.PubSub, name: Wadm.PubSub},
      Supervisor.child_spec(
        {Gnat.ConnectionSupervisor, Wadm.Api.Connection.settings_from_config(config)},
        id: :api_connection_supervisor
      ),
      Supervisor.child_spec(
        {Redix, host: config.redis.host, name: :model_store},
        id: :model_store
      ),
      Supervisor.child_spec(
        {Gnat.ConsumerSupervisor,
         %{
           connection_name: :api_nats,
           module: Wadm.Api.ApiServer,
           subscription_topics: [
             %{topic: "wadm.api.>", queue_group: "wadm_api_server"}
           ]
         }},
        id: :wadm_api_consumer_supervisor
      )
    ]

    Supervisor.init(children, strategy: :one_for_one)
```
## ApiServer
处理用户请求
```erlang
  def request(%{topic: topic, body: body}) do
    topic
    |> String.split(".")
    # wadm
    |> List.delete_at(0)
    # api
    |> List.delete_at(0)
    |> List.to_tuple()
    |> handle_request(body)
  end
```
### deploy ## deploy_monitor
启动deploymonitor
```erlang
  def init(opts) do
    Logger.debug(
      "Starting Deployment Monitor for deployment #{opts.app_spec.name} v#{opts.app_spec.version}"
    )

    PubSub.subscribe(Wadm.PubSub, "deployments:#{opts.lattice_id}")

    {:ok,
     %State{
       spec: opts.app_spec,
       lattice_id: opts.lattice_id,
       recon_state: :initializing,
       needs_reconciliation: false
     }, {:continue, :ensure_lattice_supervisor}}
  end
```
### deploy ## lattice_supervisor
启动lattice_supervisor
```cpp
def handle_continue(:ensure_lattice_supervisor, state) do
    # Make sure that there's a lattice supervisor running
    {:ok, _pid} = Wadm.LatticeSupervisor.start_lattice_supervisor(state.lattice_id)

    Process.send_after(self(), :host_ping, 1_000)
    {:noreply, state}
end
```
lattice_supervisor分别启动ConnectionSupervisor、ConsumerSupervisor、LatticeStateMonitor
```erlang
    children = [
      Supervisor.child_spec(
        {Gnat.ConnectionSupervisor, gnat_supervisor_settings},
        id: lattice_id
      ),
      Supervisor.child_spec(
        {Gnat.ConsumerSupervisor,
         %{
           connection_name: lattice_id,
           module: Wadm.LatticeEventListener,
           subscription_topics: [
             %{topic: "*.wasmbus.evt.*", queue_group: "wadmevtmon"},
             %{topic: "wasmbus.evt.*", queue_group: "wadmevtmon"}
           ]
         }},
        id: String.to_atom("evt_#{supervised_lattice_id}")
      ),
      Supervisor.child_spec(
        {Wadm.LatticeStateMonitor, supervised_lattice_id},
        id: String.to_atom("st_#{supervised_lattice_id}")
      )
    ]

    Supervisor.init(children, strategy: :one_for_one)
```
# 项目结构及文件描述
```shell
.
├── Dockerfile
├── README.md
├── config
│   ├── config.exs
│   ├── dev.exs
│   ├── prod.exs
│   └── test.exs
├── lib
│   ├── application.ex  //项目启动
│   ├── config_plan.ex  //项目配置
│   ├── supervisor.ex   //进程管理
│   ├── wadm
│   │   ├── api
│   │   │   ├── api_server.ex //处理用户请求
│   │   │   └── connection.ex //连接nats
│   │   ├── deployments
│   │   │   ├── cloud_events.ex //管理cloudevents事件
│   │   │   └── deployment_monitor.ex //deployment_monitor进程
│   │   ├── lattice_event_listener.ex //consumer_supervisor进程
│   │   ├── lattice_state_monitor.ex //lattice_state_monitor进程
│   │   ├── lattice_supervisor.ex //lattice_supervisor进程、连接nats等
│   │   ├── model
│   │   │   ├── actor_component.ex //actor component模型
│   │   │   ├── app_spec.ex // application模型
│   │   │   ├── capability_component.ex //capability component模型
│   │   │   ├── decoder.ex //map转模型封装工具
│   │   │   ├── link_definition.ex //linkdefinition模型
│   │   │   ├── spreadscaler.ex // spreadscaler 模型
│   │   │   ├── store.ex // redis操作封装
│   │   │   ├── validator.ex //校验封装
│   │   │   └── weighted_target.ex //spreadscaler weighted模型
│   │   ├── nats.ex //发送nats消息
│   │   └── reconciler
│   │       ├── app_spec.ex //application转具体的actor、capability等对象，调用traits.ex
│   │       ├── command.ex // 封装部署、等指令消息
│   │       ├── spreadscaler.ex // 匹配spreadscaler
│   │       └── traits.ex// application转具体的actor、capability等对象
│   └── wadm.ex
├── mix.exs //项目信息、依赖管理等
└── mix.lock
```

# kubernetes 部署

- 部署redis 用于存储oam 模版数据 （目前只支持无密码验证），默认0 库
- 部署wasmcloud kubernetes [链接](https://github.com/wasmCloud/wasmcloud-otp/tree/main/wasmcloud_host/chart)
- 部署 wadm [链接](https://gitlab.oneitfarm.com/deployv2_baseimages/wadm_kubernetes)
- wadm 调用演示 [链接](https://gitlab.oneitfarm.com/deployv2_baseimages/wadm_kubernetes)
