# Headwind MDM Server — 技术栈与功能概要

## 项目简介

**Headwind MDM** 是一个开源的 Android 移动设备管理（MDM）平台，面向企业应用开发者和 IT 管理员。提供通过二维码扫描注册设备、应用部署、设备配置管理、设备监控等功能，通过 Web 控制面板进行管理。

- **官网**: https://h-mdm.com
- **许可证**: Apache License 2.0
- **版本**: Server 0.1.0 / Web Client 5.39.2

---

## 技术栈

### 后端

| 类别 | 技术 | 版本 |
|------|------|------|
| **语言** | Java 8 | 1.8 |
| **构建工具** | Apache Maven (多模块) | — |
| **Web 框架** | JAX-RS / Jersey | 2.25.1 |
| **依赖注入** | Google Guice + Guice-Servlet + Guice-Multibindings | 4.2.2 |
| **ORM** | MyBatis + MyBatis-Guice | 3.5.3 / 3.10 |
| **数据库** | PostgreSQL | JDBC 42.3.3 |
| **数据库迁移** | Liquibase | 3.6.3 |
| **鉴权** | JWT (jjwt) + Session + 可选 LDAP | jjwt 0.9.1 |
| **消息推送** | ActiveMQ (嵌入式) + MQTT + HTTP Long Polling | 5.15.16 |
| **API 文档** | Swagger (swagger-jersey2-jaxrs) | 1.5.24 |
| **APK 解析** | apk-parser | 2.6.10 |
| **二维码生成** | qrgen | 2.5.0 |
| **Excel 导出** | Apache POI | 5.4.0 |
| **图片处理** | imgscalr | 4.2 |
| **日志** | SLF4J + Log4j | SLF4J 2.0.16 / Log4j 1.2 |
| **插件类扫描** | ClassGraph | 4.8.53 |
| **简化代码** | Lombok | 1.18.20 |
| **运行容器** | Apache Tomcat (部署为 WAR) | — |

### 测试

| 类别 | 技术 | 版本 |
|------|------|------|
| **单元测试** | JUnit | 4.13.1 |
| **Mock** | Mockito | 1.10.19 |
| **断言** | AssertJ | 3.14.0 |

### 前端

| 类别 | 技术 | 版本 |
|------|------|------|
| **框架** | AngularJS 1.x | — |
| **UI 组件** | UI Bootstrap, UI Router | — |
| **图表** | Chart.js | — |
| **构建** | Grunt + Bower + npm (frontend-maven-plugin) | Node 18.16.0 |
| **国际化** | angular-translate (13 种语言) | — |

---

## 项目模块结构

