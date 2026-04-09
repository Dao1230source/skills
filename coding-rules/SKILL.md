---
name: coding-rules
description: 全局代码编写规范，包含Sonar、Alibaba Java Guide规则，commons-lang3/commons-collections4判空处理，以及项目模块使用规范（Assign批量赋值、Tree树结构、JPA Extension）
license: MIT
compatibility: opencode
metadata:
  author: zengfugen
  version: 3.0.0
  language: Java
---

## What I do（我做什么）

我提供全局的代码编写规范和最佳实践，确保所有生成的代码符合企业级质量标准。

## When to use me（何时使用）

在以下场景使用我：

- 编写任何 Java 代码时
- 进行代码审查时
- 需要确保代码质量符合 Sonar 和 Alibaba 规范时
- 需要处理对象判空逻辑时
- 需要参考标准项目结构时
- **编写 README 文档时**
- **使用 Assign 模块进行批量赋值时**
- **使用 Tree 模块处理树形结构时**
- **使用 jpa-extension-starter 进行 JPA 数据访问时**

**触发关键词**：

- "代码规范"、"编码规范"、"编码"、"编码规则"
- "Sonar"、"Alibaba Java Guide"
- "判空"、"commons-lang3"、"commons-collections4"
- "项目结构"、"代码架构"
- **"README"、"文档编写"、"项目文档"**
- **"批量赋值"、"Assign"、"Acquire"**
- **"Tree"、"树形结构"、"树"、"DAG"、"深度计算"、"扁平化"**
- **"JPA"、"Repository"、"CRUD"、"批量操作"、"逻辑删除"、"权限控制"**

---

## 第一部分：代码规范要求

### 1.1 代码质量标准

所有代码必须遵循以下规范：

- **Sonar 质量规则**: 无 blocker、critical、major 级别问题
- **Alibaba Java Coding Guidelines**: 遵循阿里巴巴 Java 开发手册
- **圈复杂度**: 单个方法不超过 15
- **方法长度**: 不超过 60 行

### 1.2 命名规范

- **变量/方法**: 驼峰命名法 (camelCase)
- **常量**: 大写蛇形命名 (UPPER_SNAKE_CASE)
- **类名**: 大驼峰命名法 (PascalCase)

### 1.3 Lombok 注解

所有对象的 getter/setter 必须使用 Lombok 注解实现：

```java

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Demo {

}
```

### 1.4 @Data 注解与继承

**当 @Data 注解使用在继承父类的子类上时，必须添加 @EqualsAndHashCode(callSuper = true)：**

```java
// 错误 ❌ - 编译器警告
@Data
public class User extends BaseEntity {
    private String name;
}

// 正确 ✅
@EqualsAndHashCode(callSuper = true)
@Data
public class User extends BaseEntity {
    private String name;
}
```

### 1.5 @NonNull 注解（继承接口）

**当继承 `org.source.utility.tree.define.Element` 或类似接口时，需要在重写方法上添加 @NonNull：**

```java
// 错误 ❌ - 编译提示 Not annotated method overrides method annotated with @NonNullApi
@Data
public class MyElement extends EnhanceElement<String> {
    @Override
    public String getId() {  // 需要添加 @NonNull
        return "id";
    }
}

// 正确 ✅
@EqualsAndHashCode(callSuper = true)
@Data
public class MyElement extends EnhanceElement<String> {
    @Override
    public @NonNull String getId() {  // 添加 @NonNull
        return "id";
    }
}
```

> **注意**：需要 import `org.springframework.lang.NonNull`

### 1.6 日志规范

- 使用 SLF4J 的 `@Slf4j` Lombok 注解
- **禁止使用** `System.out.println()`
- 正确日志级别：DEBUG < INFO < WARN < ERROR

### 1.5 异常处理

- 必须有明确的业务含义
- 禁止吞掉异常（未记录日志的 catch 块）
- 使用自定义业务异常继承 `BaseException`

### 1.6 文档代码示例规范

**所有说明文档（README、SKILL.md 等）中的 Java 代码示例必须添加方法名或类名，避免 IDEA 打开时报错。**

#### 错误示例 ❌

```java
// 错误：缺少方法包装，且 if 语句没有大括号
if(CollectionUtils.isNotEmpty(list))
        processList(list);
```

#### 正确示例 ✅

```java
public void collectionsUseCase() {
    // 集合判空
    if (CollectionUtils.isNotEmpty(list)) {
        // 处理非空集合
    }
}
```

#### 常见代码块类型

| 类型   | 包装方式                                     |
|------|------------------------------------------|
| 方法代码 | `public void methodName() { ... }`       |
| 类定义  | `public class ClassName { ... }`         |
| 接口定义 | `public interface InterfaceName { ... }` |
| 枚举定义 | `public enum EnumName { ... }`           |

### 1.7 控制语句规范

**所有控制语句（if/else/for/while/do-while）必须使用大括号，即使只有一行代码：**

