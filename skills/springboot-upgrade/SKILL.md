---
name: springboot-upgrade
description: Spring Boot全版本升级。提供定制化的Spring Boot版本升级方案（2.x→3.x、3.x→4.x等）和JDK升级支持。包括Maven依赖调整、项目代码兼容性修复和编译验证。当用户提到"Spring Boot升级"、"JDK升级"、"Spring框架迁移"或版本相关的依赖冲突问题时使用此技能。
license: Complete terms in LICENSE.txt
---

# Spring Boot 全版本升级

## 使用时机

当用户提到以下场景时，立即使用此技能：
- Spring Boot版本升级（如"升级到Spring Boot 3.5"、"从2.7迁移到3.x"）
- JDK版本升级（如"升级到JDK 21"、"支持Java 21"）
- Spring框架迁移场景（如"2.x升级到3.x"）
- 版本升级导致的依赖冲突或编译错误解决

## 升级工作流

### 第一步：信息收集与确认

与用户交互，获取升级的核心参数。如果用户信息不完整，主动提问：

```
【当前状态】
- 当前 Spring Boot 版本：【填写版本】
- 当前 JDK 版本：【填写版本】

【升级目标】
- 目标 Spring Boot 版本：【填写版本】
- 目标 JDK 版本：【填写版本】
```

**内部包处理**：扫描项目中的自定义内部包（如 `frm-art2-spring-boot-starter`、`config-agent`、`spring-cloud-stream-binder-rabbit` 等），在升级方案中标记这些包，提醒用户补充版本号，标记为 `【填写版本号】`。

### 第二步：Maven 依赖升级方案

**目标**：修改 pom.xml 完成依赖适配，为代码修改打下基础。

#### 1. 基础配置调整（必做）

- **升级Spring Boot父依赖版本**
- **调整JDK编译配置**

#### 2. 核心依赖版本升级（按目标版本自动匹配）

根据升级目标应用下表的版本规则：

| 依赖类型 | SB2.x | SB3.x | SB4.x |
|---------|--------|--------|--------|
| Spring 核心（web/mvc） | 5.x | 6.x | 7.x |
| Tomcat 服务器 | 9.x | 10.1.x | 10.1.x+ |
| Hibernate Validator | 6.x | 8.x | 8.x+ |
| Logback 日志 | 1.2.x | 1.5.x | 1.6.x+ |

**匹配规則**：
- SB 2.x → SB 3.x：Spring 5.x → 6.x，其他依赖按表匹配
- SB 3.x → SB 4.x：Spring 6.x → 7.x，其他依赖按表匹配
- 同大版本升级（如3.5.0→3.5.7）：保持大版本不变

#### 3. 内部包处理

扫描所有内部包，在升级计划中提醒用户填写版本号，格式：`【填写版本号】`。

示例：
- `frm-art2-spring-boot-starter` → 版本 `【填写版本号】`
- `config-agent` → 版本 `【填写版本号】`
- 其他自定义starter包

#### 4. 依赖兼容性检查与冲突解决

**检查冲突**：
```powershell
mvn dependency:tree | Out-File -FilePath dependency-tree.txt -Encoding UTF8
mvn dependency:analyze
mvn dependency:tree -Dverbose | Select-String -Pattern "conflict" -Context 1,1
```

**通用解决方法**：
- 排除旧版本依赖（使用 `<exclusion>`）
- 统一版本管理（在 pom.xml 中的 `<dependencyManagement>` 管理版本）
- 升级至合适的兼容版本

### 第三步：项目代码修改（最小化修正，保留业务逻辑）

**目标**：通过「编译检查→代码修正→重复验证」闭环，解决所有代码适配问题。

#### 1. 编译检查

```powershell
mvn clean compile
```

获取所有编译错误信息。

#### 2. 根据错误类型修复代码

常见错误处理方案详见 [编译错误参考](./references/compile-errors.md)：
- 反射访问限制报错
- Deprecated API 过时 API 替换
- Spring Security 相关错误
- HttpClient 版本升级
- 其他常见错误

#### 3. 编译验证闭环

遵循以下流程直至成功：
1. 修复一组错误
2. 执行 `mvn clean compile`
3. 检查是否有新错误
4. 如有错误，返回第1步；无错误则进行第四步

### 第四步：最终升级验证

#### 编译验证

```powershell
✅ mvn clean compile 无报错
```

#### 可选验证（如有测试代码）

```powershell
✅ mvn test
```
确保单元测试100%通过。

#### 应用验证（可选）

```powershell
✅ 启动项目，测试1-2个核心接口可正常访问
```

## 输出格式规范

每次升级方案应包含：

1. **核心参数总结**（开头）
   ```
   【升级参数】
   当前版本：Spring Boot 2.x.x + JDK x
   目标版本：Spring Boot 3.x.x + JDK x
   ```

2. **分模块呈现**
   - Maven 依赖升级（必做项清单）
   - 内部包版本确认（用户需补充）
   - 项目代码修改（按错误类型）
   - 最终升级验证检查表

3. **代码示例**
   - 提供具体的 pom.xml 修改示例
   - 给出常见代码修改的示例

4. **问题排查提示**
   - 引导用户补充报错日志
   - 对特殊场景进行深度分析

## 核心原则

1. **定制化** - 根据当前版本→目标版本的具体路径，提供精准的升级方案
2. **最小化修改** - 只修改升级必需的部分，保留业务逻辑不动
3. **验证闭环** - 每次修改后立即编译验证
4. **版本一致性** - 使用 `<dependencyManagement>` 统一管理版本

## 技能边界

1. 专注 Spring Boot 升级的技术适配，不涉及业务逻辑重构
2. 内部自定义包需用户主动提供后续版本信息
3. 特殊场景（非通用错误）需用户补充详细报错日志后进一步分析
