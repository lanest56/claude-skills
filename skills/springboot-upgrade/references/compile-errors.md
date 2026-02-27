# Spring Boot 升级常见编译错误速解指南

此文档包含 Spring Boot 版本升级过程中常见的编译错误及其解决方案，按错误类型组织。

## 目录

1. [反射访问限制报错](#反射访问限制报错)
2. [Deprecated API 过时API替换](#deprecated-api-过时api替换)
3. [Spring Security 6 相关错误](#spring-security-6-相关错误)
4. [HttpClient 版本升级](#httpclient-版本升级)
5. [其他常见错误](#其他常见错误)

---

## 反射访问限制报错

### 症状

错误信息类似于：
```
WARNING: An illegal reflective access operation has occurred
java.lang.reflect.InaccessibleObjectException: Unable to make member accessible
```

### 原因

Java 9+ 引入了模块化系统，对反射访问进行了限制。某些库（特别是老版本的Spring、Hibernate等）使用反射访问内部类。

### 解决方案

**方法1：添加JVM参数（临时方案）**
```powershell
# 在启动Java应用时添加
java -jar app.jar --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.util=ALL-UNNAMED
```

**方法2：修改Maven SUREFIRE配置（推荐用于测试）**
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <argLine>
            --add-opens java.base/java.lang=ALL-UNNAMED
            --add-opens java.base/java.util=ALL-UNNAMED
        </argLine>
    </configuration>
</plugin>
```

**方法3：升级相关库到最新版本（最优解）**
- 升级Spring Boot到最新版本
- 升级Hibernate Validator到8.x+
- 升级其他依赖到支持新JDK的版本

---

## Deprecated API 过时API替换

### Spring Web MVC 相关（Spring 5.x → 6.x）

#### 错误1：`WebMvcConfigurerAdapter` 已删除

**症状**
```
WebMvcConfigurerAdapter is deprecated and marked for removal
```

**解决方案**

旧代码（Spring 5.x）：
```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addCors(CorsRegistry registry) {
        // 配置CORS
    }
}
```

新代码（Spring 6.x）：
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCors(CorsRegistry registry) {
        // 配置CORS
    }
}
```

或使用基于Bean的配置：
```java
@Configuration
public class WebConfig {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCors(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }
        };
    }
}
```

#### 错误2：`HttpMessageConverter` 方法签名变化

**症状**
```
The method canWrite(Class, MediaType) from WebMvcConfigurer is deprecated
```

**解决方案**
在Spring 6.x中，某些方法签名已更新。查阅[Spring 6.0迁移指南](https://spring.io/blog/2022/09/23/spring-6-0-goes-ga)并更新相应代码。

---

## 其他常见错误

### XML 配置相关

#### 错误：`spring-boot-starter-web` 中缺少 servlet-api

**症状**

```
Cannot find symbol: class HttpServlet
```

**解决方案**
确保 `spring-boot-starter-web` 或 `spring-boot-starter-webflux` 已添加到pom.xml：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### Jackson 相关错误

#### 错误：Jackson 版本不兼容，JSON 序列化失败

**症状**
```
com.fasterxml.jackson.databind.exc.InvalidDefinitionException
```

**解决方案**
在 `<dependencyManagement>` 中统一 Jackson 版本，确保与 Spring Boot 版本一致（通常无需手动指定，由 Spring Boot BOM 管理）。

---

## 编译错误修复流程

1. **执行编译，记录所有错误**
   ```powershell
   mvn clean compile > compile-errors.log 2>&1
   ```

2. **按错误类型分类**
   - 反射访问类
   - Deprecated API 类
   - 依赖缺失类
   - 其他类

3. **查阅本指南，按类型应用解决方案**

4. **修改代码，再次编译**
   ```powershell
   mvn clean compile
   ```

5. **重复步骤2-4，直至编译通过**

---

## 参考资源

- [Spring Boot 2.7 → 3.0 迁移指南](https://spring.io/blog/2022/08/24/spring-boot-3-0-0-m2-is-now-available)
- [Spring Security 6.0 迁移指南](https://spring.io/blog/2022/02/21/spring-security-without-the-servlet-api/)
- [Apache HttpClient 5.x 迁移指南](https://hc.apache.org/httpcomponents-client-5.0/migration-guide/)
- [JDK 11+ 迁移指南](https://docs.oracle.com/javase/11/migrate/index.html)