```java
// 错误 ❌ - 缺少大括号
public void badExample() {
    if (condition) return;
    
    if (isValid)
        doSomething();
        
    for (int i = 0; i < 10; i++)
        process(i);
}

// 正确 ✅ - 使用大括号
public void goodExample() {
    if (condition) {
        return;
    }
    
    if (isValid) {
        doSomething();
    }
    
    for (int i = 0; i < 10; i++) {
        process(i);
    }
}
```

> **原因**：
> 1. 避免后续维护时添加代码导致逻辑错误
> 2. 符合 Sonar 规则 java:S121（控制语句应该使用大括号）
> 3. 符合阿里巴巴 Java 开发手册规范

### 1.8 注释规范（强制要求）

**所有公共类和公共方法必须有完整的 JavaDoc 注释：**

#### 类注释要求

```java
// 错误 ❌ - 缺少注释
public class UserService {
    // ...
}

// 正确 ✅ - 完整的类注释
/**
 * 用户服务类
 * <p>
 * 提供用户相关的业务逻辑处理，包括：
 * <ul>
 *     <li>用户创建和更新</li>
 *     <li>用户查询和删除</li>
 *     <li>用户权限管理</li>
 * </ul>
 * </p>
 *
 * @author dao1230source
 * @since 1.0.0
 */
public class UserService {
    // ...
}
```

#### 方法注释要求

```java
// 错误 ❌ - 缺少注释
public User getUserById(Long id) {
    return userRepository.findById(id);
}

// 正确 ✅ - 完整的方法注释
/**
 * 根据ID获取用户信息
 * <p>
 * 从数据库查询用户信息，如果用户不存在则返回 null
 * </p>
 *
 * @param id 用户ID，不能为 null
 * @return 用户对象，如果不存在则返回 null
 * @throws IllegalArgumentException 如果 id 为 null
 */
public User getUserById(Long id) {
    if (id == null) {
        throw new IllegalArgumentException("用户ID不能为空");
    }
    return userRepository.findById(id);
}
```

#### 字段注释要求

```java
// 错误 ❌ - 缺少注释
@Data
public class User {
    private String name;
    private Integer age;
}

// 正确 ✅ - 完整的字段注释
@Data
public class User {
    /**
     * 用户姓名
     */
    private String name;
    
    /**
     * 用户年龄
     */
    private Integer age;
}
```

#### JavaDoc 标签规范

| 标签 | 使用场景 | 说明 |
|------|----------|------|
| `@author` | 类、接口 | 作者信息（必须） |
| `@since` | 类、接口、方法 | 版本信息（必须） |
| `@param` | 方法参数 | 参数说明 |
| `@return` | 方法返回值 | 返回值说明 |
| `@throws` | 方法异常 | 异常说明 |
| `@see` | 相关引用 | 相关类、方法链接 |

#### 注释内容规范

1. **禁止使用尾行注释**（代码行末尾的注释）
2. **注释必须有意义**：说明用途、含义、注意事项，不要重复代码内容
3. **使用 `<p>` 分段**：长注释使用 `<p>` 标签分段，提高可读性
4. **使用 `<ul>/<li>` 列表**：列举多项内容时使用列表标签
5. **注释更新**：代码修改时同步更新注释

#### 尖括号类型注释规范（强制要求）

**当java注释中出现泛型类型（如 `List<V>`、`Map<K,V>`、`List<List<V>>`）时，必须使用 `{@literal }` 包裹：**

```java
// 错误 ❌ - 尖括号未包裹，HTML转义问题
/**
 * 批量key返回 Map<K, List<V>>类型
 */

// 正确 ✅ - 使用 {@literal } 包裹
/**
 * 批量key返回 {@literal Map<K, List<V>>}类型
 */
```

**常见类型示例**：

| 类型 | 注释写法 |
|------|---------|
| `List<V>` | `{@literal List<V>}` |
| `Map<K,V>` | `{@literal Map<K,V>}` |
| `Map<K, List<V>>` | `{@literal Map<K, List<V>>}` |
| `Map<K, Map<KK, V>>` | `{@literal Map<K, Map<KK, V>>}` |
| `List<List<V>>` | `{@literal List<List<V>>}` |

```java
// 错误 ❌ - 尾行注释
private String name; // 用户姓名

// 正确 ✅ - JavaDoc 注释
/**
 * 用户姓名
 */
private String name;

// 错误 ❌ - 无意义注释
/**
 * 设置名称
 */
public void setName(String name) {
    this.name = name;
}

// 正确 ✅ - 有意义的注释
/**
 * 设置用户姓名
 * <p>
 * 姓名长度限制为 2-50 个字符，超出范围会被截断
 * </p>
 *
 * @param name 用户姓名，不能为 null 或空字符串
 */
public void setName(String name) {
    if (StringUtils.isBlank(name)) {
        throw new IllegalArgumentException("用户姓名不能为空");
    }
    this.name = name.substring(0, Math.min(name.length(), 50));
}
```

> **强制要求**：
> - 所有 public 类必须有 JavaDoc 类注释（含 `@author` 和 `@since`）
> - 所有 public 方法必须有 JavaDoc 方法注释（含 `@param` 和 `@return`）
> - 所有字段必须有 JavaDoc 字段注释
> - 符合 Sonar 规则 java:S1135（跟踪注释中的 TODO）、java:S1172（未使用的参数）

