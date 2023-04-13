---
title: "chatgpt学习记录-k8s apiserver"   
author: "Kingram"  
date: 2023-04-13   
lastmod: 2022-04-13

tags: [  
"cloud-native",
"chatgpt_learning"
]
---

## 介绍kubernetes apiserver 概念、原理、调用流程

Kubernetes 是一个容器编排平台，它可以帮助我们管理容器应用程序的生命周期。Kubernetes 通过一组 API 对外提供服务，并通过集群中的不同组件之间的通信来保障应用程序的顺利运行。其中，ApiServer 是 Kubernetes 的核心组件之一，它提供了 Kubernetes API 的访问点，并负责对所有资源对象的操作进行授权和验证，例如创建、更新、删除、查询等操作。

ApiServer 的工作原理如下：

1. ApiServer 会从 etcd 中读取全部的 Kubernetes 资源定义文件，包括 Pod、Service、ReplicationController、Deployment 等；
2. 客户端向 ApiServer 发送请求，ApiServer 根据 API 版本和资源类型路由请求到对应的 API 动词处理器，例如创建资源的 POST 请求将路由到 create API 处理器函数；
3. API 处理器进行基本的请求验证，校验访问权限，并发出对 etcd 数据库的 CRUD 请求；
4. etcd 响应 API 处理器的请求，并进行数据的 CURD 操作；
5. ApiServer 经过 API 版本检查，数据资源分类，数据解析等处理后，将最终结果返回给客户端。

调用流程如下：

客户端 -> 连接到 ApiServer -> 访问 Kubernetes API -> ApiServer 根据请求路由到对应 API 动词处理器 -> 处理器进行验证和授权 -> 处理器发出对 etcd 的请求 -> etcd 响应处理器的请求并进行 CRUD 操作 -> ApiServer 进行数据解析处理，返回给客户端。

总而言之，ApiServer 可以看成一个 HTTP 服务器，通过 OpenAPI 来管理所有的 Kubernetes 资源，并提供了对外的 RESTful API 接口，使得我们可以通过 RESTful 协议对 kubernetes 进行操作和管理。

## 客户端请求到apiserver后经历哪些流程？

客户端请求到 ApiServer 的过程大致如下：

1. 客户端发送请求，请求发送到 kube-apiserver 监听的端口（默认为 6443），请求可以是 RESTful 协议，也可以是 gRPC 协议；
2. kube-apiserver 首先进行 TLS 认证和授权，根据不同的认证方式来决定是否放行这个请求；
3. 接着 kube-apiserver 把请求路由到对应的 API 版本、资源类型等信息处理器上；
4. 进入处理器之前，kube-apiserver 还需要进行资源访问控制、身份认证等等步骤；
5. 处理器通过 codegen 工具自动生成，显示了各个版本和资源的所有 API。其中，每个 API 都有各种方法，如 GET、POST、PUT、PATCH、DELETE 等，用于操作、创建、修改和删除 Kubernetes cluster 里的资源对象；
6. 请求的处理器会首先解析 RESTful URL，识别请求的 HTTP 方法（GET、POST、PUT、PATCH、DELETE 等），并验证请求参数的正确性；
7. 处理器把请求的参数转换成访问 etcd 的 key 和 value，生成 etcd 请求后，发到 etcd 具体的 etcd 副本实例处理，从而读取或写入相应的数据；
8. 处理器处理完请求并成功返回结果，kube-apiserver 将结果返回给客户端。

以上并不是一个标准的流程，因为不同的请求和资源类型，有着不同的处理过程，但是整个流程的框架是通用的。而在这个流程中，ApiServer 起到了路由转发、权限控制和数据交互的作用。

## 上面第七条有代码示例吗？


这里提供一段使用 Go 语言访问 Kubernetes API 的代码片段，用于对比理解：

