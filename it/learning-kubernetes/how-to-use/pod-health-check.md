# Pod 健康检查

对 Pod 对健康检查可以通过两类探针来检查： LivenessProbe 和 ReadinessProbe。

## 1 LivenessProbe 

LivenessProbe ，「存活探针」，用于判断容器是否存活（running状态）。

如果 LivenessProbe 探测到容器不健康，则 kubelet 将杀掉该容器，并根据容器的重启策略做相应的处理。如果一个容器不包含 LivenessProbe ，那么 kubelet 认为该容器的 LivenessProbe 返回的值永远是 “Success“。

### 1.1 实现方式

LivenessProbe 有三种实现方式：

1. ExecAction
2. TCPSocketAction
3. HTTPGetAction

#### 1.1.1 ExecAction

在容器内部执行一个命令，如果该命名的返回码为 0，则表明容器健康。

#### 1.1.2 TCPSocketAction

通过容器的 IP 地址和端口号执行 TCP 检查，如果能够建立 TCP 连接，则表明容器健康。

#### 1.1.3 HTTPGetAction

通过容器 IP 地址、端口号及路径调用 HTTP Get 方法，如果响应的状态码大于等于 200 且小于 400，则认为容器状态健康。

### 1.2 探针参数

每种实现方式，都需要设置 initialDelaySeconds 和 timeoutSeconds 两个参数：

* initialDelaySeconds：启动容器后进行首次健康检查的等待时间，单位为 s。
* timeoutSeconds：健康检查发送请求后等待响应的超时时间，单位为 s。当超时发生时，kubelet 会认为容器已经无法提供服务，将会重启该容器。

### 1.3 实践注意事项

#### 1.3.1 应该检查什么

简易的存活探针仅仅检查了服务器是否响应。

为了更好地进行存活检查，需要将探针配置为请求特定的URL路径（例如 /healthz），并让应用从内部运行的所有重要组件执行状态检查，以确保它们都没有终止或停止响应。

要确保 /healthz HTTP 端点不需要认证，否则探测会一直失败，导致容器无限重启。

一定要检查应用程序的内部，而没有任何外部因素的影响。比如，当无法连接到后端数据库时，前端 Web 服务器的存活探针不应该返回失败；因为如果问题的原因在数据库，重启 Web 服务器容器不会解决问题。

#### 1.3.2 保持探针轻量

存活探针不应该消耗太多的计算资源，并且运行不应该花太长时间。默认情况下，探测器执行的频率相对较高，必须在一秒内执行完毕。一个过重的探针会大大减慢容器的运行。

#### 1.3.3 无须在探针中实现重试循环

探针的失败阈值是可配置的，并且通常在容器终止之前，探针必须失败多次。

即使将失败阈值设置为 1 ，Kubernetes 为了确认一次探测的失败，会尝试若干次。

## 2 ReadinessProbe 

ReadinessProbe ，「就绪探针」，用于判断容器是否启动完成（ready状态），可以接收请求。

如果 ReadinessProbe 检测到失败，则Pod的状态将被修改。Endpoint Controller 将从 Service 的 Endpoint 中删除包含该容器所在 Pod 的 Endpoint。



