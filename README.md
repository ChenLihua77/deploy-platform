一个使用 Go 实现的多产品自动部署控制平台。工作台统一管理产品 Agent、目标节点、部署预检、正式安装、任务日志和审计记录，并通过固定步骤模板约束远程执行行为。

当前项目仍处于开发验证阶段。部署操作可能连接远程主机并执行安装步骤，只能对已授权的测试或交付节点使用。执行前请核对产品版本、安装介质、目标地址和变更授权。

功能概览


统一展示和管理多个产品部署 Agent
维护允许部署的目标节点白名单
在正式安装前执行环境和安装介质预检
对风险项进行显式提示，并要求填写继续执行原因
正式安装前进行二次确认，并校验预检任务与部署参数一致
异步执行任务并在页面实时展示状态和日志
记录节点新增、删除等管理操作的审计日志
产品 Agent 只接受结构化参数，不接受页面或 AI 提交任意 Shell 命令
节点和审计数据以权限为 0600 的本地 JSON 文件保存

产品支持状态




产品
标识
已注册能力
当前实现状态




洞鉴 X-Ray
dongjian
单机/分布式，管理节点/引擎节点
已提供工作台、预检和部署 Runtime


牧云 CloudWalker
muyun
单机/K8s，管理节点/Linux/Windows Agent
已提供单机管理端工作台、预检和部署 Runtime


全悉 T-Answer
quanxi
26.04.001 软件单机管理节点
已提供工作台、预检和部署 Runtime


谛听 D-Sensor
diting
软件管理节点及多类探针
已注册介质预检与任务 Runtime；正式安装保持安全阻断


雷池 SafeLine
safeline
23.01.014 反向代理单机版
已提供工作台、预检和部署 Runtime



各产品的设计依据、预检规则和能力边界见 docs/ 目录。
技术栈


Go 1.22+
Go 标准库 net/http

golang.org/x/crypto
原生 HTML、CSS 和 JavaScript（嵌入 Go 源码，无前端构建依赖）

项目结构


.
├── cmd/controller/       # HTTP 控制器、路由和工作台页面
├── internal/
│   ├── agent/            # 产品 Agent 接口和能力模型
│   ├── core/             # 通用任务状态
│   ├── executor/         # 受控执行器接口
│   ├── nodes/            # 节点白名单和审计存储
│   ├── registry/         # 产品 Agent 注册中心
│   └── security/         # 目标节点授权策略
├── products/             # 各产品 Agent 与 Runtime
├── docs/                 # 产品部署 Agent 设计文档
├── data/                 # 节点和审计运行数据
└── go.mod


快速启动

从源码运行


cd /Users/chenlihua/Documents/Codex/2026-08-04/he/deploy-platform
go mod download
go run ./cmd/controller


也可以先构建再运行：

go build -o deploy-controller ./cmd/controller
./deploy-controller


启动后访问：

工作台：http://127.0.0.1:18090/

健康检查：http://127.0.0.1:18090/health

产品能力接口：http://127.0.0.1:18090/api/v1/products


服务目前固定监听 127.0.0.1:18090，仅允许本机访问。
构建


go build -o deploy-controller ./cmd/controller


构建产物与当前操作系统、CPU 架构相关。在其他平台部署时，应在目标平台构建或配置交叉编译。
目标节点管理

部署 Runtime 只允许操作已加入工作台白名单的目标节点。默认只接受私网 IP：

curl -X POST http://127.0.0.1:18090/api/v1/nodes \
  -H 'Content-Type: application/json' \
  -d '{
    "ip": "192.168.10.20",
    "name": "测试节点",
    "note": "部署验证"
  }'
