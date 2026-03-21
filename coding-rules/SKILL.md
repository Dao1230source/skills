---
name: coding-rules
description: 全局代码编写规范，包含Sonar、Alibaba Java Guide规则，commons-lang3/commons-collections4判空处理，以及项目模块使用规范（Assign批量赋值、Tree树结构）
license: MIT
compatibility: opencode
metadata:
  author: zengfugen
  version: 2.0.0
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
- **使用 Assign 模块进行批量赋值时**
- **使用 Tree 模块处理树形结构时**

**触发关键词**：

- "代码规范"、"编码规范"、"编码"、"编码规则"
- "Sonar"、"Alibaba Java Guide"
- "判空"、"commons-lang3"、"commons-collections4"
- "项目结构"、"代码架构"
- **"批量赋值"、"Assign"、"Acquire"**
- **"Tree"、"树形结构"、"树"、"DAG"、"深度计算"、"扁平化"**

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

### 1.4 日志规范

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
// 集合判空
if(CollectionUtils.isNotEmpty(list)){
        // 处理非空集合
        }
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

### 1.7 其他强制要求

- **注释**: 所有公共方法必须有 JavaDoc 注释，不要使用尾行注释
- **依赖注入**: 优先使用构造函数注入
- **不可变性**: 优先使用不可变对象（final 字段、record 类型）
- **线程安全**: 注意并发安全问题

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