```
hmdm-server/                              # 根 POM — 多模块 Maven 项目 (Java 8)
│
├── common/                               # [jar] 核心共享库
│   └── src/main/java/com/hmdm/
│       ├── auth/                         # 认证接口定义 (AuthService, AuthType)
│       ├── event/                        # 事件系统 (EventService, EventListener)
│       ├── guice/module/                 # Guice 通用模块 (ConfigureModule, PersistenceModule)
│       ├── persistence/
│       │   ├── domain/                   # 领域实体 (Device, Application, Configuration,
│       │   │                             #   User, UserRole, Customer, Group, Setting, Icon...)
│       │   └── mapper/                   # MyBatis Mapper 接口 + XML 映射文件
│       │       ├── *.java                #   DeviceMapper, UserMapper, ConfigurationMapper,
│       │       │                         #   ApplicationMapper, CustomerMapper, UserRoleMapper...
│       │       └── *.xml                 #   对应的 SQL 映射 XML
│       ├── rest/
│       │   ├── filter/                   # Servlet 过滤器 (AuthFilter, CORSFilter,
│       │   │                             #   HSTSFilter, IPFilter, PublicIPFilter)
│       │   └── json/                     # 通用请求/响应 DTO 和视图模型
│       ├── security/                     # 安全相关接口
│       ├── service/                      # 通用服务 (EmailService, DeviceStatusService,
│       │                                 #   FileUploadService, RsaKeyService...)
│       └── util/                         # 工具类 (APKFileAnalyzer, CryptoUtil, PasswordUtil,
│                                         #   StringUtil, FileUtil, RESTUtil...)
│
├── jwt/                                  # [jar] JWT 鉴权模块 (jjwt 0.9.1)
│   └── src/main/java/com/hmdm/security/jwt/
│       ├── JwtTokenGenerator.java        # JWT Token 生成
│       ├── JwtTokenValidator.java        # JWT Token 验证
│       └── rest/                         # JWT REST 资源 (Token 刷新等)
│
├── notification/                         # [jar] 消息推送模块
│   └── src/main/
│       ├── java/com/hmdm/notification/
│       │   ├── guice/module/             # Guice 模块 (NotificationLiquibaseModule,
│       │   │                             #   NotificationPersistenceModule,
│       │   │                             #   NotificationRestModule, MqttConfigModule)
│       │   ├── persistence/
│       │   │   ├── domain/               # 实体 (PushMessage, PendingPush)
│       │   │   └── mapper/               # MyBatis Mapper
│       │   └── rest/
│       │       ├── NotificationResource.java  # 推送消息 REST API
│       │       ├── LongPollingServlet.java    # HTTP Long Polling 降级方案
│       │       └── json/                      # 请求/响应 DTO
│       └── resources/
│           └── liquibase/                # 数据库迁移脚本
│
├── server/                               # [war] 主 Web 应用 → launcher.war
│   └── src/main/
│       ├── java/com/hmdm/
│       │   ├── guice/module/             # Guice 模块 (MainRestModule,
│       │   │                             #   PrivateRestModule, PublicRestModule)
│       │   ├── rest/
│       │   │   ├── resource/             # JAX-RS REST 端点 (26 个资源类)
│       │   │   │   ├── AuthResource.java           # 登录认证
│       │   │   │   ├── DeviceResource.java         # 设备 CRUD
│       │   │   │   ├── ApplicationResource.java    # 应用管理
│       │   │   │   ├── ConfigurationResource.java  # 配置模板管理
│       │   │   │   ├── ConfigurationFileResource.java  # 配置文件
│       │   │   │   ├── GroupResource.java          # 设备分组管理
│       │   │   │   ├── UserResource.java           # 用户管理
│       │   │   │   ├── UserRoleResource.java       # 角色管理
│       │   │   │   ├── CustomerResource.java       # 租户管理
│       │   │   │   ├── SettingsResource.java       # 系统设置
│       │   │   │   ├── FilesResource.java          # 文件上传/下载
│       │   │   │   ├── PublicResource.java         # 公开 API (设备注册、配置/应用下载)
│       │   │   │   ├── SyncResource.java           # 设备数据同步
│       │   │   │   ├── PushApiResource.java        # 推送消息
│       │   │   │   ├── SummaryResource.java        # 仪表盘摘要
│       │   │   │   ├── QRCodeResource.java         # 二维码生成
│       │   │   │   ├── SignupResource.java         # 自助注册
│       │   │   │   ├── PasswordResetResource.java  # 密码重置
│       │   │   │   ├── UpdateResource.java         # 应用更新
│       │   │   │   ├── VideosResource.java         # 视频文件管理
│       │   │   │   ├── StatsResource.java          # 使用统计收集
│       │   │   │   ├── IconResource.java           # 图标管理
│       │   │   │   ├── HintResource.java           # 提示信息
│       │   │   │   ├── PublicFilesResource.java    # 公开文件
│       │   │   │   └── DownloadFilesServlet.java   # 文件直接下载
│       │   │   └── json/                 # 请求/响应 DTO + View 模型
│       │   │       ├── SyncResponse.java           # 同步响应
│       │   │       ├── PushRequest.java            # 推送请求
│       │   │       ├── SummaryResponse.java        # 摘要响应
│       │   │       └── view/                       # 视图模型
│       │   │           ├── devicelist/             #   设备列表视图
│       │   │           └── user/                   #   用户视图
│       │   └── task/                     # 后台任务
│       │       ├── CustomerStatusTask.java   # 租户状态检查
│       │       ├── FileCheckTask.java        # 文件完整性检查
│       │       └── FileMigrateTask.java      # 文件迁移
│       ├── resources/
│       │   └── liquibase/                # 核心数据库迁移脚本
│       └── webapp/                       # AngularJS 前端
│           ├── WEB-INF/                  # web.xml + context.xml
│           ├── app/                      # 前端应用源码
│           │   ├── app.js                # AngularJS 主入口 (headwind-kiosk)
│           │   ├── components/
│           │   │   ├── main/             #   主布局
│           │   │   ├── header/           #   头部导航
│           │   │   ├── about/            #   关于页面
│           │   │   └── control-panel/    #   控制面板
│           │   │       ├── controller/   #     AngularJS 控制器
│           │   │       └── view/         #     HTML 模板
│           │   └── shared/
│           │       ├── service/          #   共享服务 (API 调用、认证、工具)
│           │       └── view/             #   共享视图组件
│           ├── css/                      # 样式表
│           ├── images/                   # 图片资源
│           ├── lib/                      # 前端依赖库 (17 个)
│           │   ├── angular/              #   AngularJS 核心
│           │   ├── angular-resource/     #   ngResource
│           │   ├── angular-cookies/      #   ngCookies
│           │   ├── angular-sanitize/     #   ngSanitize
│           │   ├── angular-animate/      #   ngAnimate
│           │   ├── angular-bootstrap/    #   UI Bootstrap
│           │   ├── angular-ui-router/    #   UI Router
│           │   ├── angular-chart.js/     #   Chart.js 适配
│           │   ├── angular-css/          #   CSS 动态加载
│           │   ├── angular-breadcrumb/   #   面包屑导航
│           │   ├── angular-input-dropdown/  # 下拉输入
│           │   ├── angular-ui-mask/      #   输入掩码
│           │   ├── angular-bootstrap-colorpicker/  # 颜色选择器
│           │   ├── bootstrap-css-only/   #   Bootstrap CSS
│           │   ├── chart.js/             #   图表库
│           │   ├── ng-idle/              #   空闲超时检测
│           │   ├── ng-tags-input/        #   标签输入
│           │   ├── oclazyload/           #   延迟加载
│           │   ├── blueimp-md5/          #   MD5 哈希
│           │   └── intro.js/             #   引导教程
│           ├── ext/                      # 外部扩展
│           │   ├── leaflet-1.5.1/        #   交互地图
│           │   └── leaflet.markercluster-1.4.1/  # 地图标记聚合
│           └── localization/             # 国际化 (13 种语言)
│               ├── locale-en_US.json
│               ├── locale-ru_RU.json
│               ├── locale-fr_FR.json
│               ├── locale-pt_PT.json
│               ├── locale-ar_AE.json
│               ├── locale-es_ES.json
│               ├── locale-de_DE.json
│               ├── locale-zh_TW.json
│               ├── locale-zh_CN.json
│               ├── locale-ja_JP.json
│               ├── locale-tr_TR.json
│               ├── locale-vi_VN.json
│               └── locale-it_IT.json
│
├── plugins/                              # [pom] 插件模块父 POM
│   │
│   ├── platform/                         # [jar] 插件平台核心 (ClassGraph 扫描)
│   │   └── src/main/java/com/hmdm/plugin/
│   │       ├── guice/module/             # Guice 模块 (PluginLiquibaseModule,
│   │       │                             #   PluginPersistenceModule, PluginRestModule)
│   │       ├── persistence/
│   │       │   ├── domain/               #   插件基础设施实体
│   │       │   └── mapper/               #   MyBatis Mapper
│   │       ├── rest/                     #   插件 REST 端点
│   │       └── service/                  #   插件服务 (PluginConfiguration 接口定义)
│   │
│   ├── audit/                            # [jar] API 审计日志插件
│   │   └── src/main/java/com/hmdm/plugins/audit/
│   │       ├── guice/module/             # Guice 模块
│   │       ├── persistence/
│   │       │   ├── domain/               #   审计日志实体
│   │       │   └── mapper/               #   MyBatis Mapper
│   │       └── rest/
│   │           ├── filter/               #   审计过滤器 (请求/响应拦截记录)
│   │           └── json/                 #   DTO
│   │
│   ├── deviceinfo/                       # [jar] 设备动态信息追踪插件
│   │   └── src/main/java/com/hmdm/plugins/deviceinfo/
│   │       ├── guice/module/             # Guice 模块
│   │       ├── persistence/
│   │       │   ├── domain/               #   设备信息实体 (GPS/WiFi/移动网络/电池)
│   │       │   └── mapper/               #   MyBatis Mapper
│   │       ├── rest/json/                #   DTO
│   │       └── service/                  #   设备信息采集服务
│   │
│   ├── devicelog/                        # [pom] 设备日志收集插件 (嵌套模块)
│   │   ├── core/                         #   [jar] 日志收集核心
│   │   │   └── src/main/java/com/hmdm/plugins/devicelog/
│   │   │       ├── guice/module/         #     Guice 模块
│   │   │       ├── model/                #     日志数据模型
│   │   │       ├── persistence/          #     持久化接口 (抽象层)
│   │   │       ├── rest/
│   │   │       │   ├── resource/         #     日志 REST API
│   │   │       │   └── json/             #     DTO
│   │   │       └── task/                 #     日志后台任务
│   │   │
│   │   └── postgres/                     #   [jar] PostgreSQL 持久化实现
│   │       └── src/main/java/com/hmdm/plugins/devicelog/persistence/postgres/
│   │           ├── dao/
│   │           │   ├── domain/           #     PostgreSQL 特定实体
│   │           │   └── mapper/           #     PostgreSQL MyBatis Mapper
│   │           └── guice/module/         #     Guice 模块 (绑定实现)
│   │
│   ├── messaging/                        # [jar] 设备消息插件
│   │   └── src/main/java/com/hmdm/plugins/messaging/
│   │       ├── guice/module/             # Guice 模块
│   │       ├── persistence/
│   │       │   ├── domain/               #   消息实体
│   │       │   └── mapper/               #   MyBatis Mapper
│   │       └── rest/json/                #   DTO
│   │
│   ├── push/                             # [jar] 定时推送插件
│   │   └── src/main/java/com/hmdm/plugins/push/
│   │       ├── guice/module/             # Guice 模块
│   │       ├── persistence/
│   │       │   ├── domain/               #   定时推送实体
│   │       │   └── mapper/               #   MyBatis Mapper
│   │       └── rest/json/                #   DTO
│   │
│   └── xtra/                             # [jar] 扩展功能插件
│       └── src/main/java/com/hmdm/plugins/xtra/
│           ├── guice/module/             # Guice 模块
│           └── (其他扩展功能)
│
├── swagger/                              # [pom] Swagger API 文档
│   └── ui/                               #   [war] Swagger UI (独立部署)
│
└── install/                              # 安装与初始化资源
    ├── sql/                              #   数据库初始化 SQL (中/英文)
    │   ├── hmdm_init.en.sql              #     英文版建库脚本
    │   └── hmdm_init.ru.sql              #     俄文版建库脚本
    ├── context_template.xml              #   Tomcat context.xml 模板
    ├── log4j_template.xml                #   Log4j 配置模板
    └── emails/                           #   邮件模板
```

