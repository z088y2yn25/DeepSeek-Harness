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

# ✅ 常用 GitHub Release 自动发布工具（下载/安装地址）
## 1. GitHub CLI（gh，官方命令行，最简单）
> 官方工具，可手动/脚本创建 release、上传附件，适合 CI/自动化调用
官网：[https://cli.github.com/](https://cli.github.com/)
下载页：[https://github.com/cli/cli/releases](https://github.com/cli/cli/releases)

安装命令：
```bash
# Windows winget
winget install GitHub.cli
# macOS
brew install gh
# Ubuntu/Debian
apt install gh
```
使用：`gh release create v1.0.0 --generate-notes`

## 2. GoReleaser（最流行，二进制项目自动打包+发布）
> Go/Rust/Python 等项目，一键跨平台编译、打包、生成Changelog、上传Release资产
官网：[https://goreleaser.com/](https://goreleaser.com/)
下载页：[https://github.com/goreleaser/goreleaser/releases](https://github.com/goreleaser/goreleaser/releases)

## 3. semantic-release（JS/TS 语义化全自动发版）
> 根据 commit 自动判断版本号、生成changelog、自动创建GitHub Release，前端/Node项目首选
npm包，无单独二进制下载，项目内安装：
```bash
npm install semantic-release @semantic-release/github -D
```
仓库：Raft 共识算法凭借逻辑清晰、易于工程实现的特性
https://github.com/cvdfsl9002/hmhgmh-45645/commit/4c57810ad1ffdc399b0e6bf5b2c0a3dc68e6356a
成为分布式存储、配置中心、消息元数据集群等系统的首选共识方案
https://github.com/cvdfsl9002/hmhgmh-45645/commit/d17eb644719ddf70fe2e958504f5468e0e75af38
很多开发者对 Raft 基础选主、日志复制、持久化机制都比较熟悉，但踩坑往往集中在**集群配置变更
https://github.com/cvdfsl9002/hmhgmh-45645/commit/f543547d8fe0458409a58f3ef15cd357fce2d2ec
初学者会想当然采用“一步式变更
https://github.com/cvdfsl9002/hmhgmh-45645/commit/d6760185d9effdc4c32b1cd5db7f378949ce6006
把旧节点从集群配置里删掉
https://github.com/cvdfsl9002/hmhgmh-45645/commit/fec58d93cfeb7e7499ae788968b3a28eb0bc7aef
在网络分区、消息延迟的扰动下，极容易爆发最致命的问题
https://github.com/cvdfsl9002/hmhgmh-45645/commit/c7d5b4fb143c207160daed3ed003cadfd9039d01
两个节点同时认为自己是集群 Leader
https://github.com/cvdfsl9002/hmhgmh-45645/commit/881e8e10eeccec933fa1e7075e3d17e7851cfdaa
最终导致数据不一致，且故障回溯和修复成本极高
https://github.com/cvdfsl9002/hmhgmh-45645/commit/c1276fe0fa97fdb48e23c76fbbbc784d1d6c4638
论文给出的标准解法，就是联合共识
https://github.com/cvdfsl9002/hmhgmh-45645/commit/6f88a3c356517420c368a9213f7a6400ac9c1d23
假设变更消息只送达了 A、B、C；D、E 因为网络分区没有收到新配置
https://github.com/cvdfsl9002/hmhgmh-45645/commit/ad6f675dae2e5544623cc5047f69096a38a78936
在收到新配置的节点视角里，集群多数派
https://github.com/cvdfsl9002/hmhgmh-45645/commit/2cab0f77d6c49b970e19a9b887350480538d5d2b
而未收到配置变更的 D、E，仍然使用旧配置，旧集群多数派需要 3 票
https://github.com/cvdfsl9002/hmhgmh-45645/commit/6f65ab273143a572bb00728820a9473def7f08fc
两个 Leader 独立处理写入，日志不再收敛，分布式一致性被彻底破坏
https://github.com/cvdfsl9002/hmhgmh-45645/commit/2cedac490a3be9f61d085adaa9980abbc8fb2938
集群先提交一条配置日志，将配置切换为 C
https://github.com/cvdfsl9002/hmhgmh-45645/commit/dbd2f0a191864f233e7aaa42ae9dd7115911c710
新旧节点都还参与集群投票
https://github.com/cvdfsl9002/hmhgmh-45645/commit/64e10a2a7b28933ff2df1b06a0512c31767a2203
待迁移的老节点还保留投票权
https://github.com/cvdfsl9002/hmhgmh-45645/commit/7686534d0fa35cac2e6364b09623f9af7ba08dc2
联合配置这条日志已经被 C_joint 的多数派提交之后
https://github.com/cvdfsl9002/hmhgmh-45645/commit/748894f04e46c73129b1c30f0e11b81a0dbe8703
联合配置只是投票规则的叠加
https://github.com/cvdfsl9002/hmhgmh-45645/commit/e62318c163798c1b2343a4af7f7f54ade596df4e
前置检查：集群健康，无网络分区、无日志落后节点
https://github.com/cvdfsl9002/hmhgmh-45645/commit/e4e5f9fe5f0a5e26bb90831dfcabb72c2595ad8a
确认待下线节点E的日志已经追上 Leader
https://github.com/cvdfsl9002/hmhgmh-45645/commit/8688bd9c16d67efc79aec9eeb5617384515008e3
联合变更过程中允许并发发起多次成员变更
https://github.com/cvdfsl9002/hmhgmh-45645/commit/0811536a17803141d37e0db827ff4f883d44e7cd
下线故障节点时，跳过联合共识直接删节点
https://github.com/cvdfsl9002/hmhgmh-45645/commit/818f64b9980ace5bd267db4c8475be1442f161f5
忽略联合阶段的超时和回滚机制
https://github.com/cvdfsl9002/hmhgmh-45645/commit/577635bc9b6acea44fa1a2ae999318ce6ce8c575

## 4. release-it（轻量通用，Node生态）
> 简单易用，版本号、tag、changelog、GitHub Release一体化
```bash
npm install release-it -D
```
仓库：[https://github.com/release-it/release-it](https://github.com/release-it/release-it)

## 5. GitHub Actions（无下载！内置免费方案）
不需要下载任何本地工具，直接在仓库配置 workflow yml，**推tag自动触发构建+发布Release**，绝大多数开源项目首选。

---
# 快速选型建议
- 只是简单创建release、上传文件：**GitHub CLI（gh）**
- 编译多平台二进制（exe/linux/mac）：**GoReleaser**
- Node/前端库，自动语义版本：**semantic-release**
- 不想装本地工具：**GitHub Actions**

https://www.v-tianjin.com/user/7801
https://www.v-tianjin.com/user/7586
https://www.v-tianjin.com/user/8276
https://www.v-tianjin.com/user/9042
https://www.v-tianjin.com/user/1407
https://www.v-tianjin.com/user/2367
https://www.v-tianjin.com/user/1467
https://www.v-tianjin.com/user/9957
https://www.v-tianjin.com/user/3107
https://www.v-tianjin.com/user/9303
https://www.v-tianjin.com/user/5869
https://www.v-tianjin.com/user/9878
https://www.v-tianjin.com/user/6237
https://www.v-tianjin.com/user/5491
https://www.v-tianjin.com/user/4286
https://www.v-tianjin.com/user/9447
https://www.v-tianjin.com/user/4205
https://www.v-tianjin.com/user/4290
https://www.v-tianjin.com/user/9454
https://www.v-tianjin.com/user/3216
https://www.v-tianjin.com/user/3668
https://www.v-tianjin.com/user/2859
https://www.v-tianjin.com/user/2999
https://www.v-tianjin.com/user/8315
https://www.v-tianjin.com/user/1799
https://www.v-tianjin.com/user/9054
https://www.v-tianjin.com/user/2206
https://www.v-tianjin.com/user/8818
https://www.v-tianjin.com/user/8238
https://www.v-tianjin.com/user/1487
https://www.v-tianjin.com/user/2786
https://www.v-tianjin.com/user/7880
https://www.v-tianjin.com/user/1019
https://www.v-tianjin.com/user/6937
https://www.v-tianjin.com/user/6607
https://www.v-tianjin.com/user/7342
https://www.v-tianjin.com/user/3461
https://www.v-tianjin.com/user/5778
https://www.v-tianjin.com/user/9748
https://www.v-tianjin.com/user/1856
https://www.v-tianjin.com/user/5205
https://www.v-tianjin.com/user/5862
https://www.v-tianjin.com/user/5694
https://www.v-tianjin.com/user/5752
https://www.v-tianjin.com/user/6128
https://www.v-tianjin.com/user/9223
https://www.v-tianjin.com/user/25711
https://www.v-tianjin.com/user/6916
https://www.v-tianjin.com/user/9643
https://www.v-tianjin.com/user/52221
https://www.v-tianjin.com/user/7329
https://www.v-tianjin.com/user/2286
https://www.v-tianjin.com/user/4549
https://www.v-tianjin.com/user/6244
https://www.v-tianjin.com/user/1110
https://www.v-tianjin.com/user/68841
https://www.v-tianjin.com/user/7334
https://www.v-tianjin.com/user/6593
https://www.v-tianjin.com/user/8494
https://www.v-tianjin.com/user/9773
https://www.v-tianjin.com/user/6383
https://www.v-tianjin.com/user/88591
https://www.v-tianjin.com/user/5189
https://www.v-tianjin.com/user/3877
https://www.v-tianjin.com/user/9044
https://www.v-tianjin.com/user/42081
https://www.v-tianjin.com/user/1989
https://www.v-tianjin.com/user/1268
https://www.v-tianjin.com/user/6228
https://www.v-tianjin.com/user/5955
https://www.v-tianjin.com/user/8270
https://www.v-tianjin.com/user/8928
https://www.v-tianjin.com/user/9220
https://www.v-tianjin.com/user/2154
https://www.v-tianjin.com/user/6080
https://www.v-tianjin.com/user/5127
https://www.v-tianjin.com/user/7026
https://www.v-tianjin.com/user/4007
https://www.v-tianjin.com/user/8294
https://www.v-tianjin.com/user/1901
https://www.v-tianjin.com/user/9513
https://www.v-tianjin.com/user/9389
https://www.v-tianjin.com/user/9363
https://www.v-tianjin.com/user/8827
https://www.v-tianjin.com/user/5853
https://www.v-tianjin.com/user/7586/questions
https://www.v-tianjin.com/user/3877/follows
https://www.v-tianjin.com/user/9054/follows
https://www.v-tianjin.com/user/7329/follows
https://www.v-tianjin.com/user/5127/favorites
https://www.v-tianjin.com/user/1901/follows
https://www.v-tianjin.com/user/9363/follows
https://www.v-tianjin.com/user/42081/follows
https://www.v-tianjin.com/user/9363/questions
https://www.v-tianjin.com/user/9054/favorites
https://www.v-tianjin.com/user/3107/favorites
https://www.v-tianjin.com/user/6916/follows
https://www.v-tianjin.com/user/1467/favorites
https://www.v-tianjin.com/user/3107/follows
https://www.v-tianjin.com/user/7880/follows
https://www.v-tianjin.com/user/5752/favorites
https://www.v-tianjin.com/user/5694/favorites
https://www.v-tianjin.com/user/6607/questions
https://www.v-tianjin.com/user/2999/questions
https://www.v-tianjin.com/user/6916/favorites
https://www.v-tianjin.com/user/6937/favorites
https://www.v-tianjin.com/user/8827/questions
https://www.v-tianjin.com/user/9513/questions
https://www.v-tianjin.com/user/68841/favorites
https://www.v-tianjin.com/user/5853/follows
https://www.v-tianjin.com/user/9044/questions
https://www.v-tianjin.com/user/9447/questions
https://www.v-tianjin.com/user/7801/questions
https://www.v-tianjin.com/user/7342/follows
https://www.v-tianjin.com/user/68841/questions
https://www.v-tianjin.com/user/5955/questions
https://www.v-tianjin.com/user/9454/questions
https://www.v-tianjin.com/user/9220/follows
https://www.v-tianjin.com/user/8238/follows
https://www.v-tianjin.com/user/8294/follows
https://www.v-tianjin.com/user/8928/questions
https://www.v-tianjin.com/user/9447/follows
https://www.v-tianjin.com/user/9513/favorites
https://www.v-tianjin.com/user/8494/follows
https://www.v-tianjin.com/user/2367/favorites
https://www.v-tianjin.com/user/3668/follows
https://www.v-tianjin.com/user/1901/questions
https://www.v-tianjin.com/user/6128/questions
https://www.v-tianjin.com/user/1467/follows
https://www.v-tianjin.com/user/1268/favorites
https://www.v-tianjin.com/user/8270/favorites
https://www.v-tianjin.com/user/4205/questions
https://www.v-tianjin.com/user/8270/follows
https://www.v-tianjin.com/user/9748/questions
https://www.v-tianjin.com/user/2786/favorites
https://www.v-tianjin.com/user/9748/follows
https://www.v-tianjin.com/user/2286/questions
https://www.v-tianjin.com/user/9643/questions
https://www.v-tianjin.com/user/2286/follows
https://www.v-tianjin.com/user/9389/favorites
https://www.v-tianjin.com/user/5694/follows
https://www.v-tianjin.com/user/5189/follows
https://www.v-tianjin.com/user/3461/favorites
https://www.v-tianjin.com/user/5491/questions
https://www.v-tianjin.com/user/3877/questions
https://www.v-tianjin.com/user/7026/questions
https://www.v-tianjin.com/user/5127/questions
https://www.v-tianjin.com/user/1856/questions
https://www.v-tianjin.com/user/5853/favorites
https://www.v-tianjin.com/user/6228/follows
https://www.v-tianjin.com/user/52221/favorites
https://www.v-tianjin.com/user/2367/questions
https://www.v-tianjin.com/user/5189/favorites
https://www.v-tianjin.com/user/8238/favorites
https://www.v-tianjin.com/user/2206/favorites
https://www.v-tianjin.com/user/9773/follows
https://www.v-tianjin.com/user/8818/follows
https://www.v-tianjin.com/user/1407/questions
https://www.v-tianjin.com/user/5127/follows
https://www.v-tianjin.com/user/1268/follows
https://www.v-tianjin.com/user/5205/favorites
https://www.v-tianjin.com/user/9389/follows
https://www.v-tianjin.com/user/2206/questions
https://www.v-tianjin.com/user/5955/favorites
https://www.v-tianjin.com/user/2999/follows
https://www.v-tianjin.com/user/9773/questions
https://www.v-tianjin.com/user/2786/questions
https://www.v-tianjin.com/user/1487/favorites
https://www.v-tianjin.com/user/4007/follows
https://www.v-tianjin.com/user/9447/favorites
https://www.v-tianjin.com/user/9303/questions
https://www.v-tianjin.com/user/3461/questions
https://www.v-tianjin.com/user/7026/favorites
https://www.v-tianjin.com/user/4290/follows
https://www.v-tianjin.com/user/8827/follows
https://www.v-tianjin.com/user/9223/favorites
https://www.v-tianjin.com/user/6080/follows
https://www.v-tianjin.com/user/4205/follows
https://www.v-tianjin.com/user/9454/follows
https://www.v-tianjin.com/user/6593/questions
https://www.v-tianjin.com/user/9044/favorites
https://www.v-tianjin.com/user/3216/follows
https://www.v-tianjin.com/user/8928/favorites
https://www.v-tianjin.com/user/8270/questions
https://www.v-tianjin.com/user/6607/favorites
https://www.v-tianjin.com/user/8494/questions
https://www.v-tianjin.com/user/25711/questions
https://www.v-tianjin.com/user/1799/questions
https://www.v-tianjin.com/user/1856/follows
https://www.v-tianjin.com/user/52221/follows
https://www.v-tianjin.com/user/5491/favorites
https://www.v-tianjin.com/user/9042/questions
https://www.v-tianjin.com/user/9878/follows
https://www.v-tianjin.com/user/2859/questions
https://www.v-tianjin.com/user/1019/favorites
https://www.v-tianjin.com/user/9878/favorites
https://www.v-tianjin.com/user/4286/favorites
https://www.v-tianjin.com/user/7334/favorites
https://www.v-tianjin.com/user/6593/follows
https://www.v-tianjin.com/user/9054/questions
https://www.v-tianjin.com/user/4007/questions
https://www.v-tianjin.com/user/7342/favorites
https://www.v-tianjin.com/user/5853/questions
https://www.v-tianjin.com/user/1019/follows
https://www.v-tianjin.com/user/5869/questions
https://www.v-tianjin.com/user/6228/questions
https://www.v-tianjin.com/user/1110/questions
https://www.v-tianjin.com/user/9223/follows
https://www.v-tianjin.com/user/9042/follows
https://www.v-tianjin.com/user/7586/favorites
https://www.v-tianjin.com/user/5205/questions
https://www.v-tianjin.com/user/9957/follows
https://www.v-tianjin.com/user/8238/questions
https://www.v-tianjin.com/user/5869/follows
https://www.v-tianjin.com/user/4205/favorites
https://www.v-tianjin.com/user/1110/follows
https://www.v-tianjin.com/user/5189/questions
https://www.v-tianjin.com/user/9223/questions
https://www.v-tianjin.com/user/4549/favorites
https://www.v-tianjin.com/user/3216/questions
https://www.v-tianjin.com/user/9044/follows
https://www.v-tianjin.com/user/1989/favorites
https://www.v-tianjin.com/user/4290/questions
https://www.v-tianjin.com/user/5778/favorites
https://www.v-tianjin.com/user/1799/follows
https://www.v-tianjin.com/user/5491/follows
https://www.v-tianjin.com/user/5869/favorites
https://www.v-tianjin.com/user/1487/questions
https://www.v-tianjin.com/user/2859/follows
https://www.v-tianjin.com/user/52221/questions
https://www.v-tianjin.com/user/4549/follows
https://www.v-tianjin.com/user/9957/favorites
https://www.v-tianjin.com/user/6244/favorites
https://www.v-tianjin.com/user/7342/questions
https://www.v-tianjin.com/user/7801/follows
https://www.v-tianjin.com/user/3461/follows
https://www.v-tianjin.com/user/88591/favorites
https://www.v-tianjin.com/user/8818/questions
https://www.v-tianjin.com/user/2286/favorites
https://www.v-tianjin.com/user/25711/favorites
https://www.v-tianjin.com/user/1856/favorites
https://www.v-tianjin.com/user/2999/favorites
https://www.v-tianjin.com/user/7026/follows
https://www.v-tianjin.com/user/6080/favorites
https://www.v-tianjin.com/user/5862/follows
https://www.v-tianjin.com/user/6228/favorites
https://www.v-tianjin.com/user/6607/follows
https://www.v-tianjin.com/user/1799/favorites
https://www.v-tianjin.com/user/7801/favorites
https://www.v-tianjin.com/user/3107/questions
https://www.v-tianjin.com/user/5955/follows
https://www.v-tianjin.com/user/8928/follows
https://www.v-tianjin.com/user/7880/favorites
https://www.v-tianjin.com/user/9389/questions
https://www.v-tianjin.com/user/7334/follows
https://www.v-tianjin.com/user/2154/favorites
https://www.v-tianjin.com/user/9957/questions
https://www.v-tianjin.com/user/6244/follows
https://www.v-tianjin.com/user/68841/follows
https://www.v-tianjin.com/user/7329/questions
https://www.v-tianjin.com/user/88591/questions
https://www.v-tianjin.com/user/6593/favorites
https://www.v-tianjin.com/user/4286/questions
https://www.v-tianjin.com/user/6128/favorites
https://www.v-tianjin.com/user/6383/follows
https://www.v-tianjin.com/user/3216/favorites
https://www.v-tianjin.com/user/2154/follows
https://www.v-tianjin.com/user/1268/questions
https://www.v-tianjin.com/user/9773/favorites
https://www.v-tianjin.com/user/9363/favorites
https://www.v-tianjin.com/user/42081/questions
https://www.v-tianjin.com/user/6383/favorites
https://www.v-tianjin.com/user/9643/favorites
https://www.v-tianjin.com/user/7329/favorites
https://www.v-tianjin.com/user/1407/follows
https://www.v-tianjin.com/user/9748/favorites
https://www.v-tianjin.com/user/8276/questions
https://www.v-tianjin.com/user/1901/favorites
https://www.v-tianjin.com/user/6937/follows
https://www.v-tianjin.com/user/5694/questions
https://www.v-tianjin.com/user/3877/favorites
https://www.v-tianjin.com/user/8276/follows
https://www.v-tianjin.com/user/9220/questions
https://www.v-tianjin.com/user/8494/favorites
https://www.v-tianjin.com/user/6244/questions
https://www.v-tianjin.com/user/1019/questions
https://www.v-tianjin.com/user/9303/follows
https://www.v-tianjin.com/user/9878/questions
https://www.v-tianjin.com/user/3668/favorites
https://www.v-tianjin.com/user/2859/favorites
https://www.v-tianjin.com/user/8818/favorites
https://www.v-tianjin.com/user/1407/favorites
https://www.v-tianjin.com/user/1467/questions
https://www.v-tianjin.com/user/9513/follows
https://www.v-tianjin.com/user/5862/questions
https://www.v-tianjin.com/user/4549/questions
https://www.v-tianjin.com/user/4290/favorites
https://www.v-tianjin.com/user/6937/questions
https://www.v-tianjin.com/user/42081/favorites
https://www.v-tianjin.com/user/1989/questions
https://www.v-tianjin.com/user/6237/questions
https://www.v-tianjin.com/user/9303/favorites
https://www.v-tianjin.com/user/8315/questions
https://www.v-tianjin.com/user/6916/questions
https://www.v-tianjin.com/user/8276/favorites
https://www.v-tianjin.com/user/2786/follows
https://www.v-tianjin.com/user/1487/follows
https://www.v-tianjin.com/user/8294/favorites
https://www.v-tianjin.com/user/8315/follows
https://www.v-tianjin.com/user/7334/questions
https://www.v-tianjin.com/user/2206/follows
https://www.v-tianjin.com/user/25711/follows
https://www.v-tianjin.com/user/9454/favorites
https://www.v-tianjin.com/user/4286/follows
https://www.v-tianjin.com/user/8827/favorites
https://www.v-tianjin.com/user/5778/questions
https://www.v-tianjin.com/user/8315/favorites
https://www.v-tianjin.com/user/4007/favorites
https://www.v-tianjin.com/user/5752/follows
https://www.v-tianjin.com/user/6237/follows
https://www.v-tianjin.com/user/9042/favorites
https://www.v-tianjin.com/user/6128/follows
https://www.v-tianjin.com/user/9220/favorites
https://www.v-tianjin.com/user/1110/favorites
https://www.v-tianjin.com/user/6237/favorites
https://www.v-tianjin.com/user/5778/follows
https://www.v-tianjin.com/user/2154/questions
https://www.v-tianjin.com/user/6080/questions
https://www.v-tianjin.com/user/5205/follows
https://www.v-tianjin.com/user/7586/follows
https://www.v-tianjin.com/user/3668/questions
https://www.v-tianjin.com/user/9643/follows
https://www.v-tianjin.com/user/8294/questions
https://www.v-tianjin.com/user/5752/questions
https://www.v-tianjin.com/user/2367/follows
https://www.v-tianjin.com/user/5862/favorites
https://www.v-tianjin.com/user/88591/follows
https://www.v-tianjin.com/user/6383/questions
https://www.v-tianjin.com/user/1989/follows
https://www.v-tianjin.com/user/7880/questions
