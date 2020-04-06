# Pod 健康检查

对 Pod 对健康检查可以通过两类探针来检查： LivenessProbe 和 ReadinessProbe。

## 1 LivenessProbe 探针

LivenessProbe 探针，「存活探针」，用于判断容器是否存活（running状态）。

如果 LivenessProbe 探针探测到容器不健康，则 kubelet 将杀掉该容器，并根据容器的重启策略做相应的处理。如果一个容器不包含 LivenessProbe 探针，那么 kubelet 认为该容器的 LivenessProbe 探针返回的值永远是 “Success“。

LivenessProbe 有三种实现方式：

1. ExecAction
2. TCPSocketAction
3. HTTPGetAction

### 1.1 ExecAction

在容器内部执行一个命令，如果该命名的返回码为 0，则表明容器健康。

### 1.2 TCPSocketAction

通过容器的 IP 地址和端口号执行 TCP 检查，如果能够建立 TCP 连接，则表明容器健康。

### 1.3 HTTPGetAction

通过容器 IP 地址、端口号及路径调用 HTTP Get 方法，如果响应的状态码大于等于 200 且小于 400，则认为容器状态健康。

### 1.4 探针参数

每种实现方式，都需要设置 initialDelaySeconds 和 timeoutSeconds 两个参数：

* initialDelaySeconds：启动容器后进行首次健康检查的等待时间，单位为 s。
* timeoutSeconds：健康检查发送请求后等待响应的超时时间，单位为 s。当超时发生时，kubelet 会认为容器已经无法提供服务，将会重启该容器。

## 2 ReadinessProbe 探针

ReadinessProbe 探针，「就绪探针」，用于判断容器是否启动完成（ready状态），可以接收请求。

如果 ReadinessProbe 探针检测到失败，则Pod的状态将被修改。Endpoint Controller 将从 Service 的 Endpoint 中删除包含该容器所在 Pod 的 Endpoint。