---

## 核心功能

### 1. 设备管理
- 通过扫描二维码注册 Android 7+ 设备
- 支持免注册的"应用模式"
- 设备分组管理
- 设备状态监控（在线/离线、电量、网络、GPS 位置等）
- 设备配置文件的创建与下发

### 2. 应用管理
- 应用目录管理（上传 APK、版本管理）
- 应用自动部署到设备
- 应用配置参数集中管理
- APK 文件解析（包名、版本号、权限等）
- 应用图标管理

### 3. 设备配置
- 配置模板（Configuration）管理
- 设备功能开关控制（GPS、Wi-Fi、蓝牙等）
- 系统更新策略配置
- 配置文件关联与下发

### 4. 通知与推送
- MQTT 实时推送（嵌入式 ActiveMQ Broker，端口 31000）
- HTTP Long Polling 降级方案
- 定时推送消息
- 推送消息发送（单设备/批量/按组）

### 5. 多租户
- 支持 `shared`（多租户）和 `private`（单租户）两种部署模式
- 客户（Customer）/租户管理
- 客户自助注册
- UI 品牌自定义（rebranding）

### 6. 权限系统
- 基于角色的访问控制（RBAC）
- 细粒度权限配置
- 用户管理
- 支持本地认证和 LDAP 认证
- JWT Token + Session 双重认证

