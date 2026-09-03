学校宿舍管理系统
项目简介
本项目是一套面向高校宿舍日常运营的前后端分离全栈 Web 系统，将分散在宿管台账、纸质登记、多部门沟通中的宿舍事务统一到同一平台。系统以角色权限和楼栋/个人数据范围为边界，覆盖管理员、宿管、学生、维修师傅四类用户，打通「资源台账 → 入住退宿 → 日常检查 → 报修协同 → 费用收缴 → 安全管控 → 通知触达 → 统计审计」的完整业务闭环。

解决什么问题
管理分散：楼栋床位、学生档案、卫生考勤、报修缴费、访客安全原先往往多系统/多表格并行，难追溯。
权限不清：不同岗位需要不同操作面；若只靠前端隐藏按钮，存在越权风险。
协同低效：学生报修后缺少标准派单与完工反馈；费用欠缴难以及时提醒；违禁检查结果难以及时触达学生。
数据难用：入住率、违纪情况、缴费进度缺少统一看板与导出能力。
系统如何工作
统一身份与鉴权
后端基于 Spring Security + JWT 完成登录鉴权；接口层使用 @PreAuthorize 做角色控制，服务层再按楼栋/本人做数据过滤。前端按角色收敛菜单与路由，但权限真源在后端。

宿舍主数据与入住生命周期
维护楼栋 / 楼层 / 房间 / 床位台账，支持床位可视化；学生档案可维护与批量导入导出；支持入住、退宿、调宿及管理员批量退宿等流程。

日常管理与安全
宿管可在本楼栋开展卫生检查、晚归考勤、寝室评分；登记访客、录入违禁品（可自动向学生发送整改消息）、上报安全事件；管理员可处理事件并查看全校数据。

维修跨角色状态机
学生/宿管发起报修 → 管理员派单 → 维修师傅接单/完工 → 学生评价。工单在不同角色间按可见性隔离（如未派单前师傅不可见，其他师傅不可见他人工单）。

费用与消息触达
支持住宿费/水电费出账、缴费与催缴；抄表可自动关联用量与金额。公告支持全校/楼栋等 scope；站内消息支持未读统计与已读回写，形成“业务动作 → 通知 → 用户查看”的闭环。

统计、导出与审计
提供入住率、违纪时间线等查询，支持 Excel 导出；敏感写操作可记录操作日志，便于审计。

角色一览
角色	数据范围	典型能力
管理员	全校	主数据维护、出账缴费、派单、公告、统计导出、操作日志
宿管	绑定楼栋	本楼入住/卫生/考勤/访客安全/抄表/催缴
学生	本人（及本寝室相关只读）	账单/公告/消息、报修与评价
维修师傅	派给自己的工单	接单、开工、完工
更完整的权限矩阵见 docs/ROLE_PERMISSION_MATRIX.md。

技术概览
后端：Spring Boot 3.2 + JDK 17 + MyBatis-Plus + Spring Security + JWT
前端：Vue 3 + Vite + TypeScript + Element Plus + Pinia + ECharts
数据库：MySQL 8（utf8mb4）
质量保障：接口规范与分阶段验收文档、角色权限矩阵、API 级四角色 E2E（近期全量 96/96 通过）

