---
name: springboot-vue-development
description: 使用 Java 17、Spring Boot、MyBatis-Plus、MySQL、Vue 3 和 Element Plus 开发前后端分离管理系统。编码风格参考 sky-take-out 的 Controller/Service/Mapper 分层、DTO/VO、BeanUtils、Lombok 与日志。适用于新项目搭建、CRUD、账号权限、业务流程、管理页面、接口联调和测试，不限定业务领域；维护已有项目时遵守已确认的技术与接口契约。
---

# Spring Boot + Vue 通用管理系统开发规范

## 1. 适用范围

用于不同业务领域的前后端分离管理系统，例如人员、资产、库存、预约、教学或业务后台。本文规定技术选型和编码习惯，不预设业务实体、角色名称、接口清单、数据库名称、项目目录或业务规则。

编码风格参考 sky-take-out：清晰分层、Service 接口与实现分离、DTO/VO 普通 Java 类、BeanUtils 对象转换、Lombok、中文注释及业务日志。**参考编码风格不等于复制其技术依赖、业务模型或历史实现。** 持久化使用 MyBatis-Plus，前端使用 Vue 3，默认接口文档为 Markdown。

- 用户当前要求决定实施范围；写方案、解释代码或生成规范不授权修改业务工程。
- 新项目按本技能建立结构；既有项目先识别其约定，在相关范围内统一风格，不批量改名或更换框架。
- 用户明确指定的技术、字段和契约优先于本文默认值。
- 不把示例中的名称、参数、角色和业务条件当成所有项目的要求。
- 新功能完成纵向链路和必要验证；缺失的关键业务决策需要澄清，独立工作继续推进。

## 2. 技术栈与版本管理

| 范围 | 默认方案 |
| --- | --- |
| Java | Java 17 |
| 后端框架 | Spring Boot 3.x、Spring MVC、Maven |
| 数据访问 | MyBatis-Plus 的 Spring Boot 3 Starter、MySQL Connector/J |
| 数据库 | MySQL 8.x、InnoDB、utf8mb4 |
| 校验与模型 | Jakarta Bean Validation、Lombok |
| 对象转换与 JSON | Spring BeanUtils、Jackson |
| 认证 | Spring Security、JWT；账号密码使用 BCrypt |
| 日志 | SLF4J，使用 Lombok @Slf4j |
| 前端 | Vue 3、Vite、Vue Router、Pinia、Axios、Element Plus |
| 图表 | 有统计图需求时使用 ECharts |
| 后端测试 | Spring Boot Test、JUnit 5、MockMvc、Mockito；按需隔离数据库测试 |
| 前端测试 | 纯逻辑单元测试、Playwright 浏览器测试 |
| 接口说明 | Markdown，按需提供 IDE HTTP 请求示例 |

新项目选择经过兼容核实的精确版本；维护项目以 pom.xml、package.json 和锁文件为准，不为普通功能顺带升级。Spring Boot 3 的 Servlet、Validation 使用 jakarta 包；javax.sql、javax.crypto 等 Java SE API 不进行机械替换。

采用单体后端与独立前端。单模块是简单项目的默认选择；只有明确的复用或构建需求才拆 Maven 模块。默认不引入微服务、Redis、消息队列、工作流引擎或复杂缓存体系；业务确有需要时再论证并实施。

不同时引入 JPA 与 MyBatis-Plus 实现同一数据访问职责；不为了对象转换再引入 MapStruct 等第二套方案。依赖版本集中管理，优先遵循 Spring Boot BOM；npm 使用锁文件保证安装可重复。

## 3. 目录与命名

### 后端

根包由项目组织和名称决定，不固定为 com.sky 或某个历史项目包名。

```text
src/main/java/<根包>/
  Application.java
  controller/
    admin/        管理端接口（有管理端时）
    user/         用户端接口（有用户端时）
  service/        业务接口
    impl/         业务实现
  mapper/         数据访问接口
  entity/         数据库实体
  dto/            请求数据对象
  vo/             响应数据对象
  result/         统一响应对象
  config/         配置与自动填充
  exception/      业务异常与统一异常处理
  security/       认证、授权与身份上下文
  constant/       确有复用价值的常量
  enumeration/    业务枚举
src/main/resources/
  application.yml
src/test/
```