### 1.9 未使用的导入语句（强制要求）

**禁止保留未使用的导入语句：**

#### 手动删除

```java
// 错误 ❌ - 包含未使用的导入
import java.util.ArrayList;
import java.util.List;
import java.util.Date;  // 未使用
import java.io.IOException;  // 未使用

public class UserService {
    public List<String> getUserNames() {
        return new ArrayList<>();
    }
}

// 正确 ✅ - 只保留使用的导入
import java.util.ArrayList;
import java.util.List;

public class UserService {
    public List<String> getUserNames() {
        return new ArrayList<>();
    }
}
```

#### 使用 IDE 自动优化

**IntelliJ IDEA**:
1. 快捷键：`Ctrl + Alt + O` (Windows/Linux) 或 `Cmd + Option + O` (macOS)
2. 右键菜单：`Optimize Imports`
3. 自动优化：Settings → Editor → General → Auto Import → Optimize imports on the fly

**Eclipse**:
1. 快捷键：`Ctrl + Shift + O`
2. 右键菜单：`Source → Organize Imports`

> **强制要求**：
> - 提交代码前必须删除未使用的导入语句
> - 符合 Sonar 规则 java:S1148（未使用的导入应该删除）
> - 符合阿里巴巴 Java 开发手册规范
> - 保持代码整洁，避免混淆

### 1.10 未使用的代码块（强制要求）

**禁止保留未使用的代码块、方法、类、变量：**

#### 未使用的方法

```java
// 错误 ❌ - 包含未使用的方法
public class UserService {
    public User getUserById(Long id) {
        return userRepository.findById(id);
    }
    
    // 未使用的方法
    private void oldMethod() {
        // 旧的实现，已被删除或重构
    }
}

// 正确 ✅ - 删除未使用的方法
public class UserService {
    public User getUserById(Long id) {
        return userRepository.findById(id);
    }
}
```

#### 未使用的变量和常量

```java
// 错误 ❌ - 包含未使用的变量和常量
public class Constants {
    public static final String OLD_CONSTANT = "old_value";  // 未使用
    private String unusedField;  // 未使用
}

// 正确 ✅ - 删除未使用的变量和常量
public class Constants {
    // 只保留使用的常量
}
```

#### 未使用的内部类

```java
// 错误 ❌ - 包含未使用的内部类
public class OuterClass {
    public void doSomething() {
        // ...
    }
    
    // 未使用的内部类
    private static class OldImplementation {
        // 旧的实现
    }
}

// 正确 ✅ - 删除未使用的内部类
public class OuterClass {
    public void doSomething() {
        // ...
    }
}
```

#### 检查工具

**IntelliJ IDEA**:
1. 代码检查：`Analyze → Inspect Code`
2. 查看结果：`Unused declaration` 节点
3. 快速修复：右键 → `Safe Delete`

**SonarQube**:
- 规则 `java:S1144` - Unused private methods should be removed
- 规则 `java:S1848` - Unused private fields should be removed
- 规则 `java:S2094` - Unused classes should be removed

> **强制要求**：
> - 提交代码前必须删除未使用的方法、类、变量、常量
> - 定期运行代码检查，清理未使用的代码
> - 保持代码整洁，避免死代码积累

### 1.12 README 文档编写规范（强制要求）

**所有项目 README.md 文档必须遵循以下规范，以便于被人类和 AI 搜索引擎识别和使用：**

#### 文档结构规范

**必须包含的章节**（按顺序排列）：

1. **项目标题**：包含项目名称、徽章（License、Maven Central、Java Version、Spring Boot Version）
2. **简介**：一句话描述项目核心功能 + 关键词列表
3. **核心特性表格**：以表格形式列出核心功能及其关键词
4. **适用场景**：明确列出项目适用场景（使用 ✅ 符号）
5. **快速导航**：提供文档章节快速链接
6. **环境要求表格**：列出技术栈及版本要求
7. **安装配置**：提供 Maven 和 Gradle 两种配置方式
8. **快速开始**：提供完整的代码示例（Entity → Repository → Service）
9. **核心功能详解**：每个功能都有详细的说明、表格、代码示例
10. **API 参考**：列出核心接口、类及其方法
11. **最佳实践**：提供 ✅ 正确示例和 ❌ 错误示例对比
12. **性能说明表格**：列出性能对比数据
13. **常见问题 FAQ**：Q&A 格式，至少 5 个常见问题
14. **项目结构**：以树形结构展示项目目录
15. **许可证**：明确声明许可证类型
16. **作者信息**：包含姓名、邮箱、组织、GitHub
17. **相关链接**：GitHub Repository、Issue Tracker、Documentation
18. **支持与反馈**：提供问题反馈流程

#### 关键词优化规范

**必须包含关键词部分**（便于搜索引擎识别）：

```markdown
**关键词**: Spring Data JPA, JPA Enhancement, Repository Pattern, Batch Operations, Logic Delete, Permission Control
```

