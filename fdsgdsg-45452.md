# 方法：使用循环实现，**不使用切片 [::-1]**
思路：从字符串最后一个下标向前遍历，逐个取出字符拼接成反转字符串
```python
# 获取用户输入
s = input("请输入字符串：")
reversed_str = ""

# 从最后一位索引向前循环
i = len(s) - 1
while i >= 0:
    reversed_str += s[i]
    i -= 1

# 输出结果
print("反转后的字符串：", reversed_str)
```

## 运行示例
```
请输入字符串：python
反转后的字符串：nohtyp
```

## 另一种 for 循环版本（同样不用切片）
```python
s = input("请输入字符串：")
res = ""
# range 倒序生成索引
for i in range(len(s)-1, -1, -1):
    res += s[i]
print("反转后的字符串：", res)
```


```







https://github.com/cvdfsl9002/hmhgmh-45645/commit/4c57810ad1ffdc399b0e6bf5b2c0a3dc68e6356a
https://github.com/cvdfsl9002/hmhgmh-45645/commit/d17eb644719ddf70fe2e958504f5468e0e75af38
https://github.com/cvdfsl9002/hmhgmh-45645/commit/f543547d8fe0458409a58f3ef15cd357fce2d2ec
https://github.com/cvdfsl9002/hmhgmh-45645/commit/d6760185d9effdc4c32b1cd5db7f378949ce6006
https://github.com/cvdfsl9002/hmhgmh-45645/commit/fec58d93cfeb7e7499ae788968b3a28eb0bc7aef
https://github.com/cvdfsl9002/hmhgmh-45645/commit/c7d5b4fb143c207160daed3ed003cadfd9039d01
https://github.com/cvdfsl9002/hmhgmh-45645/commit/881e8e10eeccec933fa1e7075e3d17e7851cfdaa
https://github.com/cvdfsl9002/hmhgmh-45645/commit/c1276fe0fa97fdb48e23c76fbbbc784d1d6c4638
https://github.com/cvdfsl9002/hmhgmh-45645/commit/6f88a3c356517420c368a9213f7a6400ac9c1d23
https://github.com/cvdfsl9002/hmhgmh-45645/commit/ad6f675dae2e5544623cc5047f69096a38a78936
https://github.com/cvdfsl9002/hmhgmh-45645/commit/2cab0f77d6c49b970e19a9b887350480538d5d2b
https://github.com/cvdfsl9002/hmhgmh-45645/commit/6f65ab273143a572bb00728820a9473def7f08fc
https://github.com/cvdfsl9002/hmhgmh-45645/commit/2cedac490a3be9f61d085adaa9980abbc8fb2938
https://github.com/cvdfsl9002/hmhgmh-45645/commit/dbd2f0a191864f233e7aaa42ae9dd7115911c710
https://github.com/cvdfsl9002/hmhgmh-45645/commit/64e10a2a7b28933ff2df1b06a0512c31767a2203
https://github.com/cvdfsl9002/hmhgmh-45645/commit/7686534d0fa35cac2e6364b09623f9af7ba08dc2
https://github.com/cvdfsl9002/hmhgmh-45645/commit/748894f04e46c73129b1c30f0e11b81a0dbe8703
https://github.com/cvdfsl9002/hmhgmh-45645/commit/e62318c163798c1b2343a4af7f7f54ade596df4e
https://github.com/cvdfsl9002/hmhgmh-45645/commit/e4e5f9fe5f0a5e26bb90831dfcabb72c2595ad8a
https://github.com/cvdfsl9002/hmhgmh-45645/commit/8688bd9c16d67efc79aec9eeb5617384515008e3
https://github.com/cvdfsl9002/hmhgmh-45645/commit/0811536a17803141d37e0db827ff4f883d44e7cd
https://github.com/cvdfsl9002/hmhgmh-45645/commit/818f64b9980ace5bd267db4c8475be1442f161f5
https://github.com/cvdfsl9002/hmhgmh-45645/commit/577635bc9b6acea44fa1a2ae999318ce6ce8c575





# K8s Helm 部署 ingress-nginx Ingress Controller 完整实操（环境准备→业务访问）

> 
> 说明：本文使用 **ingress-nginx（社区Nginx Ingress控制器）**，不是NGINX官方商业版；区分：**Ingress资源是路由规则，Ingress Controller才是真正的反向代理Pod**，必须先部署控制器，Ingress规则才能生效。
> 集群环境：Kubernetes 1.24+，Helm 3，自建K8s/云厂商集群均可；自建集群Service默认只能用NodePort，云厂商可以LoadBalancer。

## 一、环境准备

### 1. 前置检查

```
# 1. 确认kubectl连通集群
kubectl get nodes

