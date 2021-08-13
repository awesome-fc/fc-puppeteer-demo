## 简介

Puppeteer 可以将 Chrome 或者 Chromium 以无界面的方式运行（当然也可以运行在有界面的服务器上），然后可以通过代码控制浏览器的行为，即使是非界面的模式运行，Chrome 或 Chromium 也可以在内存中正确渲染网页的内容。

那么 Puppeteer 能做什么呢？其实有很多地方都可以受用 Puppeteer，比如：

- 生成网页截图或者 PDF
- 抓取 SPA（Single-Page Application) 进行服务器渲染（SSR）
- 高级爬虫，可以爬取大量异步渲染内容的网页
- 模拟键盘输入、表单自动提交、登录网页等，实现 UI 自动化测试
- 捕获站点的时间线，以便追踪你的网站，帮助分析网站性能问题

## 快速部署一个分布式 Puppeteer Web 应用

为了快速部署弹性高可用的 Puppeteer Web 应用，这里我们选择[函数计算](https://help.aliyun.com/product/50980.html), 有了函数计算服务，搭建一个弹性高可用应用，做的事情很简单，那就是写好业务代码，部署到函数计算，仅此而已。

使用函数计算后，系统架构图如下：

![](https://img.alicdn.com/tfs/TB1nh05YVT7gK0jSZFpXXaTkpXa-1054-608.png)

#### 体验效果

[https://1986114430573743.cn-hangzhou.fc.aliyuncs.com/2016-08-15/proxy/puppeteer-test/html2png/?url=https://www.aliyun.com/product/fc](https://1986114430573743.cn-hangzhou.fc.aliyuncs.com/2016-08-15/proxy/puppeteer-test/html2png/?url=https://www.aliyun.com/product/fc)

> PS：因为本身代码包很大，第一次请求可能会有几秒的冷启动时间，通过使用预留模式可以完全去除冷启动，由于超出本文范围，这里不再阐述。

## 快速开始

puppeteer目前可以通过s工具进行部署，可以参考[start-puppeteer](https://github.com/devsapp/start-puppeteer)

### 准备工作

1. 安装 [Funcraft](https://help.aliyun.com/document_detail/140283.html)，最简单的方式就是直接下载可执行的二进制文件。

- 安装 Funcraft 到本机。详情请参见安装 Funcraft。
- 执行 fun --version 检查安装是否成功。
- 执行 fun config 配置 Funcraft。然后按照提示依次配置 Account ID、Access Key ID、Access Key Secret、Default region name。

```bash
# fun config
Aliyun Account ID 1234xxx
Aliyun Access Key ID xxxx
Aliyun Access Key Secret xxxx
Default region name cn-xxxx
The timeout in seconds for each SDK client invoking 300
The maximum number of retries for each SDK client 5
Allow to anonynously report usage statistics to improve the tool over time? (Y/n)
```

2. 安装[Docker](https://hub.docker.com/search/?offering=community&q=&type=edition)

puppeteer 的安装，即使是在传统的 linux 机器上，也不是那么的轻松， 因为 puppeteer 本身依赖了非常多的系统库，要安装哪些系统库、如何安装这些系统库成了一个比较头痛的问题，要将这些系统库打包到函数计算的代码包中更是一个困难的问题

3. 免费开通相关的阿里云服务

- [函数计算](https://fc.console.aliyun.com)

- [文件存储](https://nasnext.console.aliyun.com)

- [容器镜像服务](https://cr.console.aliyun.com)

### nodejs8

对于 nodejs8, 函数计算命令行工具 [Funcraft](https://help.aliyun.com/document_detail/140283.html) 已经集成了 Puppeteer 的解决方案，只要 package.json 中包含了 puppeteer 依赖，然后使用 fun install -d 即可一键安装所有系统依赖。

```bash
cd nodejs8

fun install -d

fun nas sync

fun deploy -y
```

唯一的不足，Funcraft 集成 Puppeteer 的解决方案 Puppeteer 版本比较老， 可能最新的 Puppeteer 版本会有些兼容性问题。

### nodejs10

nodejs10 版本使用的是最新的 Puppeteer 版本 5.3.1，可以依靠 fun install 进行第三方依赖打包

```bash
cd nodejs10

fun install -v

fun nas sync

fun deploy -y
```

### nodejs12

nodejs12 版本使用的是最新的 Puppeteer 版本 5.3.1，可以依靠 fun install 进行第三方依赖打包

```bash
cd nodejs12

fun install -v

fun nas sync

fun deploy -y
```

### custom-container

如果您对 zip 代码包这种形式不是很满意，您可以使用函数计算的[自定义镜像功能](https://help.aliyun.com/document_detail/179368.html), 但是相比于代码包，容器镜像依赖的基础环境带来了额外的数据下载和解压的时间, 为了更好的冷启动体验，推荐您使用以下最佳实践：

- 容器镜像地址使用 VPC 网络地址 registry-vpc.Endpoint（例如 registry-vpc.cn-hangzhou.aliyuncs.com/fc-demo/helloworld:v1beta1），内网的传输带宽和稳定性都更有保证。
- 推荐使用阿里云容器镜像服务和函数计算同地域的 VPC Endpoint 获得最优的镜像拉取延时和稳定性。
- 镜像最小化，仅保存必要的依赖和代码，避免不需要的文档、数据或其他文件造成的额外延迟。
- 容器镜像配合预留实例一起使用，详情请参见[预留实例简介](https://help.aliyun.com/document_detail/138103.html)。

#### 操作

首先: Replace YOUR_ACR_IMAGE in template.yml

```bash
# 第一次 docker login 下
docker login --username=<YOUR_USER_NAME> registry.<YOUR_REGION>.aliyuncs.com -p <YOUR_USER_PWD>

# Build the Docker image
fun build --use-docker

# Deploy the function, push the image via the internet registry host (the function config uses the VPC registry for faster image pulling)
fun deploy --push-registry acr-internet
```

## FAQ

#### 在安装 puppeteer 时，Fun 都做了哪些事情？

puppeteer 本身是一个 npm 包，它的安装是非常简单的，通过 npm install 即可。这里的问题在于，puppeteer 依赖了 chromium，而 chromium 又依赖一些系统库。所以 npm install 后，还会触发下载 chromium 的操作。这里用户经常遇到的问题，主要是：

1. 由于 chromium 的体积比较大，所以经常遇到网络问题导致下载失败。
2. npm 仅仅只下载 chromium，chromium 依赖的系统库并不会自动安装。用户还需要自行查找缺失的依赖进行安装。

Fun 做的优化主要是：

1. 通过检测网络环境，对于国内用户，会帮助配置淘宝 NPM 镜像实现加速下载的效果。
2. 自动为用户安装 chromium 所缺失的依赖库。

#### Fun 是如何把大依赖部署到函数计算的？不是有代码包大小的限制吗?

基本上所有的 FaaS 为了优化函数冷启动，都会加入函数代码包大小的限制。函数计算也不例外。但是，Fun 通过内置 NAS（阿里云文件存储） 解决方案，可以一键帮用户创建、配置 NAS，并上传依赖到 NAS 上。而函数计算在运行时，可以自动从 NAS 读取到函数依赖。

为了帮助用户自动化地完成这些操作，Fun 内置了一个向导（3.1.1 版本仅支持 node，后续会支持更多，欢迎 github issue 提需求），在检测到代码体积大小超过平台限制时，会提示是否由 Fun 将其改造成 NAS 的方案，整个向导的逻辑如下：

1. 询问是否使用 Fun 来自动化的配置 NAS 管理依赖？（如果回答是，则进入向导，回答否，则继续发布流程）

2. 检测用户的 yml 中是否已经配置了 NAS

   - 如果已经配置，则提示用户选择已经配置的 NAS 存储函数依赖
   - 如果没有配置，则提示用户是否使用 NasConfig: Auto 自动创建 NAS 配置
     - 如果选择了是，则帮助用户自动配置 nas、vpc 资源。
     - 如果选择了否，则列出用户当前 NAS 控制台上已经有的 NAS 资源，让用户选择

3. 无论上面使用哪种方式，最终都会在 template.yml 生成 NAS 以及 VPC 相关的配置

4. 根据语言检测，比如 node runtime，会将 node_modules 以及 .fun/root 目录映射到 nas 目录（通过 .nas.yml 实现）

5. 自动执行 fun nas sync 帮用户把本地的依赖上传到 NAS 服务

6. 自动执行 fun deploy -y，帮用户把代码上传到函数计算

7. 提示帮助信息，对于 HTTP Trigger 的，提示函数的 Endpoint，直接打开浏览器访问即可看到效果

#### 是否可以指定 puppeteer 的版本？

可以的，只需要修改 package.json 中的 puppeteer 的版本，重新安装即可。

> Funcraft 集成 nodejs8 的 Puppeteer 的解决方案 Puppeteer 版本比较老， 可能最新的 Puppeteer 版本会有些兼容性问题，在这里建议使用 nodejs10+ 版本

#### 函数计算实例中的时区采用的 UTC，是否有办法改为北京时间？

某些网页的显示效果是和时区挂钩的，时区不同，可能会导致显示的内容有差异。使用本文介绍的方法，可以非常容易的使用 puppeteer 的最新版本，而在 puppeteer 的最新版本 2.0 提供了一个新的 API [page.emulateTimezone(timezoneId)](https://github.com/puppeteer/puppeteer/blob/v2.0.0/docs/api.md?spm=a2c6h.12873639.0.0.334842faCdFe42#pageemulatetimezonetimezoneid) , 可以非常容易的修改时区。

#### 如果 Puppeteer 后续版本更新后，依赖更多的系统依赖，本文介绍的方法还适用吗？

Fun 内置了 .so 缺失检测机制，当在本地调试运行时，会智能地根据报错识别出缺失的依赖库，然后精准地给出安装命令，可以做到一键安装。

#### 如果添加了新的依赖，如何更新？

如果添加了新的依赖，比如 node_modules 目录添加了新的依赖库，只需要重新执行 fun nas sync 进行同步即可。
如果修改了代码，只需要使用 fun deploy 重新部署即可。由于大依赖和代码通过 NAS 进行了分离，依赖通常不需要频繁变化，所以调用的频率比较低，而 fun deploy 的由于没有了大依赖，部署速度也会非常的快。

#### 示例返回的是下载文件， 能不能浏览器直接显示截图结果？

可以， 使用[自定义域名](https://help.aliyun.com/document_detail/90722.html)即可

## 参考

[Serverless 实战 —— 快速开发一个分布式 Puppeteer 网页截图服务](https://developer.aliyun.com/article/727915)