**关键词要求**：
- 至少包含 5-10 个关键词
- 包含技术栈关键词（如 Spring Boot、Java、JPA）
- 包含核心功能关键词（如 Batch Insert、Logic Delete）
- 包含使用场景关键词（如 CRUD、Repository Pattern）
- 英文关键词为主，中文关键词为辅

#### 代码示例规范

**所有代码示例必须**：

1. **添加完整类名或方法名**：避免 IDEA 打开时报错
2. **包含注释说明**：关键代码必须有注释
3. **提供 ✅ 正确示例和 ❌ 错误示例对比**：最佳实践章节必须有对比
4. **标注性能数据**：性能相关代码必须标注时间复杂度

**错误示例 ❌**：

```java
// 缺少方法包装，IDEA 报错
User user = helper.add(user);
```

**正确示例 ✅**：

```java
/**
 * 新增用户
 */
public User addUser(User user) {
    return add(user);  // 自动检查 @CheckExists 字段
}
```

#### 表格使用规范

**必须使用表格的场景**：

1. **核心特性表格**：列出功能及其关键词
2. **环境要求表格**：列出技术栈及版本
3. **API 方法表格**：列出方法及其说明
4. **性能对比表格**：列出不同方法的性能数据
5. **节点类型表格**：列出不同类型及其特点

**表格格式**：

```markdown
| 特性 | 描述 | 关键词 |
|------|------|--------|
| **批量操作** | 支持 MySQL ON DUPLICATE KEY UPDATE | Batch Insert, Bulk Operations |
```

#### 性能说明规范

**必须包含性能对比表格**：

```markdown
| 数据量 | save() 循环 | saveAll() | onDuplicateUpdateBatch() |
|--------|--------------|-----------|--------------------------|
| 100 条 | ~1s | ~0.1s | **~0.01s** |
| 1000 条 | ~10s | ~1s | **~0.1s** |
```

**性能标注要求**：
- 使用粗体 `**text**` 标注最优方案
- 提供具体性能数据（时间或时间复杂度）
- 至少对比 3 种不同方法

#### FAQ 规范

**FAQ 章节必须包含**：

- **至少 5 个常见问题**：Q1 到 Q5
- **Q&A 格式**：问题用 `### Q1:` 格式，答案用 `**答:**:` 格式
- **代码示例**：每个答案必须包含代码示例
- **标签说明**：每个问题包含标签（如 [性能]、[批量操作]）

**FAQ 格式示例**：

```markdown
### Q1: 如何实现自定义查询？

**答**: 使用 Condition 构造器或 Specification 接口：

```java
public List<User> customQuery() {
    Condition<User> condition = new Condition<User>()
        .eq(User::getUsername, "john");
    return findAll(condition);
}
```
```

#### 徽章使用规范

**必须包含以下徽章**：

```markdown
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Maven Central](https://img.shields.io/maven-central/v/io.github.dao1230source/jpa-extension-starter.svg)](https://search.maven.org/)
[![Java](https://img.shields.io/badge/Java-17+-green.svg)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen.svg)](https://spring.io/)
```

#### README 检查清单

**提交前必须检查以下内容**：

- ✅ 项目标题包含徽章
- ✅ 包含关键词部分
- ✅ 核心特性使用表格展示
- ✅ 环境要求使用表格展示
- ✅ 提供 Maven 和 Gradle 配置
- ✅ 快速开始包含完整示例（Entity → Repository → Service）
- ✅ 所有代码示例包含方法名或类名
- ✅ 最佳实践包含 ✅ 正确示例和 ❌ 错误示例对比
- ✅ 包含性能对比表格
- ✅ FAQ 至少包含 5 个问题
- ✅ 包含项目结构树形图
- ✅ 包含作者信息和相关链接

> **强制要求**：
> - 所有项目必须有 README.md 文档
> - README 必须遵循以上规范
> - 定期更新 README，保持内容准确
> - 关键词部分便于 AI 搜索引擎识别

---

## 第二部分：判空处理规范

### 2.1 commons-lang3 对象判空

```java
public void langUseCase() {
    // 字符串判空
    if (StringUtils.isNotBlank(str)) {
        // 处理非空字符串
    }

    // 数组判空
    if (ArrayUtils.isNotEmpty(array)) {
        // 处理非空数组
    }

    // 对象判空
    if (ObjectUtils.allNotNull(obj1, obj2)) {
        // 所有对象都不为null
    }

    // 布尔值处理
    if (BooleanUtils.isTrue(flag)) {
        // 处理true值
    }
}
```

### 2.2 commons-collections4 集合判空

```java
public void collectionsUseCase() {
    // 集合判空
    if (CollectionUtils.isNotEmpty(list)) {
        // 处理非空集合
    }

    // Map判空
    if (MapUtils.isNotEmpty(map)) {
        // 处理非空Map
    }

    // 安全判空
    boolean isEmpty = CollectionUtils.isEmpty(collection);
}
```

---

## 第三部分：项目模块规范

### 3.1 Assign 批量赋值模块