### 7. 插件系统
- 可扩展的插件架构
- 运行时插件扫描与注册（ClassGraph）
- 插件可提供：持久层、数据库迁移、REST API、后台任务
- 内置插件：设备日志、设备信息、审计、消息、推送

### 8. 日志与审计
- 设备应用日志收集与 Web 面板查看
- REST API 请求/响应审计记录

### 9. 其他功能
- 仪表盘摘要统计
- 密码找回/重置（通过 SMTP 邮件）
- 使用统计数据收集
- 13 种语言国际化支持
- 反向代理支持

---

## REST API 概览

### 公开接口 (`/rest/public/`)

| 路径 | 功能 |
|------|------|
| `/rest/public/auth` | 用户登录认证 |
| `/rest/public/auth/jwt` | JWT Token 认证 |
| `/rest/public/sync` | 设备数据同步 |
| `/rest/public` | 移动客户端 API（设备注册、配置/应用下载） |
| `/rest/public/files` | 公开文件下载 |
| `/rest/public/qrcode` | 设备注册二维码生成 |
| `/rest/public/stats` | 使用统计收集 |

### 私有接口 (`/rest/private/`) — 需要认证

| 路径 | 功能 |
|------|------|
| `/rest/private/devices` | 设备 CRUD、搜索、分配 |
| `/rest/private/groups` | 设备分组管理 |
| `/rest/private/configurations` | 配置模板 CRUD |
| `/rest/private/applications` | 应用目录管理（CRUD、版本、上传） |
| `/rest/private/users` | 用户管理 |
| `/rest/private/roles` | 角色管理 |
| `/rest/private/customers` | 租户管理 |
| `/rest/private/settings` | 系统设置 |
| `/rest/private/files` | 文件上传/下载 |
| `/rest/private/push` | 推送消息发送 |
| `/rest/private/summary` | 仪表盘摘要 |
| `/rest/private/signup` | 客户自助注册 |
| `/rest/private/password` | 密码重置 |
| `/rest/private/update` | 应用更新管理 |
| `/rest/private/videos` | 视频文件管理 |

