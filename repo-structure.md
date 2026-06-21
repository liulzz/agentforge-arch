# AgentForge 智能体中台 · Monorepo 仓库目录规范

## 一、仓库组织方式

采用 **根目录 + 子模块（monorepo）** 模式，所有子产品代码统一在一个仓库中管理。

```
agentforge/                          # 根仓库
├── ai-lcap-Server/                  # 低代码平台 · 后端（Python）
├── ai-lcap-web/                     # 低代码平台 · 前端（Vue3）
├── ai-arp-Server/                   # 运行平台 · 后端（Java）
├── ai-arp-web/                      # 运行平台 · 前端（Vue3）
├── ai-aep-Server/                   # 评测平台 · 后端（Java）
├── ai-aep-web/                      # 评测平台 · 前端（Vue3）
├── ai-aeco-Server/                  # 生态平台 · 后端（Java）
├── ai-aeco-web/                     # 生态平台 · 前端（Vue3）
├── shared/                          # 共享资源
│   ├── proto/                       # gRPC / Protobuf 定义
│   ├── common-libs/                 # 公共工具库
│   └── docker/                      # 公共 Dockerfile 模板
├── deploy/                          # 部署配置
│   ├── helm/                        # Helm Charts
│   ├── docker-compose/              # 本地开发编排
│   └── k8s/                         # K8s YAML（非 Helm 场景）
├── docs/                            # 项目文档
├── scripts/                         # CI/CD 脚本
├── .github/                         # GitHub Actions
├── .gitlab-ci.yml                   # GitLab CI（可选）
├── Makefile                         # 根级构建入口
└── README.md
```

---

## 二、Java 后端工程目录（ai-arp-Server / ai-aep-Server / ai-aeco-Server）

三个 Java 后端工程结构统一，采用 **Spring Cloud 多模块 Maven 工程**。

```
ai-arp-Server/
├── pom.xml                          # 父 POM（依赖版本管理）
├── ai-arp-api/                      # API 定义模块（对外接口 + DTO）
│   ├── pom.xml
│   └── src/main/java/
│       └── com/agentforge/arp/api/
│           ├── dto/                 # 数据传输对象
│           ├── vo/                  # 视图对象
│           ├── enums/               # 枚举定义
│           └── constants/           # 常量
├── ai-arp-service/                  # 业务实现模块（核心逻辑）
│   ├── pom.xml
│   └── src/
│       ├── main/
│       │   ├── java/com/agentforge/arp/
│       │   │   ├── controller/      # REST 控制器
│       │   │   ├── service/         # 业务接口
│       │   │   │   └── impl/        # 业务实现
│       │   │   ├── manager/         # 可复用业务层（跨 service 调用）
│       │   │   ├── mapper/          # MyBatis Mapper 接口
│       │   │   ├── entity/          # 数据库实体
│       │   │   ├── config/          # 配置类
│       │   │   ├── aspect/          # AOP 切面
│       │   │   ├── listener/        # 事件监听（Kafka Consumer）
│       │   │   ├── task/            # 定时任务
│       │   │   ├── exception/       # 自定义异常
│       │   │   └── util/            # 工具类
│       │   ├── resources/
│       │   │   ├── mapper/          # MyBatis XML
│       │   │   ├── db/              # 数据库迁移脚本（Flyway）
│       │   │   ├── application.yml
│       │   │   ├── application-dev.yml
│       │   │   ├── application-prod.yml
│       │   │   └── bootstrap.yml    # Nacos 配置中心
│       │   └── Dockerfile
│       └── test/
│           ├── java/                # 单元测试
│           └── resources/           # 测试配置
├── ai-arp-common/                   # 公共模块（工具类、通用组件）
│   ├── pom.xml
│   └── src/main/java/com/agentforge/arp/common/
│       ├── response/                # 统一响应封装
│       ├── exception/               # 全局异常处理
│       ├── utils/                   # 通用工具
│       ├── annotation/              # 自定义注解
│       └── interceptor/             # 拦截器
└── README.md
```

**包名规范**：`com.agentforge.{产品缩写}.{模块}`

| 产品 | 包名前缀 |
|------|----------|
| 运行平台 | `com.agentforge.arp` |
| 评测平台 | `com.agentforge.aep` |
| 生态平台 | `com.agentforge.aeco` |