> **模块位置**: `org.source.utility.assign`
> **完整文档**: [reference/Assign.md](reference/Assign.md)

#### 核心类与职责

| 类                  | 职责                                     |
|--------------------|----------------------------------------|
| **Assign<E>**      | 顶层构建与编排入口，管理主数据、acquires、branches、subs |
| **Acquire<E,K,T>** | 外部数据获取（批量或单条），可配置缓存、分批、异常处理            |
| **Action<E,K,T>**  | 从主对象取 key 并执行 Assemble（赋值）             |
| **Assemble<E,T>**  | 最小赋值单元，BiConsumer<E, T> 语义             |

#### 基础用法

```java
public void assignBasic() {
    // 1. 构建 Assign
    Assign.build(orderList)
            // 2. 添加 Acquire（数据获取）
            .addAcquire(this::findEmployeesByEmpCodes, EmployeeDTO::getEmpCode)
            // 3. 添加 Action（指定取值字段）
            .addAction(OrderDTO::getEmpCode)
            // 4. 添加 Assemble（赋值操作）
            .addAssemble(EmployeeDTO::getEmpName, OrderDTO::setEmpName)
            // 5. 返回并执行
            .backAcquire().backAssign().invoke();
}
```

#### Assign API 速查

| 方法                                 | 说明         |
|------------------------------------|------------|
| `parallel()` / `parallelVirtual()` | 并行执行（虚拟线程） |
| `addAcquire(fetcher, keyGetter)`   | 批量获取数据     |
| `batchSize(size)`                  | 分批大小       |
| `cache()`                          | 启用缓存       |
| `addBranch(filter)`                | 条件分支       |
| `invoke()`                         | 执行         |

#### 典型场景

**多级依赖：**

```java
public void assignMultiLevel() {
    Assign.build(orderList)
            // 第一级：获取员工信息
            .addAcquire(this::findEmployees, EmployeeDTO::getEmpCode)
            .addAction(OrderDTO::getEmpCode)
            .addAssemble(EmployeeDTO::getEmpName, OrderDTO::setEmpName)
            .backAcquire().backAssign().invoke()
            // 第二级：根据网点编码获取网点信息
            .addAcquire(this::findNets, NetDTO::getNetCode)
            .addAction(OrderDTO::getNetCode)
            .addAssemble(NetDTO::getNetName, OrderDTO::setNetName)
            .backAcquire().backAssign().invoke();
}
```

**虚拟线程并行 + 分批：**

```java
public void assignVirtualThread() {
    Assign.build(orderList)
            .parallelVirtual()
            .addAcquire(this::findEmployees, EmployeeDTO::getEmpCode)
            .batchSize(100)
            .cache()
            .addAction(OrderDTO::getEmpCode)
            .addAssemble(EmployeeDTO::getEmpName, OrderDTO::setEmpName)
            .backAcquire().backAssign().invoke();
}
```

#### Assign 注意事项

1. **命名规范**：为 Assign 和 Acquire 设置有意义的 name，便于日志追踪与缓存识别
2. **批量优先**：优先使用 `addAcquire` 批量接口，减少远程调用次数
3. **分批与限流**：对大批量使用 `batchSize`，对虚拟线程场景设置 `semaphorePermitsMax`
4. **缓存策略**：缓存 key 格式为 `{assignName}_{acquireName}`

---

### 3.2 Tree 树形结构模块

> **模块位置**: `org.source.utility.tree`
> **完整文档**: [reference/Tree.md](reference/Tree.md)

#### 核心概念

- **I** - 唯一键，元素的唯一标识（String、Integer等）
- **E extends Element<I>** - 挂载在节点上的元素，树中实际存储的业务数据
- **N extends AbstractNode<I, E, N>** - 节点，包含元素和关系信息

#### 节点类型选择

| 节点类型            | 使用场景           | 特点            |
|-----------------|----------------|---------------|
| **DefaultNode** | 简单树结构（推荐99%场景） | 最高性能          |
| **DeepNode**    | 需要深度计算         | 自动计算节点深度      |
| **EnhanceNode** | 多父节点DAG        | 支持符号链接、权限管理   |
| **FlatNode**    | JSON扁平化输出      | RESTful API响应 |

#### 基础用法

```java
public void treeBasic() {
    // 1. 创建树（选择合适的节点类型）
    Tree<Integer, DeptElement, DefaultNode<Integer, DeptElement>> tree =
            Tree.of(new DefaultNode<>());

    // 2. 设置处理器计算层级
    tree.setAfterAddHandler((node, parent) -> {
        node.getElement().setLevel(
                parent.getElement() == null ? 0 :
                        parent.getElement().getLevel() + 1
        );
    });

    // 3. 添加数据
    List<DeptElement> depts = Arrays.asList(
            new DeptElement(1, null, "公司"),       // 根节点
            new DeptElement(2, 1, "技术部"),        // 子节点
            new DeptElement(3, 1, "市场部")
    );
    tree.add(depts);

    // 4. 查询数据
    DefaultNode<Integer, DeptElement> node = tree.getById(2);  // O(1) 按ID查询
}
```