---

## 关键配置项

配置通过 Tomcat `context.xml` 的 `<context-param>` 注入：

| 配置项 | 说明 |
|--------|------|
| `jdbc.url/driver/username/password` | PostgreSQL 数据库连接 |
| `base.directory` | 应用内部数据目录 |
| `files.directory` | 上传文件存储目录 |
| `base.url` | 部署基础 URL |
| `usage.scenario` | 部署模式：`shared` / `private` |
| `secure.enrollment` / `hash.secret` | 设备注册安全配置 |
| `mqtt.server.uri` / `mqtt.auth` | MQTT 推送配置 |
| `smtp.*` | 邮件服务配置（密码找回） |
| `jwt.secretkey` / `jwt.validity` | JWT Token 配置 |
| `ldap.*` | LDAP 集成配置 |
| `aapt.command` | Android aapt 工具路径（APK 解析） |
| `rebranding.*` | UI 品牌定制 |

---

## 本地开发环境搭建（Windows）

### 1. 前置依赖

- **JDK 8** （必须，项目用 Java 8 编译）
- **Maven 3.x** + **PostgreSQL**（需启动服务）
- 无需单独安装 Tomcat — 项目内置 `tomcat7-maven-plugin`

### 2. 创建数据目录

```bash
mkdir -p D:/home/launcher/files
mkdir -p D:/home/launcher/plugins
```