```go
import (
    "fmt"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/rest"
)

func main() {
    // 创建 Kubernetes 的 RESTful 配置信息
    config, err := rest.InClusterConfig()
    if err != nil {
        panic(err.Error())
    }

    // 创建 Kubernetes Clientset
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err.Error())
    }

    // 获取 Pod 资源列表
    pods, err := clientset.CoreV1().Pods("").List(context.Background(), metav1.ListOptions{})
    if err != nil {
        panic(err)
    }

    // 将获取到的列表打印出来
    for _, pod := range pods.Items {
        fmt.Printf("Pod: %s, Namespace: %s\n", pod.GetName(), pod.GetNamespace())
    }
}
```

此代码段可以访问 Kubernetes 中的 Pod 资源列表。在代码运行过程中，首先创建了 Kubernetes REST 配置信息，然后使用此配置创建了 Kubernetes 的 clientset，最后通过 clientset 来查询 Pod 资源列表。在查询 Pod 资源列表的过程中，实际上是用了 Go clientset 中定义好的 RESTful 接口来访问 kube-apiserver，然后 kube-apiserver 再通过具体的代码进行处理。

## 我需要的是第7条（处理器把请求的参数转换成访问 etcd 的 key 和 value，生成 etcd 请求后，发到 etcd 具体的 etcd 副本实例处理，从而读取或写入相应的数据 ） 这块的源码处理示例

处理器把请求的参数转换为 etcd key 和 value 真正的代码实现是在 k8s.io/apiserver/pkg/storage/storage_backend.go 中的 RESTStorage 模块中。这里是一个简化版的代码示例，可以用来对这个过程有进一步的理解：

```go
// 通过 RESTStorage 接口，资源管理器（RESTful 资源）和 etcd 交互
type RESTStorage interface {
    // 获取资源列表（List），支持分页（params 获取查询参数）
    List(ctx context.Context, params map[string]string) (runtime.Object, error)
    // 获取一个指定资源（Get）
    Get(ctx context.Context, name string, options *metav1.GetOptions) (runtime.Object, error)
    // 创建资源（Create）
    Create(ctx context.Context, obj runtime.Object, createValidation validation.ValidateFunc, options *metav1.CreateOptions) (runtime.Object, error)
    // 根据特定的字段来更新资源（Update，通过基于名称或者标签等信息的更改）
    Update(ctx context.Context, name string, objInfo rest.UpdatedObjectInfo, createValidation validation.ValidateFunc, opts metav1.UpdateOptions) (runtime.Object, bool, error)
    // 删除一个指定资源（Delete）
    Delete(ctx context.Context, name string, options *metav1.DeleteOptions) (runtime.Object, error) 
    // 合并、替代资源（Patch）
    Patch(ctx context.Context, name string, patchBytes []byte, options metav1.PatchOptions, subresource string) (runtime.Object, error)
    // 设置资源的状态（UpdateStatus）
    UpdateStatus(ctx context.Context, obj runtime.Object, updateValidation validation.ValidateUpdateFunc, opts metav1.UpdateOptions) (runtime.Object, bool, error)
}
```

以上是 RESTStorage 中的一些方法定义，用来向客户端暴露资源的 CRUD（创建、读、更新、删除）操作。这里提供一个简单的示例，展示了 RESTful 资源（Pod，Deployment，Service）通过 RESTStorage 接口与 etcd 交互的过程，主要体现在 List 方法中：