# 2. 确认helm版本（必须helm3，helm2废弃）
helm version
```

要求：

- K8s集群正常，节点Ready
- Helm3已安装
- 集群网络插件正常（Calico/Flannel）
- 可以拉取镜像（国内环境提前准备镜像加速，否则ingress镜像拉取超时）

### 2. 添加 ingress-nginx Helm仓库

```
# 添加仓库
helm repo add ingress-nginx [https://kubernetes.github.io/ingress-nginx](https://kubernetes.github.io/ingress-nginx)

# 更新仓库索引
helm repo update

# 搜索chart
helm search repo ingress-nginx
```

> 
> 国内网络访问github.io缓慢：可以使用镜像仓库或者提前helm pull下载包离线安装

## 二、Helm 安装 ingress-nginx 控制器

### 方式1：极简安装（快速测试）

```
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --repo [https://kubernetes.github.io/ingress-nginx](https://kubernetes.github.io/ingress-nginx) \
  --namespace ingress-nginx --create-namespace
```

- `upgrade --install`：幂等命令，不存在则安装，存在则升级
- `--create-namespace`：自动创建`ingress-nginx`命名空间

### 方式2：自定义参数安装（推荐生产，可导出values.yaml微调）

```
# 导出默认配置文件，修改后安装
helm show values ingress-nginx/ingress-nginx --repo [https://kubernetes.github.io/ingress-nginx](https://kubernetes.github.io/ingress-nginx) > values.yaml

# 编辑values.yaml，常见修改点
# controller.replicaCount=2 副本数
# controller.service.type：自建集群用NodePort；公有云用LoadBalancer
# resources 资源限制、nodeSelector、tolerations污点容忍

# 使用自定义values安装
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --repo [https://kubernetes.github.io/ingress-nginx](https://kubernetes.github.io/ingress-nginx) \
  --namespace ingress-nginx --create-namespace \
  -f values.yaml
```

> 
> 自建物理机/虚拟机集群：`controller.service.type=NodePort`
> 公有云EKS/ACK/AKS：`controller.service.type=LoadBalancer`，云厂商自动分配公网IP

## 三、验证 ingress-nginx 控制器部署状态

```
# 查看命名空间下Pod
kubectl get pods -n ingress-nginx -w

# 查看Service
kubectl get svc -n ingress-nginx
```

输出示例（自建集群NodePort）：

```
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.96.xxx.xxx   <none>        80:30080/TCP,443:30443/TCP   2m
ingress-nginx-controller-admission   ClusterIP   10.96.xxx.xxx   <none>        443/TCP                      2m
```

- Pod状态必须是`Running`；镜像拉取慢会处于ContainerCreating，耐心等待或换国内镜像
- 记录：**NodePort端口 80→30080，443→30443**，后续外部访问节点IP+该端口

查看helm发布：

```
helm list -n ingress-nginx
```

### 查看控制器日志（排错用）

```
kubectl logs -f deployment/ingress-nginx-controller -n ingress-nginx
```

## 四、部署测试业务（Deployment + ClusterIP Service）

准备`demo-app.yaml`，创建一个简单nginx业务：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-nginx
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-nginx
  template:
    metadata:
      labels:
        app: demo-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: demo-nginx-svc
  namespace: default
spec:
  selector:
    app: demo-nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

应用资源：

```
kubectl apply -f demo-app.yaml

# 验证
kubectl get deploy,pod,svc
```

> 
> Service类型ClusterIP，**只能集群内部访问，外部不能直接访问**，需要Ingress做七层路由。

## 五、创建 Ingress 路由规则（核心）

> 
> 重点：K8s 1.19+ 使用 `networking.k8s.io/v1` API（不再是v1beta1）
> 新建 `ingress-demo.yaml`

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: default
  annotations:
    # 指定使用 ingress-nginx 控制器（多Ingress控制器场景必须加）
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: demo.k8s.local  # 域名，自行修改
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: demo-nginx-svc
            port:
              number: 80
```

创建Ingress资源：

```
kubectl apply -f ingress-demo.yaml

# 查看Ingress
kubectl get ingress
```

输出：

```
NAME          CLASS    HOSTS           ADDRESS   PORTS   AGE
demo-ingress  nginx    demo.k8s.local            80      30s
```

> 
> 等待片刻，`ADDRESS`字段会填充Ingress Controller Service的IP（云厂商LB IP / 节点IP）

## 六、域名解析配置 & 业务访问

### 场景A：自建K8s集群（NodePort模式）

1. 本地`/etc/hosts`（Linux/Mac）或 `C:\Windows\System32\drivers\etc\hosts` 添加解析：

```
# 节点IP      域名
192.168.1.100 demo.k8s.local
```

> 
> `192.168.1.100`：任意一个K8s工作节点IP

2. 浏览器/ curl访问：

```
curl [http://demo.k8s.local:30080](http://demo.k8s.local:30080)
```

> 
> 原理：流量 → 节点IP:30080 → ingress-nginx-controller Pod → 根据host路由转发到demo-nginx-svc → 后端Pod

### 场景B：公有云LoadBalancer模式

Ingress Controller Service会拿到公网IP，直接把域名`demo.k8s.local`DNS解析到LB公网IP，直接访问标准80端口：

```
curl [http://demo.k8s.local](http://demo.k8s.local)
```

## 七、HTTPS TLS 配置（可选）

1. 生成证书secret：

```
# 自签名证书测试，生产使用可信CA证书
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=demo.k8s.local"
kubectl create secret tls demo-tls --cert=tls.crt --key=tls.key
```

2. 修改Ingress，增加tls配置：

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - demo.k8s.local
    secretName: demo-tls
  rules:
  - host: demo.k8s.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: demo-nginx-svc
            port:
              number: 80
```

应用后访问 `[https://demo.k8s.local](https://demo.k8s.local)`

## 八、常见排错清单

1. Ingress ADDRESS一直为空：Ingress Controller Pod没正常运行；检查Pod日志
2. 访问404：host域名不匹配、path写错、backend service名称/端口错误
3. 访问连接超时：安全组/防火墙没有放行NodePort/LB端口；自建集群节点防火墙
4. 403：nginx权限问题，业务镜像问题
5. 多Ingress控制器：必须指定`ingress.class`，否则路由不生效
6. 镜像拉取失败：国内环境替换values.yaml里controller镜像为国内镜像地址

## 九、卸载（清理环境）

```
# 删除ingress路由、业务
kubectl delete -f ingress-demo.yaml
kubectl delete -f demo-app.yaml

# helm卸载ingress控制器
helm uninstall ingress-nginx -n ingress-nginx

# 删除命名空间
kubectl delete ns ingress-nginx
```