---

## 三、Python 后端工程目录（ai-lcap-Server）

低代码平台后端采用 **Python + FastAPI**，分层清晰。

```
ai-lcap-Server/
├── pyproject.toml                   # 项目配置（替代 setup.py + requirements.txt）
├── alembic/                         # 数据库迁移（Alembic）
│   ├── alembic.ini
│   ├── env.py
│   └── versions/                    # 迁移版本文件
├── app/
│   ├── __init__.py
│   ├── main.py                      # FastAPI 入口
│   ├── core/                        # 核心配置
│   │   ├── __init__.py
│   │   ├── config.py                # 配置加载（pydantic-settings）
│   │   ├── security.py              # 认证鉴权（JWT / OAuth2）
│   │   ├── events.py                # 启动/关闭事件
│   │   └── dependencies.py          # 全局依赖注入
│   ├── api/                         # API 路由层
│   │   ├── __init__.py
│   │   ├── v1/                      # API 版本化
│   │   │   ├── __init__.py
│   │   │   ├── router.py            # 路由聚合
│   │   │   ├── endpoints/           # 按资源拆分
│   │   │   │   ├── agents.py
│   │   │   │   ├── flows.py
│   │   │   │   ├── models.py
│   │   │   │   └── metadata.py
│   │   │   └── deps.py              # 路由级依赖
│   │   └── websocket/               # WebSocket 端点
│   ├── schemas/                     # Pydantic 模型（请求/响应）
│   │   ├── __init__.py
│   │   ├── agent.py
│   │   ├── flow.py
│   │   └── common.py
│   ├── services/                    # 业务逻辑层
│   │   ├── __init__.py
│   │   ├── agent_service.py
│   │   ├── flow_engine.py           # 流程执行引擎
│   │   ├── metadata_parser.py       # 元数据解析
│   │   └── model_service.py
│   ├── models/                      # SQLAlchemy ORM 模型
│   │   ├── __init__.py
│   │   ├── base.py                  # 基类（id, created_at, updated_at）
│   │   ├── agent.py
│   │   └── flow.py
│   ├── repositories/                # 数据访问层
│   │   ├── __init__.py
│   │   ├── base.py                  # 通用 CRUD
│   │   ├── agent_repo.py
│   │   └── flow_repo.py
│   ├── infrastructure/              # 基础设施集成
│   │   ├── __init__.py
│   │   ├── database.py              # 数据库连接
│   │   ├── redis.py                 # Redis 连接
│   │   ├── kafka.py                 # Kafka 生产者
│   │   └── storage.py               # MinIO / S3
│   ├── tasks/                       # 后台任务（Celery / ARQ）
│   │   ├── __init__.py
│   │   └── async_tasks.py
│   └── utils/                       # 工具函数
│       ├── __init__.py
│       ├── logger.py
│       └── helpers.py
├── tests/                           # 测试
│   ├── conftest.py                  # pytest fixtures
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── Dockerfile
├── docker-compose.yml               # 本地开发
├── Makefile                         # 常用命令
└── README.md
```

---

## 四、前端工程目录（ai-lcap-web / ai-arp-web / ai-aep-web / ai-aeco-web）

四个前端工程结构统一，采用 **Vue 3 + TypeScript + Element Plus + Vite**。