```go
func (s *storage) List(ctx context.Context, params map[string]string) (runtime.Object, error) {
    // 从 URL 参数中解析作为 selector 的过滤条件，如果没有则返回全部资源
    selector, err := rest.ParseSelectorParam(params[metav1.LabelSelectorQueryParam])
    if err != nil {
        return nil, err
    }

    // 将参数转换成 resourceVersion，方便之后的 etcd 操作
    resourceVersion := params[resourcetree.ResourceVersionQueryParam]
    if len(resourceVersion) == 0 {
        resourceVersion = "0"
    }
    version, err := strconv.ParseUint(resourceVersion, 10, 64)
    if err != nil {
        return nil, fmt.Errorf("invalid resource version: %v", err)
    }

    // 将 selector 和 resourceVersion 合并成 etcd 中的 key
    helper := s.store.Helper
    b := storage.NewListActionBase(s.typeName, s.store.Name(), version, helper.Namespace(), selector)
    // 依据 key 查询 etcd，获取 Pod，Deployment 和 Service 列表
    list, err := s.store.List(ctx, b)
    if err != nil {
        return nil, err
    }

    // 创建返回对象 List
    return &unstructured.UnstructuredList{Items: list.Items}, err
}
```

在这个示例代码中，List 方法首先从 URL 中获取 selector 参数，然后使用此参数和 resourceVersion 参数生成 etcd 的查询 key。接着，通过 list 接口，将 key 发送给 etcd，获取 Pod，Deployment 和 Service 列表。将其转换为 UnstructuredList 对象格式返回。注意此处的 etcd key 是由资源的标识信息、版本信息和 Selector 组成的。

## 上述中resourceVersion是什么？

`resourceVersion` 是 Kubernetes 对于每个资源对象的一个版本标识符。在 Kubernetes 中，每个 API 对象都有一个 `metadata.resourceVersion` 字段，它代表该对象的当前版本。每次对这个对象进行更新时，该版本号都会被更新。

Kubernetes 中使用 `resourceVersion` 来控制对象的并发修改。当多个客户端同时尝试更新同一个对象时，只有最后更新的客户端能够成功，而其他客户端的更新请求将被打回并得到一个错误响应。

在客户端访问 Kubernetes API 时，通常会对资源对象列表的查询和管理涉及到 `resourceVersion` 参数。对于 `List` 接口，`resourceVersion` 参数指示所请求的资源应该在哪个版本之后返回（这会影响到 `Watch` 接口是否应该看到所请求的资源）。对于其他写能力接口，`resourceVersion` 指定想要更新的资源的版本号。

所以，在实际使用的时候，可以通过设置 `resourceVersion` 来帮助同步更新某个资源对象的数据。

## etcd中的key有哪些部分组成？

在 Kubernetes 中，etcd key 是按照一定的规则组成的，可以分为以下几个部分：

1. `/registry`: 这是 Kubernetes 存储的根目录，所有的 API 对象都存放在这个目录下。

2. `/api/<group>/<version>`： `group` 表示资源所属的 API 组， `version` 表示 API 的版本。例如，Deployment 资源的 API group 是 `apps`，API 版本是 `v1`，因此这个部分就是 `/api/apps/v1`。

3. `/<resource>`： `resource` 表示资源类型， Deployment 资源的类型是 `deployments`，Service 资源的类型是 `services`。

4. `/<namespace>`: `namespace` 表示 Kubernetes 的命名空间。如果 API 对象不是命名空间作用域的，此处为空。

5. `/<name>`： `name` 表示 API 对象的名称，例如 Pod 的名称，Service 的名称等。

因此，一个完整的 etcd key 的格式如下：

```
/registry/<api>/<group>/<version>/<resource>/<namespace>/<name>
```

比如，一个名为 test-app 的 Deployment 资源在名为 default 的命名空间中的 etcd key 如下：

```
/registry/api/apps/v1/deployments/default/test-app
```

这种设计可以很好地管理和访问 Kubernetes 中的资源对象，并且可以通过层级关系进行快速查找和访问。

## kube-apiserver 在客户端请求过来时如何进行资源访问控制、身份认证等步骤，详细说明这个流程？

kube-apiserver 在客户端请求过来时会经过一系列的资源访问控制和身份认证等步骤。下面是这个流程的详细说明：

1. 首先，客户端会向 kube-apiserver 发送一条 HTTP 请求，请求的内容包括访问的 API 路径、HTTP 方法、请求头和请求正文等信息。