按需求创建目录，不预建空包。业务调度、领域协作组件可按职责增加 task 或领域子包；依赖 Mapper、事务或业务状态的组件不放进通用 tools/utils。不要额外堆叠 DAO、Repository、Manager、Facade 等重复层。

### 前端

```text
src/
  api/            请求实例和按资源组织的接口函数
  stores/         Pinia 全局状态
  router/         路由与访问守卫；小项目可用 router.js
  pages/          页面
    admin/        管理端页面（按需）
  components/     复用组件
  utils/          纯工具函数及已有组合式工具
  styles/         公共样式；小项目可用 styles.css
  App.vue
  main.js
```

默认使用 JavaScript ES Modules、Vue SFC 和 `<script setup>`。既有 TypeScript 项目保持 TypeScript，不降级重写。

### 命名和格式

- 类、组件使用大驼峰；变量和方法使用小驼峰；包名小写；常量大写下划线。
- 使用 XxxController、XxxService、XxxServiceImpl、XxxMapper、XxxDTO、XxxVO；实体名称表达实际业务。
- DTO 不使用 Request 后缀；返回对象放 vo，不放 dto/response。
- 普通动作采用 save、list、getById、update、deleteById；分页采用 page/pageQuery；特定业务使用 approve、cancel 等有含义的方法名。
- Controller 按业务资源组织，按需区分 admin/user。两端同名类需要避免 Spring Bean 名冲突。
- Java 四空格缩进，按方法正常换行；中文类/方法注释，复杂业务分支写必要说明，不编造作者或日期。
- 优先跟随模块已有依赖注入方式；新项目可统一构造器注入。模仿风格不要求批量替换注入方式。

## 4. 分层职责

主链：`Controller → Service 接口 → ServiceImpl → Mapper → 数据库`。

| 层 | 职责和限制 |
| --- | --- |
| Controller | 接收参数、触发校验、记录必要日志、调用 Service、包装 Result；不直接调用 Mapper、不写业务事务 |
| Service | 声明业务动作和返回类型；不混入实现逻辑 |
| ServiceImpl | 业务检查、对象转换、权限与数据范围、关联查询、事务、批量操作及分页 |
| Mapper | 继承 BaseMapper，负责数据持久化；不承载业务规则 |
| Entity | 映射数据库记录，包含业务和审计字段 |
| DTO | 仅接收本次动作允许的字段，声明输入校验 |
| VO | 仅承载可公开给前端的数据，不执行查询或业务动作 |

简单业务保持直接实现，不为每个动作创建策略、工厂、转换服务。公共业务能力优先复用已有 Service 或职责明确的组件，避免循环依赖。

## 5. DTO、VO、Entity 与对象转换

DTO 和 VO 使用 Lombok `@Data` 并实现 `Serializable`，不默认使用 record。根据构造方式增加 `@Builder`、`@NoArgsConstructor`、`@AllArgsConstructor`，不机械添加全部注解。

```java
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import java.io.Serializable;
import lombok.Data;

/** 新增资源参数。 */
@Data
public class ResourceDTO implements Serializable {
    @NotBlank
    @Size(max = 100)
    private String name;
}
```

- 新增请求不要求数据库生成的 ID；更新请求按接口设计通过路径或专用 DTO 接收 ID。
- 校验必填、长度、数值边界、格式和集合大小；状态值不能只校验非空。
- DTO 不接收或不信任角色、当前操作者、密码哈希、审计时间和不可由用户决定的业务阶段。
- Entity 使用 @TableName、@TableId、@TableField 等明确映射；是否自增按实际 DDL。
- 返回对象不直接暴露密码哈希、内部凭据等字段。管理员专用字段用独立 AdminXxxVO，不默认扩散到普通用户响应。
- 同名属性转换用 `BeanUtils.copyProperties`，在 ServiceImpl 中显式设置受保护字段和关联展示信息。