```
ai-arp-web/
├── package.json
├── tsconfig.json
├── vite.config.ts
├── .env                              # 环境变量
├── .env.development
├── .env.production
├── index.html
├── public/
│   ├── favicon.ico
│   └── logo.svg
├── src/
│   ├── main.ts                      # 入口
│   ├── App.vue
│   ├── router/                      # 路由
│   │   ├── index.ts
│   │   └── modules/                 # 按模块拆分路由
│   │       ├── agent.ts
│   │       ├── flow.ts
│   │       └── settings.ts
│   ├── stores/                      # Pinia 状态管理
│   │   ├── index.ts
│   │   ├── user.ts
│   │   ├── agent.ts
│   │   └── app.ts
│   ├── api/                         # API 请求层
│   │   ├── index.ts                 # axios 实例 + 拦截器
│   │   ├── agent.ts
│   │   ├── flow.ts
│   │   └── types/                   # API 类型定义
│   │       ├── agent.ts
│   │       └── flow.ts
│   ├── views/                       # 页面视图
│   │   ├── dashboard/
│   │   │   └── index.vue
│   │   ├── agent/
│   │   │   ├── list.vue
│   │   │   ├── detail.vue
│   │   │   └── editor.vue
│   │   ├── flow/
│   │   │   ├── list.vue
│   │   │   └── canvas.vue           # 画布编排页
│   │   └── settings/
│   │       └── index.vue
│   ├── components/                  # 公共组件
│   │   ├── layout/                  # 布局组件
│   │   │   ├── AppHeader.vue
│   │   │   ├── AppSidebar.vue
│   │   │   └── AppMain.vue
│   │   ├── common/                  # 通用组件
│   │   │   ├── SearchBar.vue
│   │   │   ├── DataTable.vue
│   │   │   └── ConfirmDialog.vue
│   │   └── business/                # 业务组件
│   │       ├── AgentCard.vue
│   │       └── FlowNode.vue
│   ├── composables/                 # 组合式函数（Hooks）
│   │   ├── useAuth.ts
│   │   ├── usePagination.ts
│   │   └── useAgent.ts
│   ├── directives/                  # 自定义指令
│   │   └── permission.ts
│   ├── styles/                      # 全局样式
│   │   ├── variables.scss
│   │   ├── reset.scss
│   │   └── mixins.scss
│   ├── utils/                       # 工具函数
│   │   ├── request.ts               # HTTP 封装
│   │   ├── auth.ts
│   │   └── helpers.ts
│   └── types/                       # 全局类型
│       ├── global.d.ts
│       └── env.d.ts
├── Dockerfile
└── README.md
```

---

## 五、共享资源目录（shared/）

```
shared/
├── proto/                           # gRPC / Protobuf 定义（跨产品通信）
│   ├── agent.proto
│   ├── flow.proto
│   └── evaluation.proto
├── common-libs/                     # 公共工具库
│   ├── java/                        # Java 公共包
│   │   └── com/agentforge/common/
│   │       ├── response/
│   │       ├── exception/
│   │       └── utils/
│   └── python/                      # Python 公共包
│       └── agentforge_common/
│           ├── response.py
│           ├── exception.py
│           └── utils.py
├── docker/                          # 公共 Dockerfile 模板
│   ├── java-base.Dockerfile
│   ├── python-base.Dockerfile
│   └── node-base.Dockerfile
└── sql/                             # 数据库初始化脚本
    ├── 01-init-schema.sql
    └── 02-seed-data.sql
```

---

## 六、部署配置目录（deploy/）

```
deploy/
├── helm/                            # Helm Charts（生产部署）
│   ├── agentforge-platform/         # 总平台 Chart
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   ├── ai-arp-server/               # 运行平台 Chart
│   ├── ai-aep-server/               # 评测平台 Chart
│   └── ai-aeco-server/              # 生态平台 Chart
├── docker-compose/                  # 本地开发编排
│   ├── docker-compose.yml           # 全量启动
│   ├── docker-compose.dev.yml       # 开发覆盖
│   └── .env.example                 # 环境变量模板
└── k8s/                             # 非 Helm 场景
    ├── namespace.yaml
    ├── configmap.yaml
    ├── secrets.yaml
    └── ingress.yaml
```

---

## 七、命名规范汇总

| 维度 | 规范 | 示例 |
|------|------|------|
| 仓库名 | `ai-{缩写}-Server` / `ai-{缩写}-web` | `ai-arp-Server` |
| Java 包名 | `com.agentforge.{缩写}.{模块}` | `com.agentforge.arp.service` |
| Python 包名 | `app.{模块}` | `app.services.agent_service` |
| npm 包名 | `@agentforge/{缩写}-web` | `@agentforge/arp-web` |
| Docker 镜像 | `agentforge/{工程名}:{版本}` | `agentforge/ai-arp-server:v1.0.0` |
| 数据库名 | `agentforge_{缩写}` | `agentforge_arp` |
| K8s 命名空间 | `agentforge-{缩写}` | `agentforge-arp` |
| Redis 前缀 | `af:{缩写}:` | `af:arp:session:` |
| Kafka Topic | `af.{缩写}.{事件}` | `af.arp.agent.created` |
