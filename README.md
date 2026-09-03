# 学校宿舍管理系统

面向高校宿舍日常运营的前后端分离 Web 系统，将宿管台账、纸质登记、多部门沟通中的宿舍事务统一到一个平台。以角色权限和楼栋/个人数据范围为边界，覆盖管理员、宿管、学生、维修师傅四类用户，打通「资源台账 → 入住退宿 → 日常检查 → 报修协同 → 费用收缴 → 安全管控 → 通知触达 → 统计审计」的业务闭环。

## 功能

| 模块 | 能力 |
|---|---|
| 用户权限 | JWT 登录鉴权，四角色菜单/接口隔离 |
| 宿舍基础数据 | 楼栋 / 楼层 / 房间 / 床位管理与可视化 |
| 学生档案 | 档案维护、Excel 批量导入导出 |
| 入住退宿 | 入住、退宿、调宿、批量退宿 |
| 卫生考勤 | 卫生检查、晚归考勤、寝室评分 |
| 维修工单 | 报修 → 派单 → 接单 → 完工 → 评价 |
| 费用管理 | 住宿费/水电费出账、缴费、催缴 |
| 访客安全 | 访客登记、违禁品检查、安全事件 |
| 公告消息 | 多 scope 公告 + 站内消息未读红点 |
| 统计日志 | 入住率/违纪查询、Excel 导出、操作日志 |

## 技术栈

- **后端**：Spring Boot 3.2.5、JDK 17、MyBatis-Plus 3.5.5、Spring Security 6、JWT (jjwt 0.12)、EasyExcel 3.3、SpringDoc OpenAPI、Spring AOP、Lombok
- **前端**：Vue 3.4、Vite 5、TypeScript (strict)、Vue Router 4、Pinia 2、Element Plus 2、ECharts 5、axios
- **数据库**：MySQL 8（utf8mb4）
- **工程化**：Maven Wrapper、pnpm、Conventional Commits、API 级 E2E 脚本

## 快速开始

环境要求：JDK 17+、Node.js 18+（pnpm 或 npm）、MySQL 8.0+

### 1. 建库（一次性）

```bash
mysql -u root -p < db/init.sql
```

创建数据库 `dorm_system`、业务账号 `dorm/dorm123`、表结构与种子数据。脚本幂等，重复执行不会报错。

> 旧库升级需手动补充公告定向字段：
> ```sql
> ALTER TABLE announcement
>   ADD COLUMN target_room_id BIGINT NULL COMMENT 'scope=2 目标寝室' AFTER building_id,
>   ADD COLUMN target_student_id BIGINT NULL COMMENT 'scope=3 目标学生' AFTER target_room_id;
> ```

### 2. 启动后端（默认 8080）

```bash
cd dorm-system-backend
.\mvnw.cmd spring-boot:run    # Windows
./mvnw spring-boot:run        # macOS / Linux
```

本地敏感配置（数据源密码、JWT secret）放在 `application-local.yml`（已 gitignore，需自行创建）。

- 健康检查：`curl http://localhost:8080/api/v1/ping` → `{"code":0,"message":"success","data":"pong"}`
- 接口文档：http://localhost:8080/swagger-ui.html

### 3. 启动前端（默认 5174）

```bash
cd dorm-system-frontend
pnpm install
pnpm dev
```

浏览器打开：http://localhost:5174

## 默认账号

| 角色 | 用户名 | 密码 |
|---|---|---|
| 管理员 | `admin` | `123456` |
| 宿管 | `dm001` / `dm002` | `123456` |
| 学生 | `stu001` / `stu002` | `123456` |
| 维修师傅 | `repair001` / `repair002` | `dorm123` |

## 角色与权限

| 角色 | 数据范围 | 典型能力 |
|---|---|---|
| 管理员 | 全校 | 主数据维护、出账缴费、派单、公告、统计导出、操作日志 |
| 宿管 | 绑定楼栋 | 本楼入住/卫生/考勤/访客安全/抄表/催缴 |
| 学生 | 本人（及本寝室相关只读） | 账单/公告/消息、报修与评价 |
| 维修师傅 | 派给自己的工单 | 接单、开工、完工 |

完整权限矩阵见 `docs/ROLE_PERMISSION_MATRIX.md`。

## 目录结构

```text
dorm-system/
├── docs/                          # 设计与验收文档
│   ├── DATABASE.md                # 库表设计
│   ├── API_SPEC.md                # 接口规范
│   ├── DEVELOPMENT_PLAN.md        # 分阶段开发计划
│   ├── ROLE_PERMISSION_MATRIX.md  # 四角色权限矩阵
│   ├── FRONTEND_MENU_VISIBILITY.md
│   ├── PHASE_8_9_10_VERIFICATION_REPORT.md
│   ├── E2E_ROLE_MATRIX_REPORT.md  # 四角色 E2E 报告
│   └── scripts/e2e-role-matrix.ps1
├── db/
│   └── init.sql                    # 建库 DDL + 种子数据
├── dorm-system-backend/            # Spring Boot 后端
└── dorm-system-frontend/           # Vue 3 前端
```

## 验证与质量

```bash
# 后端编译
cd dorm-system-backend
.\mvnw.cmd clean compile    # Windows
./mvnw clean compile        # macOS / Linux

# 后端测试
.\mvnw.cmd test

# 前端类型检查
cd dorm-system-frontend
pnpm type-check

# 四角色 API E2E（需后端已启动）
powershell -NoProfile -ExecutionPolicy Bypass -File docs\scripts\e2e-role-matrix.ps1
```

近期全量 E2E 结果：96/96 通过（见 `docs/E2E_ROLE_MATRIX_REPORT.md`）。

## 提交规范

采用 Conventional Commits：

- `feat:` 新功能
- `fix:` 缺陷修复
- `docs:` 文档
- `test:` 测试
- `refactor:` / `chore:` 重构与杂项

## License

前端为 ISC License，后端未指定开源协议，使用前请确认。