功能亮点
模块	能力
用户权限	JWT 登录鉴权，四角色菜单/接口隔离
宿舍基础数据	楼栋 / 楼层 / 房间 / 床位管理与可视化
学生档案	档案维护、批量导入导出
入住退宿	入住、退宿、调宿、批量退宿
卫生考勤	卫生检查、晚归考勤、寝室评分
维修工单	报修 → 派单 → 接单 → 完工 → 评价
费用管理	住宿费/水电费出账、缴费、催缴
访客安全	访客登记、违禁品检查、安全事件
公告消息	多 scope 公告 + 站内消息未读红点
统计日志	入住率/违纪查询、Excel 导出、操作日志
技术栈
后端：Spring Boot 3.2.5、MyBatis-Plus 3.5.5、Spring Security 6、jjwt 0.12、EasyExcel 3.3、SpringDoc OpenAPI、Lombok
前端：Vue 3.4、Vite 5、TypeScript（strict）、Vue Router 4、Pinia 2、Element Plus 2、ECharts 5、axios
数据库：MySQL 8
工程化：Maven Wrapper、pnpm、角色权限矩阵文档、API 级 E2E 脚本
目录结构
dorm-system/
├── docs/                         # 设计与验收文档
│   ├── DATABASE.md               # 库表设计
│   ├── DEVELOPMENT_PLAN.md       # 分阶段开发计划
│   ├── API_SPEC.md               # 接口规范
│   ├── ROLE_PERMISSION_MATRIX.md # 四角色权限矩阵
│   ├── FRONTEND_MENU_VISIBILITY.md
│   ├── PHASE_8_9_10_VERIFICATION_REPORT.md
│   ├── E2E_ROLE_MATRIX_REPORT.md # 四角色 E2E 报告
│   └── scripts/e2e-role-matrix.ps1
├── db/init.sql                   # 建库 DDL + 种子数据
├── dorm-system-backend/          # Spring Boot 后端
└── dorm-system-frontend/         # Vue 3 前端
默认账号
角色	用户名	密码
管理员	admin	123456
宿管	dm001 / dm002	123456
学生	stu001 / stu002	123456
维修师傅	repair001 / repair002	dorm123
本地启动
1. 建库（一次性，使用 MySQL root）
mysql -u root -p < db/init.sql
会创建数据库 dorm_system、业务用户 dorm/dorm123、表结构与种子数据。

若本地是旧库（缺公告定向字段），可执行：

ALTER TABLE announcement
  ADD COLUMN target_room_id BIGINT NULL COMMENT 'scope=2 目标寝室' AFTER building_id,
  ADD COLUMN target_student_id BIGINT NULL COMMENT 'scope=3 目标学生' AFTER target_room_id;
2. 后端（默认 8080）
cd dorm-system-backend
# Windows 示例
set JAVA_HOME=C:\Users\<你的用户>\.jdks\ms-17.0.19
.\mvnw.cmd spring-boot:run
本地敏感配置放在 application-local.yml（已 gitignore，不入库），需自行配置数据源与 JWT secret。

健康检查：

curl http://localhost:8080/api/v1/ping
# 期望：{"code":0,"message":"success","data":"pong"}
Swagger：http://localhost:8080/swagger-ui.html

3. 前端（默认 5174）
cd dorm-system-frontend
pnpm install   # 或 npm install
pnpm dev       # 或 npm run dev
浏览器打开：http://localhost:5174

若 5173 被占用，本项目前端可使用 5174。

角色与权限（摘要）
角色	典型能力
管理员	全校主数据、出账缴费、派单、公告、统计导出、操作日志
宿管	本楼栋入住/卫生/考勤/访客安全/抄表/催缴
学生	本人账单、消息、公告、报修与评价
维修师傅	查看派给自己的工单并接单/完工
完整矩阵见：docs/ROLE_PERMISSION_MATRIX.md

验证与质量
后端编译：./mvnw.cmd clean compile
后端测试：./mvnw.cmd test
前端类型检查：pnpm type-check 或 npm run type-check
四角色 API E2E（需后端已启动）：
powershell -NoProfile -ExecutionPolicy Bypass -File docs\scripts\e2e-role-matrix.ps1
近期全量 E2E 结果：96/96 通过（见 docs/E2E_ROLE_MATRIX_REPORT.md）。

提交规范
采用 Conventional Commits：

feat: 新功能
fix: 缺陷修复
docs: 文档
test: 测试
refactor: / chore: 重构与杂项
开发进度
已完成脚手架、鉴权、宿舍结构、学生档案、入住退宿、卫生考勤、维修、访客安全、费用公告、统计日志与收尾验收。
明细见 docs/DEVELOPMENT_PLAN.md。

许可证
仅供学习与交流使用，未另行声明开源协议前请勿商用。