#### Tree API 速查

| 方法                  | 时间复杂度    | 说明                |
|---------------------|----------|-------------------|
| `of(root)`          | O(1)     | 工厂方法，创建树实例        |
| `add(elements)`     | O(n·m)   | 批量添加元素            |
| `find(predicate)`   | O(n)     | 查找符合条件的所有节点       |
| `getById(id)`       | **O(1)** | 按ID查询，最快！         |
| `remove(predicate)` | O(n)     | 删除符合条件的节点，级联删除子节点 |
| `clear()`           | O(n)     | 清空树               |
| `size()`            | O(1)     | 获取树中节点数           |

#### 节点类型选择指导

```java
public void treeNodeTypeSelection() {
    // 选择原则：选择功能最少的能满足需求的类型
    if (needMultipleParents) {
        // 支持多父节点DAG
        tree = Tree.of(new EnhanceNode<>());
    } else if (needDepthCalculation) {
        // 需要深度计算
        tree = Tree.of(new DeepNode<>(true));
    } else if (needFlattenJSON) {
        // 需要属性扁平化
        tree = Tree.of(new FlatNode<>(getters));
    } else {
        // 简单树结构，最好性能（推荐）
        tree = Tree.of(new DefaultNode<>());
    }
}
```

#### Tree 最佳实践

1. **批量添加**：一次 `add()` 操作比多次小规模 `add()` 性能好
2. **按ID查询**：优先使用 `getById()` 而不是 `find()`
3. **处理器轻量**：避免在处理器中做复杂业务逻辑、I/O操作
4. **定期清理**：删除过期节点，释放内存
5. **正确使用 find()**：tree.find() 返回的是 `List<Node>`，需要从 node 获取 element

```java
public void treeFindUsage() {
    // ❌ 错误：将 Node 直接 cast 为 Element 类型
    tree.find(n -> n.getElement() instanceof ClassDocElement)
        .stream()
        .map(ClassDocElement.class::cast)  // 错误！Node不是Element
        .toList();

    // ✅ 正确：从 node 获取 element 后再 cast
    tree.find(n -> Objects.nonNull(n.getElement()) && n.getElement() instanceof ClassDocElement)
        .stream()
        .map(n -> (ClassDocElement) n.getElement())  // 正确
        .toList();
}
```

#### 典型场景

**组织结构：**

```java
public void treeOrgStructure() {
    Tree<Integer, DeptElement, DefaultNode<Integer, DeptElement>> tree =
            Tree.of(new DefaultNode<>());
    tree.add(departments);
    // 按ID查询最快
    DefaultNode<Integer, DeptElement> dept = tree.getById(2);
}
```

**文件系统（支持符号链接）：**

```java
public void treeFileSystem() {
    Tree<String, FileElement, EnhanceNode<String, FileElement>> fileTree =
            Tree.of(new EnhanceNode<>());  // 支持多父节点
    fileTree.add(files);
    // 查询所有父目录
    EnhanceNode<String, FileElement> link = fileTree.getById("/link");
    link.findParents().forEach(parent -> {
        System.out.println("← " + parent.getElement().getPath());
    });
}
```

---

### 3.3 jpa-extension-starter JPA 扩展模块

> **模块位置**: `io.github.dao1230source:jpa-extension-starter`
> **完整文档**: 项目 README.md
> **GitHub**: https://github.com/Dao1230source/jpa-extension-starter

#### 核心功能

| 功能 | 描述 | 适用场景 |
|------|------|---------|
| **UnifiedJpaRepository** | 统一仓库接口，整合 JpaRepository、JpaSpecificationExecutor 和 ExtensionRepository | 所有 JPA 项目 |
| **JpaHelper** | 丰富的 CRUD 操作封装，支持自动设置用户ID、存在性检查、逻辑删除 | Service 层封装 |
| **批量操作** | 支持 MySQL `ON DUPLICATE KEY UPDATE` 批量插入/更新 | 高性能批量处理 |
| **Condition 构造器** | 链式查询条件构造器，支持动态查询 | 动态查询场景 |
| **权限控制** | 基于 Hibernate Filter 的 AOP 权限控制 | 数据权限管理 |
| **注解支持** | @LogicDelete、@CheckExists、@Filters、@Permission | 声明式配置 |

#### Maven 配置

```xml
<dependency>
    <groupId>io.github.dao1230source</groupId>
    <artifactId>jpa-extension-starter</artifactId>
    <version>0.0.12</version>
</dependency>
```

#### Repository 创建规范

```java
import org.source.jpa.repository.UnifiedJpaRepository;

/**
 * 用户数据访问接口
 * <p>
 * 继承 UnifiedJpaRepository 获得所有增强功能
 * </p>
 */
public interface UserRepository extends UnifiedJpaRepository<UserEntity, Long> {
    // 无需额外方法，已包含所有 CRUD 操作
}
```

#### Service 创建规范