2. kube-apiserver 接收到 HTTP 请求后，会先进行身份认证。身份认证的过程主要包括以下几个步骤：

   - 如果请求的是匿名访问，则不进行身份认证；
   
   - 如果请求的是 TLS 客户端认证，则 kube-apiserver 会根据请求中的客户端证书进行身份认证；
   
   - 如果请求的是基于令牌的认证（Token-based Authentication），则 kube-apiserver 会从请求头或请求参数中读取认证令牌，并进行验证；
   
   - 如果请求的是基于用户名密码的认证（Password-based Authentication），则 kube-apiserver 会从请求头或请求参数中读取用户名和密码，并进行验证。

3. 身份认证成功后，kube-apiserver 会进入资源访问控制阶段。资源访问控制主要包括以下几个步骤：

   - 首先，kube-apiserver 会检查请求的 API 路径和 HTTP 方法是否被授权。如果被授权，则继续执行下一步，否则返回状态码 403（Forbidden）；
   
   - 然后，kube-apiserver 会根据请求中的数据，进行各种细粒度的访问控制。比如，kube-apiserver 会检查请求中的命名空间（如果有），是否与客户端拥有的命名空间权限相符。kube-apiserver 还会检查客户端是否有执行该操作的权限（比如管理 Pod 的权限）；
   
   - 最后，kube-apiserver 会将请求转发给相应的资源对象处理器（handler），完成请求的操作。如果请求成功，kube-apiserver 将返回状态码 200 和操作结果。如果请求失败，kube-apiserver 将返回状态码 4xx 或 5xx，并携带错误信息。

这就是 kube-apiserver 处理客户端请求的大致流程。它通过身份认证和资源访问控制等机制，确保了 Kubernetes 集群的安全和稳定。

## kube-apiserver 根据什么判断客户端是否有操作该命名空间的权限和管理pod的权限？

kube-apiserver 判断客户端是否有操作该命名空间的权限和管理 Pod 的权限的机制主要是通过 Kubernetes 基础设施中的几个组件来实现的。

- 访问控制列表（ACL）: 访问控制列表是 Kubernetes 中的一个核心资源对象，它定义了可以访问 API 对象的用户、组、角色和服务账户。kube-apiserver 会根据用户、组、角色和服务账户的 ACL 判断其是否有操作该命名空间的权限和管理 Pod 的权限。

- 制动角色绑定（ARB）: 授权角色绑定是 Kubernetes 中的另一个核心资源对象，它将命名空间中的角色与 Kubernetes 中的用户、组和服务账户进行绑定。kube-apiserver 会根据角色绑定判断其是否有操作该命名空间的权限和管理 Pod 的权限。

- 控制器：控制器是 Kubernetes 中的一个核心组件，它负责将期望状态与实际状态进行比较，并采取必要的措施将它们保持一致。kube-apiserver 会与控制器交互，确保客户端在操作 Pod 时，控制器对该命名空间的操作是否合法。

总的来说，kube-apiserver 通过访问控制列表、角色绑定和控制器等机制来判断客户端是否有操作该命名空间的权限和管理 Pod 的权限。这种机制可以很好地保证 Kubernetes 集群的安全和稳定。

## 上述组件涉及到哪些资源对象，分别详细介绍下

访问控制列表（ACL）、授权角色绑定（ARB）和控制器都是 Kubernetes 中的重要组件。它们分别涉及到 Kubernetes 中的资源对象，具体介绍如下：

1. 访问控制列表（ACL）

访问控制列表是 Kubernetes 中的一个核心资源对象，它定义了可以访问 API 对象的用户、组、角色和服务账户。 ACL 包括以下几个重要的资源对象：

- ServiceAccount: ServiceAccount 是 Kubernetes 集群中与某个命名空间相关联的一种资源对象，可以用于访问 API 服务器和容器中使用的其他服务。