```java
Resource resource = new Resource();
BeanUtils.copyProperties(resourceDTO, resource);
resource.setEnabled(1);
resourceMapper.insert(resource);

ResourceVO resourceVO = new ResourceVO();
BeanUtils.copyProperties(resource, resourceVO);
return resourceVO;
```

示例只表示转换顺序，实体及默认值由业务决定。不要为了方便把完整 Entity 作为更新请求；复制不等于授权。可清空字段要核对 MyBatis-Plus 的 null 更新策略，避免用户提交清空却未写入数据库。

## 6. REST API、Result、列表和文档

按资源设计 URL。新项目可采用 `/api/admin/<资源>` 和 `/api/<资源>`；既有项目保持已发布路径。GET 查询、POST 创建、PUT 修改完整约定字段、PATCH 局部变更、DELETE 删除或执行已明确约定的取消语义。

- JSON 请求用 `@Valid @RequestBody`；查询 DTO 绑定查询参数；路径变量显式声明。
- 创建资源可返回 201 和 Location；更新是否返回 VO 按接口约定统一。不要为改变代码风格修改已有成功响应。
- 所有普通 JSON 接口统一 Result；文件下载不把二进制包在 Result 中。
- 默认 Result 使用以下结构；既有项目契约优先，不能无授权切换数字/字符串状态码或 message/msg。

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {},
  "requestId": "请求追踪标识"
}
```

统一通过 `Result.success(data)`、`Result.success()`、`Result.error(code, message)` 构建响应。保留正确 HTTP 状态与稳定业务码，不把所有错误都伪装成成功。

### 列表与分页

默认列表返回 `Result<List<XxxVO>>`，data 为数组，空结果为 `[]`。不因为采用管理后台风格自动引入分页。

数据量或用户需求明确需要分页时，使用 MyBatis-Plus 分页插件，先确定 page、pageSize、最大页大小及响应结构。分页 DTO 可命名 XxxPageQueryDTO，Service 使用 pageQuery。不要混用 PageHelper；不要直接把框架内部分页对象当成未经设计的公共 API。已有列表转分页属于接口变更，必须同步前端、文档和测试。

### 文档

使用 Markdown 记录接口路径、方法、权限、请求字段、默认值、响应示例、错误码和边界条件；按模块组织。新增接口同步提供可执行 HTTP 示例，凭据使用变量。

默认不引入 Knife4j/Swagger。用户明确要求在线文档时，核实与目标 Boot 版本兼容的方案再增加。

## 7. Mapper、数据库与审计

- Mapper 继承 `BaseMapper<Entity>`；使用 LambdaQueryWrapper、LambdaUpdateWrapper 等构造查询与更新。
- 默认不在 Mapper 写 @Select/@Insert 等 SQL，不新增 XML。复杂查询先评估 MyBatis-Plus 现有能力；确实需要偏离时说明必要性，不偷偷另建持久化路线。
- 参数使用安全绑定，禁止将用户输入直接拼接到 SQL、last、排序字段或表名；动态排序使用允许列表。
- 关联展示尽量批量查询后转换，避免 N+1；空 ID 集合应显式处理，不能退化成全表查询。
- 更新和删除必须有明确范围，检查必要的受影响行数；不要使用无条件 Wrapper。
- 表名和列名使用小写下划线，Java 属性小驼峰，开启映射；主外键、唯一约束、索引以 DDL 为准，不凭 Entity 猜测。
- 新项目审计时间默认 `created_time`、`updated_time`，Java 为 createdTime、updatedTime；旧项目保留现有命名。
- 使用 MyBatis-Plus MetaObjectHandler 和字段填充声明统一设置时间，不同时引入另一套反射审计切面。
- 创建人和修改人仅在业务需要且数据库已有对应列时增加，身份取自认证上下文。
- 启用字段默认 enabled，1/0 表示启用/禁用；业务阶段另用清晰枚举。不要把禁用自动解释为逻辑删除。
- 业务唯一性由数据库唯一约束兜底，不能仅靠先查再插入；索引依据过滤、关联、排序及数据量设计。
- 初始化 SQL 与升级 SQL 分开；IF NOT EXISTS 不会升级已有表。迁移需说明执行顺序、存量数据处理和影响，不能自动重建用户数据库。

## 8. 事务、并发和状态流转

涉及多表或批量原子性时，在 ServiceImpl 的 public 方法使用 `@Transactional(rollbackFor = Exception.class)`；普通查询不机械加事务。同类自调用不作为代理事务生效依据。

先确定业务不变量，再选择唯一约束、带预期状态的条件更新、乐观锁或行锁。不要把某个预约场景的锁顺序、隔离级别复制到所有 CRUD。

- 读后写会产生竞态时，把检查与写入放在可证明正确的事务边界内。
- 多资源加锁规定统一顺序，使用锁后最新状态；注意快照读、当前读和 MyBatis 缓存的差别。
- 需要调整隔离级别时，限定相关事务并检查外层事务传播，不默认全局改数据库隔离级别。
- JVM synchronized 和前端防重复点击不能替代跨进程数据库一致性。
- 状态变更验证当前阶段、目标阶段、操作者及时间条件；条件更新须检查影响行数，禁止任意跳转。
- 批量写入明确全成全败还是允许部分成功，并使响应与实现一致；必要时限制批次数量。
- 定时任务按业务时间判断，支持幂等和分批处理；由独立事务入口处理记录，不将任务实际执行时间误作业务事件生效时间。
- Clock 注入用于需要可控时间的业务测试。时区、时间精度和区间端点属于具体项目契约，需要明确后再实现。
- 并发测试验证最终数据库状态、记录数量和冲突响应，而不只断言没有异常。

## 9. 认证、权限及异常日志

### 账号与 JWT

- 密码使用 BCrypt 和独立随机盐；只保存哈希，使用 PasswordEncoder 验证，不继承参考项目的 MD5 写法。
- 密码长度策略由产品确定，并核对 BCrypt 的 UTF-8 字节上限；新策略需考虑旧账号兼容。
- 默认由 Spring Security 完成 JWT 验证与服务器授权；不为了模仿 sky-take-out 改成第二套拦截器认证。
- JWT 密钥由环境注入；算法、密钥格式及强度匹配，验证签名、有效期及配置的 issuer/audience。
- 新项目通常使用 Authorization Bearer；既有请求头和登录返回结构保持兼容。
- 注册角色与初始状态由后端控制。角色名称、权限来源及管理员规则按业务定义，不固定某个角色名或 ID。
- 服务端校验资源归属和数据范围，覆盖列表、详情、修改、批量操作及导出；不能只隐藏前端按钮。
- 如使用 BaseContext/ThreadLocal，请求结束及异常路径必须清理；异步任务不自动继承登录身份。
- 禁用账号、改密、退出是否立即撤销令牌须明确定义并实现，不把“前端删除 token”描述为服务端撤销。
- 刷新令牌、限流、验证码和多因素认证按需求增加，不声称未实现的能力已存在。

### 日志与错误处理

- Controller 使用 @Slf4j，记录业务动作、ID、数量和必要状态，使用占位符。
- 不打印完整认证 DTO、密码、哈希、JWT、数据库凭据或未经脱敏的敏感个人信息。
- 对有请求追踪的项目复用 requestId/MDC，响应头与错误体关联，并在 finally 清理。
- 集中使用 GlobalExceptionHandler；认证过滤器的错误也输出相同响应契约。
- 参数错误通常 400、未认证 401、无权限 403、不存在 404、业务冲突 409、限流 429、系统异常 500，具体兼容性按已有接口处理。
- 处理校验异常、非法 JSON、参数类型错误和数据库唯一冲突；不把 SQL 或堆栈返回前端。
- 未预料异常记录堆栈、返回通用提示；禁止空 catch、printStackTrace 后继续成功或返回空列表掩盖故障。

## 10. 前端实现规范

- 使用 Composition API、ref/reactive/computed 等管理状态；跨页面身份等共享状态使用 Pinia，局部表单不无故放全局。
- Axios 实例统一管理 baseURL、超时、认证头、Result 解包和错误转换；API 函数按资源组织，页面不重复拼请求封装。
- Result 解包只做一次。认证失败清理会话并跳转登录，区分登录失败与已登录会话失效。
- 令牌存储遵循既定方案。没有刷新接口就不要实现虚假的自动续期；敏感密钥绝不进入前端。
- 路由按需懒加载；管理端守卫按后端身份控制界面访问，真正权限仍在服务器。
- 复用布局、页面标题、错误提示、表格、详情及表单组件；仅有一个使用点的简单逻辑不强行组件化。
- 每个异步页面有 loading、空数据、错误与重试状态；提交期间防重复提交，失败保留输入，成功刷新真实数据。
- 切换查询条件要防止旧请求覆盖新结果；异步弹窗明确本次操作目标，不误写切换后的资源。
- 禁用、删除等有影响动作提供适当确认；用户取消弹窗不当作错误提示。
- 前端校验用于及时反馈，后端必须重复执行安全和业务校验。
- 日期格式、时区、ID 类型与后端契约一致，不随意通过 toISOString 改变本地业务时间。
- 正式页面从真实 API 获取数据；模拟数据只用于测试或明确标注的演示，不提供无法保存的假入口。
- 样式沿用项目设计变量、间距、组件和响应式规则；新项目先确定视觉基线，不固化某个业务项目的颜色和布局。
- 需要图表时使用 ECharts，按需注册，处理空数据、容器 resize 和卸载 dispose；统计口径与后端一致，不编造数据填图。

## 11. 配置、启动和交付

- 数据库账号、密码、JWT 密钥等通过环境或部署配置注入；示例文件只提供变量名及非敏感占位值。
- 不假设 Spring Boot 自动加载根目录 .env；说明实际 IDE、启动脚本或配置机制。
- CORS 按环境集中配置，localhost 与 127.0.0.1 是不同来源；不能通过关闭认证解决 403。
- 前端开发代理与正式部署代理分开。VITE_ 变量会进入浏览器，不放后端秘密。
- 端口、数据库名和项目路径由工程决定，不把某台机器的绝对路径写入应用。
- 启动或联调前识别现有进程，避免盲目停止用户服务。重新编译不代表旧运行进程已加载新代码。
- 正式部署说明后端配置、前端 dist、SPA 路由回退、API 反向代理及 HTTPS/来源配置；不能把本地构建通过说成已部署。

## 12. 测试与执行流程

### 开始任务

1. 识别工程根目录、适用规则、用户目标及已有改动。
2. 读取 POM、package.json、相关配置名称和目标业务完整调用链；不要输出真实凭据。
3. 确定输入输出、权限、默认值、失败路径、业务状态、事务和数据变更；不凭页面猜 API。
4. 复用已有 Result、异常处理、请求封装和组件，确定最小改动范围。

### 实现与验证

1. 按需要完成 Entity、DTO/VO、Mapper、Service/Impl、Controller、前端、SQL 和文档。
2. 检查接口签名、序列化字段、参数校验、状态码、权限及空值行为的一致性。
3. 按风险执行编译、业务测试、浏览器测试和构建；文档或低影响机械修改不堆砌无意义测试。

典型命令（在各自实际工程目录执行，脚本是否存在以工程为准）：

```text
java -version
mvn -v
mvn test
mvn package

npm ci
npm test
npm run test:e2e
npm run build
```

覆盖本次相关的正常操作、非法参数、重复数据、不存在记录、关联限制、未登录、无权限、越权、事务回滚和并发冲突。分页、文件或定时任务仅在涉及这些功能时增加相应验证。

优先隔离测试数据库；H2 可用于适用的集成测试，但不能证明 MySQL 锁、索引、约束及隔离行为完全相同。关键数据库语义要在对应数据库验证。模拟 API 浏览器测试不能替代真实前后端联调。

造数、压测和清理须明确目标环境与授权范围，不默认向业务库批量写入或删除；报告中不保存可复用令牌或凭据。

### 完成交付

说明实现了什么、关键文件、接口/配置/SQL 的变化、实际执行的验证及结果，以及需要重启或尚未验证的部分。区分“代码完成”“测试通过”“联调通过”“已部署”，不以历史测试记录替代本次证据。

本技能可跨业务复用；具体业务流程、角色矩阵、状态机、表结构和领域约束应进入对应项目文档，不继续追加成通用技能的固定前提。