```java
import org.source.jpa.enhance.AbstractJpaHelper;

/**
 * 用户业务服务
 * <p>
 * 继承 AbstractJpaHelper 获得 JPA 增强操作
 * </p>
 */
@Service
public class UserService extends AbstractJpaHelper<UserEntity, Long> {
    
    public UserService(UserRepository repository) {
        super(repository);
    }
    
    /**
     * 新增用户（自动检查重复）
     */
    public UserEntity addUser(UserEntity user) {
        return add(user);
    }
    
    /**
     * 批量保存用户（高性能）
     */
    public int batchSaveUsers(List<UserEntity> users) {
        return saveAll(users);  // ON DUPLICATE KEY UPDATE
    }
}
```

#### Entity 注解使用规范

```java
import org.source.jpa.enhance.annotation.LogicDelete;
import org.source.jpa.enhance.annotation.CheckExists;
import org.source.jpa.enhance.enums.OperateEnum;

@Entity
@Table(name = "user")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    /**
     * 用户名，唯一字段
     */
    @CheckExists(operate = {OperateEnum.ADD, OperateEnum.UPDATE})
    private String username;
    
    /**
     * 逻辑删除标识
     */
    @LogicDelete
    private Boolean deleted;
    
    /**
     * 创建用户ID（自动填充）
     */
    private Long createUser;
}
```

#### Condition 查询构造器

```java
import org.source.jpa.enhance.Condition;

/**
 * 动态查询示例
 */
public List<UserEntity> searchUsers(String username, Boolean deleted) {
    Condition<UserEntity> condition = new Condition<UserEntity>()
        // 等于条件
        .eq(UserEntity::getUsername, username)
        // 条件性等于（值不为空时才添加）
        .eqIfPresent(UserEntity::getDeleted, deleted)
        // IN 条件
        .in(UserEntity::getDeptId, deptIds);
    
    return findAll(condition);
}
```

#### JpaHelper 核心方法速查

| 方法分类 | 方法列表 | 时间复杂度 |
|---------|---------|-----------|
| **新增** | `add(T)`, `saveAll(Collection<T>)` | 批量操作 O(n) |
| **更新** | `update(T)`, `update(T, BinaryOperator<T>)` | 单次操作 O(1) |
| **删除** | `deleteById(I)`, `removeById(I)` | 单次操作 O(1) |
| **查询** | `getById(I)`, `findById(I)`, `findAll(Condition<T>)` | getById **O(1)** |

#### 批量操作性能对比

| 数据量 | `save()` 循环 | `saveAll()` | `onDuplicateUpdateBatch()` |
|--------|--------------|------------|---------------------------|
| 100 条 | ~1s | ~0.1s | **~0.01s** |
| 1000 条 | ~10s | ~1s | **~0.1s** |
| 10000 条 | ~100s | ~10s | **~1s** |

#### 最佳实践对比

**✅ 正确示例**：

```java
// 1. Repository 继承 UnifiedJpaRepository
public interface UserRepository extends UnifiedJpaRepository<UserEntity, Long> {
}

// 2. Service 继承 AbstractJpaHelper
@Service
public class UserService extends AbstractJpaHelper<UserEntity, Long> {
    public UserService(UserRepository repository) {
        super(repository);
    }
}

// 3. 批量操作使用 saveAll（高性能）
public int batchSave(List<UserEntity> users) {
    return saveAll(users);  // ON DUPLICATE KEY UPDATE
}

// 4. 动态查询使用 Condition 构造器
public List<UserEntity> search(String username) {
    Condition<UserEntity> condition = new Condition<UserEntity>()
        .eqIfPresent(UserEntity::getUsername, username);
    return findAll(condition);
}
```

**❌ 错误示例**：

```java
// 1. Repository 继承 JpaRepository（功能受限）
public interface UserRepository extends JpaRepository<UserEntity, Long> {
}

// 2. Service 不继承 AbstractJpaHelper（无增强功能）
@Service
public class UserService {
    @Autowired
    private UserRepository repository;
    
    // 手动封装，代码冗余
}

// 3. 循环调用 save（性能差）
public void batchSave(List<UserEntity> users) {
    for (UserEntity user : users) {
        repository.save(user);  // 每次一条 SQL，性能极差
    }
}

// 4. 字符串拼接查询（难以维护）
public List<UserEntity> search(String username) {
    String sql = "SELECT * FROM user WHERE username = '" + username + "'";
    // SQL 注入风险，难以维护
}
```

#### JPA Extension 注意事项

1. **Repository 必须继承 UnifiedJpaRepository**：获得所有增强功能
2. **Service 必须继承 AbstractJpaHelper**：自动获得 CRUD 操作
3. **批量操作优先**：使用 `saveAll()` 而非循环 `save()`
4. **按 ID 查询优先**：使用 `getById()` (O(1)) 而非 `findAll()` + 过滤
5. **动态查询使用 Condition**：避免字符串拼接 SQL
6. **逻辑删除使用 @LogicDelete**：无需手动管理删除状态
7. **存在性检查使用 @CheckExists**：自动防止重复数据

---

## 第四部分：项目结构参考