- Role: Role 定义了一组规则，这些规则用于判断哪些用户、组和 ServiceAccount 有权访问该命名空间下的资源对象。

- RoleBinding: RoleBinding 用于将 Role 与用户、组和 ServiceAccount 进行关联。RoleBinding 根据 Role 中定义的规则来授权特定的操作。例如，RoleBinding 可以将一个 Role 与具有一定权限的 ServiceAccount 关联起来，这样 ServiceAccount 就可以进行一定级别的资源访问。

2. 授权角色绑定（ARB）

授权角色绑定是 Kubernetes 中的另一个核心资源对象，它将命名空间中的角色与 Kubernetes 中的用户、组和服务账户进行关联。ARB 包括以下几个重要的资源对象：

- Role: Role 定义了一组规则，这些规则用于判断哪些用户、组和 ServiceAccount 有权访问该命名空间下的资源对象。

- ClusterRole: ClusterRole 是一种与命名空间无关的角色，它定义了可以在整个集群中使用的一组规则。

- RoleBinding: RoleBinding 用于将 Role 与用户、组和 ServiceAccount 进行关联，根据 Role 中定义的规则来授权特定的操作。

- ClusterRoleBinding: ClusterRoleBinding 是将 ClusterRole 与用户、组和 ServiceAccount 进行关联，根据 ClusterRole 中定义的规则来授权特定的操作。

3. 控制器

Kubernetes 中的控制器是一种实现了某种声明式 API，用于控制一组运行 Pod 的控制器。控制器利用声明式 API 机制，将期望状态与实际状态进行比较，并采取必要的措施将它们保持一致。Kubernetes 中的控制器包括以下几个资源对象：

- ReplicaSet: ReplicaSet 是 Kubernetes 中的一种资源对象，它确保恰好有指定数量的 Pod 保存工作负载的副本。

- Deployment: Deployment 是 Kubernetes 中的一种资源对象，它提供了一个声明式 API，用于协调 Pod、ReplicaSet 和 Service 对象。

- StatefulSet: StatefulSet 是 Kubernetes 中的一种资源对象，它与 ReplicaSet 相似，但具有管理有状态应用程序的能力，它可以保证同一 Pod 的状态在不同节点之间转移时不变。

- DaemonSet: DaemonSet 是 Kubernetes 中的一种资源对象，它确保在每个节点上运行一个 Pod 实例，保证在整个集群中具有相同的副本数。

总的来说，访问控制列表、授权角色绑定和控制器是 Kubernetes 中非常重要的组件，它们都涉及到不同类型的资源对象，用于保护 Kubernetes 集群的安全和稳定性。

## 看起来在kubernetes中访问控制列表（ACL）和授权角色绑定（ARB）组件的功能类似，他们具体有什么区别？

访问控制列表（ACL）和授权角色绑定（ARB）在 Kubernetes 集群中都是用于控制访问和授权的重要组件。它们之间的主要区别在于作用对象和作用范围。

ACL 主要作用于命名空间，针对命名空间内的所有资源定义权限，如限制用户操作某些资源的权限以及用户访问某些资源的范围等等。

ARB 则更加灵活，它也可以绑定到命名空间上，但它主要的约束作用是在某些资源与用户或服务帐户之间的关联关系上。ARB 可以将某种资源类型关联到某个服务帐户或某个用户上，并且还允许将角色绑定到集群范围上，从而允许这些角色在整个集群内进行授权操作。

另外，ARB 还支持 roleRef、subjects、namespace、name 等各种属性，可以更加细致、精确地定义角色授权的方式。

综合来说，ACL 更加专注于固定范围内的访问控制，主要针对命名空间内的资源定义权限；而 ARB 在定义权限的同时，还考虑到资源与用户、服务帐户之间的关联关系，更加灵活，可对集群范围都进行授权，主要用于控制角色与用户及服务帐户之间的关联关系。