### 3. 创建 PostgreSQL 数据库

```sql
CREATE USER hmdm WITH PASSWORD 'hmdm';
CREATE DATABASE hmdm WITH OWNER=hmdm;
```

### 4. 配置 build.properties

从模板复制并修改：

```bash
cp server/build.properties.example server/build.properties
```

关键修改项（Windows 本地开发）：

| 配置项 | 值 |
|--------|-----|
| `base.directory` | `D:/home/launcher` |
| `files.directory` | `D:/home/launcher/files` |
| `plugins.files.directory` | `D:/home/launcher/plugins` |
| `base.url` | `http://localhost:9090` |
| `initialization.completion.signal.file` | `D:/home/launcher/initialized.txt` |
| `log4j.config` | `file:///D:/home/launcher/log4j-hmdm.xml` |

### 5. 下载 ActiveMQ 依赖（必须先做）

**⚠️ 注意：** `notification/pom.xml` 中 ActiveMQ 版本需要 **5.15.16**（最后一个支持 Java 8 的版本），**不能**使用 5.18.6（需要 Java 11+）。

确保 `notification/pom.xml` 中 ActiveMQ 版本为 5.15.16：

```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-broker</artifactId>
    <version>5.15.16</version>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-mqtt</artifactId>
    <version>5.15.16</version>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-jaas</artifactId>
    <version>5.15.16</version>
</dependency>
```

### 6. 构建项目

```bash
# 首次构建：安装所有模块到本地 Maven 仓库
mvn clean install -DskipTests

# 后续构建只需 package
mvn clean package -DskipTests
```

### 7. 复制 log4j 配置

```bash
cp install/log4j_template.xml D:/home/launcher/log4j-hmdm.xml
```

### 8. 启动应用

```bash
# Maven 自动下载内嵌 Tomcat 7，端口 9090，应用部署在根路径 /
mvn tomcat7:run -pl server
```

### 9. 访问

浏览器打开：**`http://localhost:9090`**

### 10. 停止

在终端按 `Ctrl + C` 停止 Tomcat。

### 启动成功标志

控制台输出以下日志表示启动成功：

```
[HMDM-INITIALIZER]: Application initialization was successful
Apache ActiveMQ 5.15.16 ... started
Listening for connections at: mqtt://127.0.0.1:31000
Starting ProtocolHandler ["http-bio-9090"]
```

### 常见问题

| 问题 | 解决方案 |
|------|---------|
| `类文件具有错误的版本 55.0, 应为 52.0` | ActiveMQ 版本用了 5.18.x，降级到 5.15.16 |
| `Could not resolve dependencies` | 先执行 `mvn install -DskipTests` 安装本地模块 |
| `Connection refused` 访问 PostgreSQL | 确保 PostgreSQL 服务已启动 |
| `${xxx}` 占位符未被替换 | 在 `build.properties` 中补充缺失的属性 |
| `ClassFormatException: Invalid byte tag in constant pool: 19` | Tomcat 7 BCEL 库无法解析 Java 9+ 的 module-info.class。已通过以下两处修改修复：(1) `web.xml` 升级至 Servlet 3.0 + `metadata-complete="true"`；(2) `context.xml` 添加 `<JarScanner scanAllDirectories="false" scanClassPath="false"/>` |