```
org.source.xxx/
├── app/              # 多领域聚合逻辑
    └── XxxApp        # 聚合业务逻辑
├── controller/       # REST控制器
    └── XxxController # REST 控制器
├── domain/           # 领域模型
    ├── entity/       # 数据库表实体类
        └── XxxEnitity # 对应数据库表的实体类
    ├── repository/   # 数据库表单表操作
        └── XxxRepository # 继承`UnifiedJpaRepository`，对应库表的Jpa封装扩展
    └── service/      # 数据库表领域操作，一个领域可能涉及多表
        └── XxxService # 继承`AbstractJpaHelper`，对应数据库操作扩展，以及最小领域的操作
├── facade/           # 数据门户，外部数据和数据库数据交换的转换层
    ├── data/         # 流转数据，不涉及于外部的交互，即不是外部输入也不是输出到外部
        └── XxxData  # 流转数据 
    ├── mapper/       # 数据转换，输入数据-数据库数据-输出数据 的转换层
        └── XxxMapper # 两个对象的转换继承`TwoMapper`，三个对象的转换继承`ThreeMapper`
    ├── input/        # 输入数据
        └── XxxIn  # web、接口、消息队列等输入
    ├── output/       # 输出数据
        └── XxxOut    # 输出给前端、三方系统、推送消息等
    └── XxxFacade     # 聚合门户调用app层，一般只负责数据转换
├── infrastructure/   # 基础层
    ├── constant/     # 常量类
    ├── exception/ # 异常
        └── XxxException # 业务异常，继承BaseException
        └── XxxExceptionEnum # 异常的枚举类，实现EnumProcessor<XxxException>
    ├── enums/        # 业务枚举类
    ├── config/       # 配置类
    ├── util/         # 工具类
└── Application.java  # 启动类
```

---

#### 项目结构中标准类的实现的示例说明

```java
import org.source.jpa.repository.UnifiedJpaRepository;

// repository
public interface UserRepository extends UnifiedJpaRepository<UserEntity, Long> {
}
```

```java
import org.source.jpa.enhance.AbstractJpaHelper;

// service
@Component
public class UserService extends AbstractJpaHelper<UserEntity, Long> {

    public UserService(UserRepository repository) {
        super(repository);
    }
}
```

```java
import org.source.utility.mapstruct.ThreeMapper;

// 三个对象转换 in <-> entity <-> out 
@Mapper
public interface UserMapper extends ThreeMapper<UserIn, UserEntity, UserOut> {

    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);

}
```

```java
import org.source.utility.mapstruct.TwoMapper;

// 两个对象转换
@Mapper
public interface UserTenantMapper extends TwoMapper<UserTenantEntity, UserTenantData> {

    UserTenantMapper INSTANCE = Mappers.getMapper(UserTenantMapper.class);

}
```

```java
import org.source.utility.exception.BaseException;
import org.source.utility.exception.EnumProcessor;

// 业务异常
@EqualsAndHashCode(callSuper = true)
@Getter
public class JpaExtException extends BaseException {

    public JpaExtException(@NotNull EnumProcessor<?> content, @Nullable Throwable cause, @Nullable String extraMessage, @Nullable Object... objects) {
        super(content, cause, extraMessage, objects);
    }

    public JpaExtException(EnumProcessor<?> content, String extraMessage, Object... objects) {
        super(content, extraMessage, objects);
    }

    public JpaExtException(EnumProcessor<?> content, Throwable e) {
        super(content, e);
    }

    public JpaExtException(EnumProcessor<?> content) {
        super(content);
    }

}
```

```java
import org.source.utility.exception.EnumProcessor;

// 业务异常枚举
@Getter
@AllArgsConstructor
public enum JpaExtExceptionEnum implements EnumProcessor<JpaExtException> {

    NOT_NULL("not null");

    private final String message;

    @Override
    public String getCode() {
        return this.name();
    }

}
```

---

## 第五部分：依赖配置

### 推荐依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>io.github.dao1230source</groupId>
        <artifactId>parent</artifactId>
        <version>0.0.12</version>
        <relativePath/>
    </parent>
    <artifactId>demo</artifactId>
    <dependencies>
        <!-- Tool Library -->
        <dependency>
            <groupId>io.github.dao1230source</groupId>
            <artifactId>utility</artifactId>
            <version>0.0.12</version>
        </dependency>

        <!-- Null Safety -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-collections4</artifactId>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

## Reference

- [SonarQube Rules](https://rules.sonarsource.com/java/)
- [Alibaba Java Coding Guidelines](https://github.com/alibaba/p3c)
- [Apache Commons Lang3](https://commons.apache.org/proper/commons-lang/)
- [Apache Commons Collections4](https://commons.apache.org/proper/commons-collections/)
- [Project Lombok](https://projectlombok.org/)
- [SLF4J Documentation](https://www.slf4j.org/)
- **Assign 模块**: [reference/Assign.md](reference/Assign.md)
- **Tree 模块**: [reference/Tree.md](reference/Tree.md)
- **JPA Extension 模块**: [jpa-extension-starter GitHub](https://github.com/Dao1230source/jpa-extension-starter)