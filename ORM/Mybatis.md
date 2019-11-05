![image-20191104210254063](image/image-20191104210254063.png)

 https://mybatis.org/mybatis-3/zh/index.html 

 http://mybatis.org/spring/zh/index.html 

 http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/ 

[TOC]



# 简介

是一款优秀的持久层框架，支持定制化SQL、存储过程和高级映射。MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。MB可以使用简单的XML或者注解来配置和映射原生类型、接口和Java的POJO为数据库中的记录。



# 入门

## 安装

只需要将jar包置于classpath

## SqlSessionFactory

每个基于MyBatis的应用都是以一个`SqlSessionFactory`的实例为核心的。`SqlSessionFactory`的实例可以通过`SqlSessionFactoryBuilder`获得，而`SqlSessionFactoryBuilder`可以从XML配置文件或者一个预先定制的Configuration的实例构建出。

然后可以通过session获取Mapper实例，再进行CURD。

## 作用域和生命周期

错误的使用会导致严重的并发问题。

`SqlSessionFactoryBuilder`可以被实例化、使用和丢弃。一旦创建了`SqlSessionFactory`就不需要它了，因此实例的最佳作用域是方法作用域（局部方法变量）。

`SqlSessionFactory`一旦被创建就应该再应用的允许期间一直存在，没有任何理由丢弃它或者重新创建另一个实例。最佳时间是应用作用域，最简单的就是单例模式或者静态单例模式。

`SqlSession`每个线程都应该有自己的实例。SqlSession的实例不是线程安全的，因此是不能被共享的，所以它的最佳作用域是请求或方法作用域，绝对不能将`SqlSeesion`实例的引用放在一个类的静态作用域，甚至一个类的实例变量也不想。也绝不能将`SqlSession`实例的引用放在任何类型的托管作用域中，比如Servlet框架中的`HttpSession`。如果你现在正在使用一种Web框架，要考虑把`SqlSession`放在一个和HTTP请求对象相似的作用域中。换句话说，每次收到的HTTP请求，就可以打开一个`SqlSession`,返回一个响应就关闭它。

映射器实例是从`SqlSession`中获得的，因此最大作用域是方法作用域。

**Note**

依赖注入框架可以创建线程安全的、基于事务的SqlSession和映射器，并将它们直接注入到你的bean中，因此可以直接忽略它们的生命周期。







# 配置

## 属性（properties）

这些属性是可外部配置并且可动态替换的，既可以在典型的Java属性文件中配置，亦可以通过`properties`元素的子元素来传递：

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

然后其中的属性就可以在整个配置文件中被用来替换需要动态配置的属性值

```XML
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource
```

这个例子中的`ysername`和`password`将会由`properties`元素中设置的相应值来替换，`driver`和`url`中的属性会由config.properties文件中相应的值来替换。

通过方法参数传递的属性具有最高优先级，resource/url属性中指定的配置文件次之，最低优先级的是properties属性中指定的属性。

## 设置（settings）

是MyBatis中极为重要的调整设置，它们会改变MB的运行时行为。下表描述了设置中各项的意图和默认值

| 设置名                           | 描述                                                         | 有效值                                                       | 默认值                                                |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------- |
| cacheEnabled                     | 全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。   | true \| false                                                | true                                                  |
| lazyLoadingEnabled               | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false                                                 |
| aggressiveLazyLoading            | 当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false                                                | false （在 3.4.1 及之前的版本默认值为 true）          |
| multipleResultSetsEnabled        | 是否允许单一语句返回多结果集（需要驱动支持）。               | true \| false                                                | true                                                  |
| useColumnLabel                   | 使用列标签代替列名。不同的驱动在这方面会有不同的表现，具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。 | true \| false                                                | true                                                  |
| useGeneratedKeys                 | 允许 JDBC 支持自动生成主键，需要驱动支持。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能支持但仍可正常工作（比如 Derby）。 | true \| false                                                | False                                                 |
| autoMappingBehavior              | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                               |
| autoMappingUnknownColumnBehavior | 指定发现自动映射目标未知列（或者未知属性类型）的行为。`NONE`: 不做任何反应`WARNING`: 输出提醒日志 (`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`)`FAILING`: 映射失败 (抛出 `SqlSessionException`) | NONE, WARNING, FAILING                                       | NONE                                                  |
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                |
| defaultStatementTimeout          | 设置超时时间，它决定驱动等待数据库响应的秒数。               | 任意正整数                                                   | 未设置 (null)                                         |
| defaultFetchSize                 | 为驱动的结果集获取数量（fetchSize）设置一个提示值。此参数只可以在查询设置中被覆盖。 | 任意正整数                                                   | 未设置 (null)                                         |
| defaultResultSetType             | Specifies a scroll strategy when omit it per statement settings. (Since: 3.5.2) | FORWARD_ONLY \| SCROLL_SENSITIVE \| SCROLL_INSENSITIVE \| DEFAULT(same behavior with 'Not Set') | Not Set (null)                                        |
| safeRowBoundsEnabled             | 允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true \| false                                                | False                                                 |
| safeResultHandlerEnabled         | 允许在嵌套语句中使用分页（ResultHandler）。如果允许使用则设置为 false。 | true \| false                                                | True                                                  |
| mapUnderscoreToCamelCase         | 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。 | true \| false                                                | False                                                 |
| localCacheScope                  | MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。 | SESSION \| STATEMENT                                         | SESSION                                               |
| jdbcTypeForNull                  | 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType 常量，常用值：NULL, VARCHAR 或 OTHER。              | OTHER                                                 |
| lazyLoadTriggerMethods           | 指定哪个对象的方法触发一次延迟加载。                         | 用逗号分隔的方法列表。                                       | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage         | 指定动态 SQL 生成的默认语言。                                | 一个类型别名或完全限定类名。                                 | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler           | 指定 Enum 使用的默认 `TypeHandler` 。（新增于 3.4.5）        | 一个类型别名或完全限定类名。                                 | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls               | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值初始化的时候比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 | true \| false                                                | false                                                 |
| returnInstanceForEmptyRow        | 当返回行的所有列都是空时，MyBatis默认返回 `null`。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集 （如集合或关联）。（新增于 3.4.2） | true \| false                                                | false                                                 |
| logPrefix                        | 指定 MyBatis 增加到日志名称的前缀。                          | 任何字符串                                                   | 未设置                                                |
| logImpl                          | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置                                                |
| proxyFactory                     | 指定 Mybatis 创建具有延迟加载能力的对象所用到的代理工具。    | CGLIB \| JAVASSIST                                           | JAVASSIST （MyBatis 3.3 以上）                        |
| vfsImpl                          | 指定 VFS 的实现                                              | 自定义 VFS 的实现的类全限定名，以逗号分隔。                  | 未设置                                                |
| useActualParamName               | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 `-parameters` 选项。（新增于 3.4.1） | true \| false                                                | true                                                  |
| configurationFactory             | 指定一个提供 `Configuration` 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为`static Configuration getConfiguration()` 的方法。（新增于 3.2.3） |                                                              |                                                       |

一个配置的完整示例如下：

```XML
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

## 类型别名（typeAliases）

是为Java类型设置的一个短的名字，它只和XML配置有关，存在的意义仅在于用来减少类的完全限定名的冗余

有两种方式

有两种使用方式，可直接在XML中配置

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

也可以指定一个包名然后使用注解

```XML
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

```JAVA
@Alias("author")
public class Author {
    ...
}
```

此外，

还未一些常见的java类型内建了相应的类型别名。它们都是不区分大小写的

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

## 类型处理器（typeHandlers）

无论是MB在预处理语句中设置一个参数时，还是从结果集中取出一个值时，都会用类型处理器将获取的值以合适的方式转换成Java类型。

可以重写类型处理器或创建自己的类型处理器来处理不支持或非标准型的类型。具体做法为：实现`org.apache.ibatis.type.TypeHandler`接口或者继承`org.apache.ibatis.type.BaseTyeHandler`，然后选择性的将它映射到一个JDBC类型。

```java
// ExampleTypeHandler.java
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ExampleTypeHandler extends BaseTypeHandler<String> {

  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getString(columnName);
  }

  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return rs.getString(columnIndex);
  }

  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return cs.getString(columnIndex);
  }
}
```

```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

使用上述的类型处理器将会覆盖已经存在的处理Java的String类型属性和VARCHAR参数及结果的类型处理器。

要注意MyBatis不会通过窥探数据库元信息来决定使用哪种类型，所以你必须在参数和结果映射中指明是什么类型的字段以使其能够绑定到正确的类型处理器上。

通过类型处理器的泛型，MB可以得知该类型处理器处理的Java类型，不过这种行为可以通过两种方式改变。

* 在类型处理的配置元素上增加一个`javaType`属性
* 在类型处理器的类上增加一个`MapperTypes`注解来指定与其相关联的JDBC类型列表。如果1被设置，2会忽略

？？？当在`ResultMap`中决定使用哪种类型处理器时，此时的Java类型是已知的（从结果类型中获取），但是JDBC类型是未知的。因此MB中使用`javaType={Java类型}，jdbcType=null`的组合来选择一个类型处理器。

## 处理枚举类型

若想映射枚举类型`Enum`,则需要从`EnumTyoeHandler` 或者`EnumOrdinalTypeHandler`中选一个来使用。

默认情况下MB会利用`EnumTypeHandler`来把Enum值转换成对应的名字。

相反的，如果想用整型值存储在数据库中，在配置文件中把`EnumOrdinalTypeHandler`加到`typeHandlers`中即可。这样`RoundingMode将通过他们的序数值来映射成对应的整形数值`

```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="java.math.RoundingMode"/>
</typeHandlers>
```

如何将`Enum`又映射为字符串又映射为整形呢？

```xml
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.apache.ibatis.submitted.rounding.Mapper">
	<resultMap type="org.apache.ibatis.submitted.rounding.User" id="usermap">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<result column="funkyNumber" property="funkyNumber"/>
		<result column="roundingMode" property="roundingMode"/>
	</resultMap>

	<select id="getUser" resultMap="usermap">
		select * from users
	</select>
	<insert id="insert">
	    insert into users (id, name, funkyNumber, roundingMode) values (
	    	#{id}, #{name}, #{funkyNumber}, #{roundingMode}
	    )
	</insert>

	<resultMap type="org.apache.ibatis.submitted.rounding.User" id="usermap2">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<result column="funkyNumber" property="funkyNumber"/>
		<result column="roundingMode" property="roundingMode" typeHandler="org.apache.ibatis.type.EnumTypeHandler"/>
	</resultMap>
	<select id="getUser2" resultMap="usermap2">
		select * from users2
	</select>
	<insert id="insert2">
	    insert into users2 (id, name, funkyNumber, roundingMode) values (
	    	#{id}, #{name}, #{funkyNumber}, #{roundingMode, typeHandler=org.apache.ibatis.type.EnumTypeHandler}
	    )
	</insert>

</mapper>
```

## 对象工厂(objectFactory)

MyBatis每次创建结果对象的新实例时，它都会使用一个对象工厂实例来完成。默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数的构造方法来实例化。如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现。

## 插件（plugins）

 MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括： 

- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
- ParameterHandler (getParameterObject, setParameters)
- ResultSetHandler (handleResultSets, handleOutputParameters)
- StatementHandler (prepare, parameterize, batch, update, query)

 通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。 

## 环境配置（environments）

环境元素定义了如何配置环境

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

- 默认使用的环境 ID（比如：default="development"）。
- 每个 environment 元素定义的环境 ID（比如：id="development"）。
- 事务管理器的配置（比如：type="JDBC"）。
- 数据源的配置（比如：type="POOLED"）。

### 事务管理器（transactionMangager）

MB中有两种类型的事务管理器(JDBC|MANAGED)

* JDBC-直接使用了JDBC的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域
* MANAGED-这个配置机会没做什么，而是让容器管理事务的整个生命周期。

**Note**

如果使用Spring则没有必要配置事务管理器，因为Spirng会使用自带的管理器来覆盖配置

### 数据源（dataSource）

## 数据库厂商标识（databaseldProvider）

```xml
<databaseIdProvider type="DB_VENDOR" />
```

## 映射器（mappers）

 告诉 MyBatis 到哪里去找映射文件。 你可以使用相对于类路径的资源引用， 或完全限定资源定位符（包括 `file:///` 的 URL），或类名和包名等。 

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

```xml
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
```

```xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```





# XML映射文件

MB的真正强大之处，为聚焦于SQL而构建，以尽可能地为你减少麻烦。

SQL元素只有很少的几个顶级元素：

- `cache` – 对给定命名空间的缓存配置。
- `cache-ref` – 对其他命名空间缓存配置的引用。
- `resultMap` – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
- `sql` – 可被其他语句引用的可重用语句块。
- `insert` – 映射插入语句
- `update` – 映射更新语句
- `delete` – 映射删除语句
- `select` – 映射查询语句

## select

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

 这个语句被称作 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值。 

Select元素允许你配置很多属性来配置每条语句地作用细节

```xml
<select
  id="selectPerson"
  parameterType="int"
  parameterMap="deprecated"
  resultType="hashmap"
  resultMap="personResultMap"
  flushCache="false"
  useCache="true"
  timeout="10"
  fetchSize="256"
  statementType="PREPARED"
  resultSetType="FORWARD_ONLY">
```

| 属性            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| `id`            | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| `parameterType` | 将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler） 推断出具体传入语句的参数，默认值为未设置（unset）。 |
| parameterMap    | 这是引用外部 parameterMap 的已经被废弃的方法。请使用内联参数映射和 parameterType 属性。 |
| `resultType`    | 从这条语句中返回的期望类型的类的完全限定名或别名。 注意如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身。可以使用 resultType 或 resultMap，但不能同时使用。 |
| `resultMap`     | 外部 resultMap 的命名引用。结果集的映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂映射的情形都能迎刃而解。可以使用 resultMap 或 resultType，但不能同时使用。 |
| `flushCache`    | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。 |
| `useCache`      | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。 |
| `timeout`       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖驱动）。 |
| `fetchSize`     | 这是一个给驱动的提示，尝试让驱动程序每次批量返回的结果行数和这个设置值相等。 默认值为未设置（unset）（依赖驱动）。 |
| `statementType` | STATEMENT，PREPARED 或 CALLABLE 中的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `resultSetType` | FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖驱动）。 |
| `databaseId`    | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |
| `resultOrdered` | 这个设置仅针对嵌套结果 select 语句适用：如果为 true，就是假设包含了嵌套结果集或是分组，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。 这就使得在获取嵌套的结果集的时候不至于导致内存不够用。默认值：`false`。 |
| `resultSets`    | 这个设置仅对多结果集的情况适用。它将列出语句执行后返回的结果集并给每个结果集一个名称，名称是逗号分隔的。 |

## insert,update和delete

这三个的实现非常接近，有如下属性

```xml
<insert
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="updateAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="deleteAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">
```

| 属性               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| `id`               | 命名空间中的唯一标识符，可被用来代表这条语句。               |
| `parameterType`    | 将要传入语句的参数的完全限定类名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器推断出具体传入语句的参数，默认值为未设置（unset）。 |
| `parameterMap`     | 这是引用外部 parameterMap 的已经被废弃的方法。请使用内联参数映射和 parameterType 属性。 |
| `flushCache`       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：true（对于 insert、update 和 delete 语句）。 |
| `timeout`          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖驱动）。 |
| `statementType`    | STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `useGeneratedKeys` | （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。 |
| `keyProperty`      | （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认值：未设置（`unset`）。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| `keyColumn`        | （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望使用多个生成的列，也可以设置为逗号分隔的属性名称列表。 |
| `databaseId`       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |

实例：

```xml
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>

<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>
```

如前所述，插入语句的配置规则更加丰富，在插入语句里面有一些额外的属性和子元素用来处理主键的生成，而且有多种生成方式。

首先，如果你的数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置到目标属性上就 OK 了。例如，如果上面的 Author 表已经对 id 使用了自动生成的列类型，那么语句可以修改为：

```
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```

如果你的数据库还支持多行插入, 你也可以传入一个 `Author` 数组或集合，并返回自动生成的主键。

```xml
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>
```

## sql

这个元素可以被用来定义可重用的SQL代码段，这些SQL代码可以被包含在 其他语句中。它可以在加载的时候被静态的设置参数。在不同的包含语句中可以设置不同的值到参数占位符上。

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```

这个 SQL 片段可以被包含在其他语句中，例如：

```xml
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

## 参数

```xml
<select id="selectUsers" resultType="User">
  select id, username, password
  from users
  where id = #{id}
</select>
```

`#{id}`可以被设置任意内容，原始类型或者简单数据类型会完成用参数值来替代。

然而，如果是传入一个复杂对象那么行为就会有一点不同

```xml
<insert id="insertUser" parameterType="User">
  insert into users (id, username, password)
  values (#{id}, #{username}, #{password})
</insert>
```

如果User类型的参数被传递到了语句中，属性将会被查找，然后将它们的值传入到预处理语句的参数中。

参数映射的功能远不止此

首先，像 MyBatis 的其他部分一样，参数也可以指定一个特殊的数据类型。

```xml
#{property,javaType=int,jdbcType=NUMERIC}
```

像 MyBatis 的其它部分一样，javaType 几乎总是可以根据参数对象的类型确定下来，除非该对象是一个 `HashMap`。这个时候，你需要显式指定 `javaType` 来确保正确的类型处理器（`TypeHandler`）被使用。

**Note**

**JDBC要求，如果一个列允许null并且会传递null的参数，就必须要指定JDBC Type**

最终可能是这种形式，但是实际中很少需要如此繁琐的配置

```xml
#{age,javaType=int,jdbcType=NUMERIC,typeHandler=MyTypeHandler}
```

对于数值类型，还有一个小鼠保留位数的设置，来指定小数点后保留的位数。

```
#{height,javaType=double,jdbcType=NUMERIC,numericScale=2}
```

字符串替换：

可以使用`${name}`进行替换 ，例如

```xml
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```

就可以实现替换表明或者查询条件，但是这样是不安全的，会导致潜在的SQL注入攻击

## 结果映射

`resultMap`元素是MB中最重要最强大的元素，可以让你从90%的JDBC`ResultSets`数据提取代码中解放出来，并在一些情形下允许你进行一些JDBC不支持的操作。

可以直接将类的属性对应到select语句中的列名。

```XML
<select id="selectUsers" resultType="com.someapp.model.User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

类型别名是你的好帮手。使用它们，你就可以不用输入类的完全限定名称了。比如：

```xml
<!-- mybatis-config.xml 中 -->
<typeAlias type="com.someapp.model.User" alias="User"/>

<!-- SQL 映射 XML 中 -->
<select id="selectUsers" resultType="User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

这些情况下,MB会在幕后自动创建一个`ResultMap`，在基于属性名映射列到JavaBean属性上。如果列名和属性名没有精确匹配，可以在SELECT语句中对列使用别名来匹配标签。

```xml
<select id="selectUsers" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```

也可以使用外部的`resultMap`来解决列名不匹配的问题

```xml
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>
```

而在引用中使用`resultMap`属性

```xml
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```

### 高级结果映射

MB创建时的一个思想是：数据库不可能永远是你所想或所需的那个样子。我们希望每个数据库都具备良好的第三范式或者BCNF范式，可惜它们不总都是这样。`ResultMap`就是MB对这个问题的答案。

```xml
<!-- 非常复杂的语句 -->
<select id="selectBlogDetails" resultMap="detailedBlogResultMap">
  select
       B.id as blog_id,
       B.title as blog_title,
       B.author_id as blog_author_id,
       A.id as author_id,
       A.username as author_username,
       A.password as author_password,
       A.email as author_email,
       A.bio as author_bio,
       A.favourite_section as author_favourite_section,
       P.id as post_id,
       P.blog_id as post_blog_id,
       P.author_id as post_author_id,
       P.created_on as post_created_on,
       P.section as post_section,
       P.subject as post_subject,
       P.draft as draft,
       P.body as post_body,
       C.id as comment_id,
       C.post_id as comment_post_id,
       C.name as comment_name,
       C.comment as comment_text,
       T.id as tag_id,
       T.name as tag_name
  from Blog B
       left outer join Author A on B.author_id = A.id
       left outer join Post P on B.id = P.blog_id
       left outer join Comment C on P.id = C.post_id
       left outer join Post_Tag PT on PT.post_id = P.id
       left outer join Tag T on PT.tag_id = T.id
  where B.id = #{id}
</select>
```

你可能想把它映射到一个智能的对象模型，这个对象表示了一篇博客，它由某位作者所写，有很多的博文，每篇博文有零或多条的评论和标签。 我们来看看下面这个完整的例子，它是一个非常复杂的结果映射（假设作者，博客，博文，评论和标签都是类型别名）。 不用紧张，我们会一步一步来说明。虽然它看起来令人望而生畏，但其实非常简单。

```xml
<!-- 非常复杂的结果映射 -->
<resultMap id="detailedBlogResultMap" type="Blog">
  <constructor>
    <idArg column="blog_id" javaType="int"/>
  </constructor>
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
    <result property="favouriteSection" column="author_favourite_section"/>
  </association>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <association property="author" javaType="Author"/>
    <collection property="comments" ofType="Comment">
      <id property="id" column="comment_id"/>
    </collection>
    <collection property="tags" ofType="Tag" >
      <id property="id" column="tag_id"/>
    </collection>
    <discriminator javaType="int" column="draft">
      <case value="1" resultType="DraftPost"/>
    </discriminator>
  </collection>
</resultMap>
```

`resultMap`元素有很多子元素和一个值得深入探讨的结构，`resultMap`元素的概念视图。

#### 结果映射（resultMap）

* `constructor`-用于在实例化类中，注入结果到构造方法中
  * `idArg`:-ID参数，标记处作为ID的结果可以帮助提高整体性能
  * `arg`-将被注入到构造方法的一个普通结果
* `id`-一个ID结果；标记处作为ID的结果可以帮助提高整体性能
* `result`-注入到字段或JavaBean属性的普通结果
* `association`-一个复杂类型的关联，许多结果将包装成这个类型
  * 嵌套结果映射-关联本身可以是一个`resultMap`元素

* `collection`-一个复杂类型的集合
  * 嵌套结果映射-集合本身可以是一个`resultMap`元素
* `discreminator`-使用结果值来决定使用哪个`resultMap`
  * `case`-基于某些值的结果映射
    * 嵌套结果映射-`case`本身是一个`resultMap`元素



##### id&reult

```xml
<id property="id" column="post_id"/>
<result property="subject" column="post_subject"/>
```

这是结果映射最基本的内容，`id`和`result`都将一个列的值映射到一个简单数据类型。

| 属性          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| `property`    | 映射到列结果的字段或属性。如果用来匹配的 JavaBean 存在给定名字的属性，那么它将会被使用。否则 MyBatis 将会寻找给定名称的字段。 无论是哪一种情形，你都可以使用通常的点式分隔形式进行复杂属性导航。 比如，你可以这样映射一些简单的东西：“username”，或者映射到一些复杂的东西上：“address.street.number”。 |
| `column`      | 数据库中的列名，或者是列的别名。一般情况下，这和传递给 `resultSet.getString(columnName)` 方法的参数一样。 |
| `javaType`    | 一个 Java 类的完全限定名，或一个类型别名（关于内置的类型别名，可以参考上面的表格）。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。 |
| `jdbcType`    | JDBC 类型，所支持的 JDBC 类型参见这个表格之后的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可能存在空值的列指定这个类型。 |
| `typeHandler` | 我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的完全限定名，或者是类型别名。 |

##### 构造方法

MB支持私有属性和JavaBean属性进行注入，但是一些人更青睐于通过构造方法进行注入。

```java
public class User {
   //...
   public User(Integer id, String username, int age) {
     //...
  }
//...
}
```

可以通过`constructor`进行注入

```xml
<constructor>
   <idArg column="id" javaType="int"/>
   <arg column="username" javaType="String"/>
   <arg column="age" javaType="_int"/>
</constructor>
```

可以指定参数名称

```xml
<constructor>
   <idArg column="id" javaType="int" name="id" />
   <arg column="age" javaType="_int" name="age" />
   <arg column="username" javaType="String" name="username" />
</constructor>
```

##### 关联

```xml
<association property="author" column="blog_author_id" javaType="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
</association>
```

关联处理“有一个”类型的关系。比如，在我们的示例中，一个博客又一个用户，关联结果映射和其它类型的映射工作方式差不多。需要指定目标属性名以及属性的`javaType`

不同之处在于，要告诉MB如何加载关联。这里有两种方式：嵌套Select查询和嵌套结果映射

* 嵌套Select查询

  ```xml
  <resultMap id="blogResult" type="Blog">
    <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
  </resultMap>
  
  <select id="selectBlog" resultMap="blogResult">
    SELECT * FROM BLOG WHERE ID = #{id}
  </select>
  
  <select id="selectAuthor" resultType="Author">
    SELECT * FROM AUTHOR WHERE ID = #{id}
  </select>
  ```

  用两个查询语句，一个用来加载Blog，一个用来加载Author，而且博客的结果映射描述了应该使用`selectAuthor`语句加载它的author属性。其它所有的属性都会被自动加载，只要它们的列名和属性名相匹配.

  这种方式虽然很简单，但在大型数据集或大型数据表上表现不佳。这个问题被称为“N+1 查询问题”。 概括地讲，N+1 查询问题是这样子的：

  - 你执行了一个单独的 SQL 语句来获取结果的一个列表（就是“+1”）。
  - 对列表返回的每条记录，你执行一个 select 查询语句来为每条记录加载详细信息（就是“N”）。

  这个问题会导致成百上千的 SQL 语句被执行。有时候，我们不希望产生这样的后果。

   好消息是，MyBatis 能够对这样的查询进行延迟加载，因此可以将大量语句同时运行的开销分散开来。 然而，如果你加载记录列表之后立刻就遍历列表以获取嵌套的数据，就会触发所有的延迟加载查询，性能可能会变得很糟糕。 

* 关联的嵌套结果查询

| 属性            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| `resultMap`     | 结果映射的 ID，可以将此关联的嵌套结果集映射到一个合适的对象树中。 它可以作为使用额外 select 语句的替代方案。它可以将多表连接操作的结果映射成一个单一的 `ResultSet`。这样的 `ResultSet` 有部分数据是重复的。 为了将结果集正确地映射到嵌套的对象树中, MyBatis 允许你“串联”结果映射，以便解决嵌套结果集的问题。使用嵌套结果映射的一个例子在表格以后。 |
| `columnPrefix`  | 当连接多个表时，你可能会不得不使用列别名来避免在 `ResultSet` 中产生重复的列名。指定 columnPrefix 列名前缀允许你将带有这些前缀的列映射到一个外部的结果映射中。 详细说明请参考后面的例子。 |
| `notNullColumn` | 默认情况下，在至少一个被映射到属性的列不为空时，子对象才会被创建。 你可以在这个属性上指定非空的列来改变默认行为，指定后，Mybatis 将只在这些列非空时才创建一个子对象。可以使用逗号分隔来指定多个列。默认值：未设置（unset）。 |
| `autoMapping`   | 如果设置这个属性，MyBatis 将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性 autoMappingBehavior。注意，本属性对外部的结果映射无效，所以不能搭配 `select` 或 `resultMap` 元素使用。默认值：未设置（unset）。 |

下边的例子演示了嵌套结果映射如何工作，

```xml
<select id="selectBlog" resultMap="blogResult">
  select
    B.id            as blog_id,
    B.title         as blog_title,
    B.author_id     as blog_author_id,
    A.id            as author_id,
    A.username      as author_username,
    A.password      as author_password,
    A.email         as author_email,
    A.bio           as author_bio
  from Blog B left outer join Author A on B.author_id = A.id
  where B.id = #{id}
</select>
```

可以映射这个结果

```xml
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" column="blog_author_id" javaType="Author" resultMap="authorResult"/>
</resultMap>

<resultMap id="authorResult" type="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
  <result property="email" column="author_email"/>
  <result property="bio" column="author_bio"/>
</resultMap>
```

所以，博客和作者的关联元素委托名为“authorResult”的结果映射来加载作者对象的实例。

**Note** id 元素在嵌套结果映射中扮演着非常重要的角色。你应该总是指定一个或多个可以唯一标识结果的属性。 虽然，即使不指定这个属性，MyBatis 仍然可以工作，但是会产生严重的性能问题。 只需要指定可以唯一标识结果的最少属性。显然，你可以选择主键（复合主键也可以）。

在这种写法下，Author的结果映射可以被重用，然而，如果你不打算重用它，或者你更喜欢将你所以的结果映射放在一个更具有描述性的结果映射元素中。你可以直接将结果映射作为子元素嵌套在内

```xml
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
  </association>
</resultMap>
```

**别名**

那如果博客（blog）有一个共同作者（co-author）该怎么办？select 语句看起来会是这样的：

```xml
<select id="selectBlog" resultMap="blogResult">
  select
    B.id            as blog_id,
    B.title         as blog_title,
    A.id            as author_id,
    A.username      as author_username,
    A.password      as author_password,
    A.email         as author_email,
    A.bio           as author_bio,
    CA.id           as co_author_id,
    CA.username     as co_author_username,
    CA.password     as co_author_password,
    CA.email        as co_author_email,
    CA.bio          as co_author_bio
  from Blog B
  left outer join Author A on B.author_id = A.id
  left outer join Author CA on B.co_author_id = CA.id
  where B.id = #{id}
</select>
```

回忆一下，Author 的结果映射定义如下：

```xml
<resultMap id="authorResult" type="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
  <result property="email" column="author_email"/>
  <result property="bio" column="author_bio"/>
</resultMap>
```

由于结果中的列名与结果映射中的列名不同。你需要指定 `columnPrefix` 以便重复使用该结果映射来映射 co-author 的结果。

```xml
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author"
    resultMap="authorResult" />
  <association property="coAuthor"
    resultMap="authorResult"
    columnPrefix="co_" />
</resultMap>
```

* 关联的多结果集（ResultSet）

  | 属性            | 描述                                                         |
  | :-------------- | :----------------------------------------------------------- |
  | `column`        | 当使用多个结果集时，该属性指定结果集中用于与 `foreignColumn` 匹配的列（多个列名以逗号隔开），以识别关系中的父类型与子类型。 |
  | `foreignColumn` | 指定外键对应的列名，指定的列将与父类型中 `column` 的给出的列进行匹配。 |
  | `resultSet`     | 指定用于加载复杂类型的结果集名字。                           |

  从版本 3.2.3 开始，MyBatis 提供了另一种解决 N+1 查询问题的方法。

  某些数据库允许存储过程返回多个结果集，或一次性执行多个语句，每个语句返回一个结果集。 我们可以利用这个特性，在不使用连接的情况下，只访问数据库一次就能获得相关数据。

  在例子中，存储过程执行下面的查询并返回两个结果集。第一个结果集会返回博客（Blog）的结果，第二个则返回作者（Author）的结果。

  ```xml
  SELECT * FROM BLOG WHERE ID = #{id}
  
  SELECT * FROM AUTHOR WHERE ID = #{id}
  ```

  在映射语句中，必须通过 `resultSets` 属性为每个结果集指定一个名字，多个名字使用逗号隔开。

  ```xml
  <select id="selectBlog" resultSets="blogs,authors" resultMap="blogResult" statementType="CALLABLE">
    {call getBlogsAndAuthors(#{id,jdbcType=INTEGER,mode=IN})}
  </select>
  ```

  现在我们可以指定使用 “authors” 结果集的数据来填充 “author” 关联：

  ```xml
  <resultMap id="blogResult" type="Blog">
    <id property="id" column="id" />
    <result property="title" column="title"/>
    <association property="author" javaType="Author" resultSet="authors" column="author_id" foreignColumn="id">
      <id property="id" column="id"/>
      <result property="username" column="username"/>
      <result property="password" column="password"/>
      <result property="email" column="email"/>
      <result property="bio" column="bio"/>
    </association>
  </resultMap>
  ```

##### 集合

能够处理“有很多个”类型的关联。

场景：一个博客有很多文章（Post），在博客类中有一个这样的属性

```java
private List<Post> posts;
```

```xml
<collection property="posts" ofType="domain.blog.Post">
  <id property="id" column="post_id"/>
  <result property="subject" column="post_subject"/>
  <result property="body" column="post_body"/>
</collection>
```

和关联元素类似，我们可以使用几种方法。

* 集合的嵌套Select查询

  ```xml
  <resultMap id="blogResult" type="Blog">
    <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
  </resultMap>
  
  <select id="selectBlog" resultMap="blogResult">
    SELECT * FROM BLOG WHERE ID = #{id}
  </select>
  
  <select id="selectPostsForBlog" resultType="Post">
    SELECT * FROM POST WHERE BLOG_ID = #{id}
  </select>
  ```

  `ofType`将JavaBean属性的类型和集合存储的类型区分开来

* 集合的嵌套结果映射

  sql语句

  ```xml
  <select id="selectBlog" resultMap="blogResult">
    select
    B.id as blog_id,
    B.title as blog_title,
    B.author_id as blog_author_id,
    P.id as post_id,
    P.subject as post_subject,
    P.body as post_body,
    from Blog B
    left outer join Post P on B.id = P.blog_id
    where B.id = #{id}
  </select>
  ```

  ```xml
  <resultMap id="blogResult" type="Blog">
    <id property="id" column="blog_id" />
    <result property="title" column="blog_title"/>
    <collection property="posts" ofType="Post" resultMap="blogPostResult" columnPrefix="post_"/>
  </resultMap>
  
  <resultMap id="blogPostResult" type="Post">
    <id property="id" column="id"/>
    <result property="subject" column="subject"/>
    <result property="body" column="body"/>
  </resultMap>
  ```

* 多结果集

  像关联元素那样，我们可以通过执行存储过程实现，它会执行两个查询并返回两个结果集，一个是博客的结果集，另一个是文章的结果集：

  ```
  SELECT * FROM BLOG WHERE ID = #{id}
  
  SELECT * FROM POST WHERE BLOG_ID = #{id}
  ```

  在映射语句中，必须通过 `resultSets` 属性为每个结果集指定一个名字，多个名字使用逗号隔开。

  ```xml
  <select id="selectBlog" resultSets="blogs,posts" resultMap="blogResult">
    {call getBlogsAndPosts(#{id,jdbcType=INTEGER,mode=IN})}
  </select>
  ```

  我们指定 “posts” 集合将会使用存储在 “posts” 结果集中的数据进行填充：

  ```xml
  <resultMap id="blogResult" type="Blog">
    <id property="id" column="id" />
    <result property="title" column="title"/>
    <collection property="posts" ofType="Post" resultSet="posts" column="id" foreignColumn="blog_id">
      <id property="id" column="id"/>
      <result property="subject" column="subject"/>
      <result property="body" column="body"/>
    </collection>
  </resultMap>
  ```

  

##### 鉴别器

有时候，一个数据库查询可能会返回多个不同的结果集（但总体上还是有一定的联系的）。 鉴别器（discriminator）元素就是被设计来应对这种情况的，另外也能处理其它情况，例如类的继承层次结构。 鉴别器的概念很好理解——它很像 Java 语言中的 switch 语句。

一个鉴别器的定义需要指定 column 和 javaType 属性。column 指定了 MyBatis 查询被比较值的地方。 而 javaType 用来确保使用正确的相等测试（虽然很多情况下字符串的相等测试都可以工作）。

```xml
<resultMap id="vehicleResult" type="Vehicle">
  <id property="id" column="id" />
  <result property="vin" column="vin"/>
  <result property="year" column="year"/>
  <result property="make" column="make"/>
  <result property="model" column="model"/>
  <result property="color" column="color"/>
  <discriminator javaType="int" column="vehicle_type">
    <case value="1" resultType="carResult">
      <result property="doorCount" column="door_count" />
    </case>
    <case value="2" resultType="truckResult">
      <result property="boxSize" column="box_size" />
      <result property="extendedCab" column="extended_cab" />
    </case>
    <case value="3" resultType="vanResult">
      <result property="powerSlidingDoor" column="power_sliding_door" />
    </case>
    <case value="4" resultType="suvResult">
      <result property="allWheelDrive" column="all_wheel_drive" />
    </case>
  </discriminator>
</resultMap>
```

**Notice**

 在这个示例中，MyBatis 会从结果集中得到每条记录，然后比较它的 vehicle type 值。 如果它匹配任意一个鉴别器的 case，就会使用这个 case 指定的结果映射。 这个过程是互斥的，也就是说，剩余的结果映射将被忽略（除非它是扩展的）。 如果不能匹配任何一个 case，MyBatis 就只会使用鉴别器块外定义的结果映射 。

拓展形式为：

```xml
<resultMap id="carResult" type="Car" extends="vehicleResult">
  <result property="doorCount" column="door_count" />
</resultMap>
```



#### 自动映射

当自动映射查询结果时，MyBatis 会获取结果中返回的列名并在 Java 类中查找相同名字的属性（忽略大小写）。 这意味着如果发现了 *ID* 列和 *id* 属性，MyBatis 会将列 *ID* 的值赋给 *id* 属性。

通常数据库列使用大写字母组成的单词命名，单词间用下划线分隔；而 Java 属性一般遵循驼峰命名法约定。为了在这两种命名方式之间启用自动映射，需要将 `mapUnderscoreToCamelCase` 设置为 true。

- `NONE` - 禁用自动映射。仅对手动映射的属性进行映射。
- `PARTIAL` - 对除在内部定义了嵌套结果映射（也就是连接的属性）以外的属性进行映射
- `FULL` - 自动映射所有属性。

## 缓存

MyBatis 内置了一个强大的事务性查询缓存机制，它可以非常方便地配置和定制。 为了使它更加强大而且易于配置，我们对 MyBatis 3 中的缓存实现进行了许多改进。

默认情况下，只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。 要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：

```
<cache/>
```

基本上就是这样。这个简单语句的效果如下:

- 映射语句文件中的所有 select 语句的结果将会被缓存。
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

**提示** 缓存只作用于 cache 标签所在的映射文件中的语句。如果你混合使用 Java API 和 XML 映射文件，在共用接口中的语句将不会被默认缓存。你需要使用 @CacheNamespaceRef 注解指定缓存作用域。

这些属性可以通过 cache 元素的属性来修改。比如：

```
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。

可用的清除策略有：

- `LRU` – 最近最少使用：移除最长时间不被使用的对象。
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

默认的清除策略是 LRU。

flushInterval（刷新间隔）属性可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。 默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。

size（引用数目）属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。

readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。





# 动态SQL

## if

```xml
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```

## choose,when,otherwise

有时候不想应用到所有的条件语句，而只想从中选择一项，针对这种情况，Mybatis提供了Choose元素，它有点像Java中的switch语句。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

## trim,where,set

在where语句中使用if的话，当if全部不匹配会导致查询失败，可以使用`where`标签，

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

*where* 元素只会在至少有一个子元素的条件返回 SQL 子句的情况下才去插入“WHERE”子句。而且，若语句的开头为“AND”或“OR”，*where* 元素也会将它们去除。

也可以使用自定义*trim*元素来定制*where*元素

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

 *prefixOverrides* 属性会忽略通过管道分隔的文本序列（注意此例中的空格也是必要的）。它的作用是移除所有指定在 *prefixOverrides* 属性中的内容，并且插入 *prefix* 属性中指定的内容 



类似的动态更新语句的方案是*set*。可以用于动态包含需要更新的列，而舍去其他的

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

这里会动态前置SET关键字并删除无关的逗号

同样的，可以用trim定制

```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

注意是删除后缀值

## foreach

对集合的遍历，通常在构建IN条件语句的时候

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

允许你指定一个集合声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头和结尾的字符串以及在迭代结果之间放置分隔符。可以将任何可迭代对象传递给*foreach*。当使用map对象时，index是键而item是值。其它的对象index是当前迭代的次数而item的值是本次迭代获取的元素。

## script

```java
    @Update({"<script>",
      "update Author",
      "  <set>",
      "    <if test='username != null'>username=#{username},</if>",
      "    <if test='password != null'>password=#{password},</if>",
      "    <if test='email != null'>email=#{email},</if>",
      "    <if test='bio != null'>bio=#{bio}</if>",
      "  </set>",
      "where id=#{id}",
      "</script>"})
    void updateAuthorValues(Author author);
```

## bind

`bind` 元素可以从 OGNL 表达式中创建一个变量并将其绑定到上下文。比如：

```xml
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```

## 多数据库支持

一个配置了“_databaseId”变量的 databaseIdProvider 可用于动态代码中，这样就可以根据不同的数据库厂商构建特定的语句。比如下面的例子：

```xml
<insert id="insert">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    <if test="_databaseId == 'oracle'">
      select seq_users.nextval from dual
    </if>
    <if test="_databaseId == 'db2'">
      select nextval for seq_users from sysibm.sysdummy1"
    </if>
  </selectKey>
  insert into users values (#{id}, #{name})
</insert>
```



# Mybatis-Spring

## 简介

会帮助你将Mybatis代码无缝地整合到Spring中。它允许Mybatis参与到Spring的事务管理之中，创建映射器mapper和SqlSession并注入到bean中，以及将Mybatis的异常转换为Spring的`DataAcessException`。

## 入门

### 安装

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>2.0.3</version>
</dependency>
```

### 快速使用

要和 Spring 一起使用 MyBatis，需要在 Spring 应用上下文中定义至少两样东西：一个 `SqlSessionFactory` 和至少一个数据映射器类。

在 MyBatis-Spring 中，可使用 `SqlSessionFactoryBean`来创建 `SqlSessionFactory`。 要配置这个工厂 bean，只需要把下面代码放在 Spring 的 XML 配置文件中：

```java
@Bean
public SqlSessionFactory sqlSessionFactory() throws Exception {
  SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
  factoryBean.setDataSource(dataSource());
  return factoryBean.getObject();
}
```

注意：`SqlSessionFactory` 需要一个 `DataSource`（数据源）。 这可以是任意的 `DataSource`，只需要和配置其它 Spring 数据库连接一样配置它就可以了。

假设你定义了一个如下的 mapper 接口：

```java
public interface UserMapper {
  @Select("SELECT * FROM users WHERE id = #{userId}")
  User getUser(@Param("userId") String userId);
} 
```

那么可以通过 `MapperFactoryBean` 将接口加入到 Spring 中:

```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

需要注意的是：所指定的映射器类**必须**是一个接口，而不是具体的实现类。在这个示例中，通过注解来指定 SQL 语句，但是也可以使用 MyBatis 映射器的 XML 配置文件。

配置好之后，你就可以像 Spring 中普通的 bean 注入方法那样，将映射器注入到你的业务或服务对象中。`MapperFactoryBean` 将会负责 `SqlSession` 的创建和关闭。如果使用了 Spring 的事务功能，那么当事务完成时，session 将会被提交或回滚。最终任何异常都会被转换成 Spring 的 `DataAccessException` 异常。

使用 Java 代码来配置的方式如下：

```java
@Bean
public UserMapper userMapper() throws Exception {
  SqlSessionTemplate sqlSessionTemplate = new SqlSessionTemplate(sqlSessionFactory());
  return sqlSessionTemplate.getMapper(UserMapper.class);
}
```

要调用 MyBatis 的数据方法，只需一行代码：

```java
public class FooServiceImpl implements FooService {

  private final UserMapper userMapper;

  public FooServiceImpl(UserMapper userMapper) {
    this.userMapper = userMapper;
  }

  public User doSomeBusinessStuff(String userId) {
    return this.userMapper.getUser(userId);
  }
}
```

## SqlSessionFactoryBean

`SqlSessionFactoryBean`实现了Spring的`FactoryBean`接口，这意味着有Spring最终创建的bean并不是`SqlSessionFactoryBean`而是它的`getObject()`方法的返回结果。如此，Spring将会在应用启动时为你创建`SqlSessionFactory`，但是，也不需要直接使用`SqlSessionFactory`,会被注入到`MapperFactoryBean`或其它继承于`SqlSessionDaoSupport`的DAO中。

### 属性

`SqlSessionFactory` 有一个唯一的必要属性：用于 JDBC 的 `DataSource`。这可以是任意的 `DataSource` 对象，它的配置方法和其它 Spring 数据库连接是一样的。

一个常用的属性是 `configLocation`，它用来指定 MyBatis 的 XML 配置文件路径。它在需要修改 MyBatis 的基础配置非常有用。通常，基础配置指的是 `<settings>` 或 `<typeAliases>` 元素。

需要注意的是，这个配置文件并不需要是一个完整的Mybatis配置。确切地说，任何`<environments>`、`<DataSource>`和`<transactionManager>`都会被忽略。

## 事务

MyBatis-Spring 借助了 Spring 中的 DataSourceTransactionManager 来实现事务管理。

一旦配置好了 Spring 的事务管理器，你就可以在 Spring 中按你平时的方式来配置事务。并且支持 @Transactional 注解和 AOP 风格的配置。在事务处理期间，一个单独的 `SqlSession` 对象将会被创建和使用。当事务完成时，这个 session 会以合适的方式提交或回滚。

事务配置好了以后，MyBatis-Spring 将会透明地管理事务。这样在你的 DAO 类中就不需要额外的代码了。

### 标准配置

要开启 Spring 的事务处理功能，在 Spring 的配置文件中创建一个 `DataSourceTransactionManager` 对象：

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <constructor-arg ref="dataSource" />
</bean>
@Bean
public DataSourceTransactionManager transactionManager() {
  return new DataSourceTransactionManager(dataSource());
}
```

## 使用SqlSession

 用 MyBatis-Spring 之后，你不再需要直接使用 `SqlSessionFactory` 了，因为你的 bean 可以被注入一个线程安全的 `SqlSession`，它能基于 Spring 的事务配置来自动提交、回滚、关闭 session。 

### SqlSessionTemplate

`SqlSessionTemplate` 是 MyBatis-Spring 的核心。作为 `SqlSession` 的一个实现，这意味着可以使用它无缝代替你代码中已经在使用的 `SqlSession`。`SqlSessionTemplate` 是线程安全的，可以被多个 DAO 或映射器所共享使用。

当调用 SQL 方法时（包括由 `getMapper()` 方法返回的映射器中的方法），`SqlSessionTemplate` 将会保证使用的 `SqlSession` 与当前 Spring 的事务相关。此外，它管理 session 的生命周期，包含必要的关闭、提交或回滚操作。另外，它也负责将 MyBatis 的异常翻译成 Spring 中的 `DataAccessExceptions`。

由于模板可以参与到 Spring 的事务管理中，并且由于其是线程安全的，可以供多个映射器类使用，你应该**总是**用 `SqlSessionTemplate` 来替换 MyBatis 默认的 `DefaultSqlSession` 实现。在同一应用程序中的不同类之间混杂使用可能会引起数据一致性的问题。

可以使用 `SqlSessionFactory` 作为构造方法的参数来创建 `SqlSessionTemplate` 对象。

```java
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
@Bean
public SqlSessionTemplate sqlSession() throws Exception {
  return new SqlSessionTemplate(sqlSessionFactory());
}
```

现在，这个 bean 就可以直接注入到你的 DAO bean 中了。你需要在你的 bean 中添加一个 SqlSession 属性，就像下面这样：

```java
public class UserDaoImpl implements UserDao {

  private SqlSession sqlSession;

  public void setSqlSession(SqlSession sqlSession) {
    this.sqlSession = sqlSession;
  }

  public User getUser(String userId) {
    return sqlSession.selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);
  }
}
```

按下面这样，注入 `SqlSessionTemplate`：

```xml
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">
  <property name="sqlSession" ref="sqlSession" />
</bean>
```

`SqlSessionTemplate` 还有一个接收 `ExecutorType` 参数的构造方法。这允许你使用如下 Spring 配置来批量创建对象，例如批量创建一些 SqlSession：

```java
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
  <constructor-arg index="1" value="BATCH" />
</bean>
@Bean
public SqlSessionTemplate sqlSession() throws Exception {
  return new SqlSessionTemplate(sqlSessionFactory(), ExecutorType.BATCH);
}
```

现在所有的映射语句可以进行批量操作了，可以在 DAO 中编写如下的代码

```java
public void insertUsers(List<User> users) {
  for (User user : users) {
    sqlSession.insert("org.mybatis.spring.sample.mapper.UserMapper.insertUser", user);
  }
}
```

注意，只需要在希望语句执行的方法与 `SqlSessionTemplate` 中的默认设置不同时使用这种配置。



### SqlSessionDaoSupport

`SqlSessionDaoSupport` 是一个抽象的支持类，用来为你提供 `SqlSession`。调用 `getSqlSession()` 方法你会得到一个 `SqlSessionTemplate`，之后可以用于执行 SQL 方法，就像下面这样:

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserDao {
  public User getUser(String userId) {
    return getSqlSession().selectOne("org.mybatis.spring.sample.mapper.UserMapper.getUser", userId);
  }
}
```



## 注入映射器

与其在数据访问对象（DAO）中手工编写使用 `SqlSessionDaoSupport` 或 `SqlSessionTemplate` 的代码，还不如让 Mybatis-Spring 为你创建一个线程安全的映射器，这样你就可以直接注入到其它的 bean 中了：

```xml
<bean id="fooService" class="org.mybatis.spring.sample.service.FooServiceImpl">
  <constructor-arg ref="userMapper" />
</bean>
```

注入完毕后，映射器就可以在你的应用逻辑代码中使用了：

```java
public class FooServiceImpl implements FooService {

  private final UserMapper userMapper;

  public FooServiceImpl(UserMapper userMapper) {
    this.userMapper = userMapper;
  }

  public User doSomeBusinessStuff(String userId) {
    return this.userMapper.getUser(userId);
  }
}
```

注意代码中并没有任何的对 `SqlSession` 或 MyBatis 的引用。你也不需要担心创建、打开、关闭 session，MyBatis-Spring 将为你打理好一切。

### 注册映射器

* XML配置

  ```xml
  <bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
  </bean>
  ```

* Java配置

  ```java
  @Bean
  public MapperFactoryBean<UserMapper> userMapper() throws Exception {
    MapperFactoryBean<UserMapper> factoryBean = new MapperFactoryBean<>(UserMapper.class);
    factoryBean.setSqlSessionFactory(sqlSessionFactory());
    return factoryBean;
  }
  ```

### 发现映射器

```java
@Configuration
@MapperScan("org.mybatis.spring.sample.mapper")
public class AppConfig {
  // ...
}
```

# Mybatis-Spring-Boot

## 安装

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```

## 快速启动

Spring Boot帮助做的：

* 自动检测存在的`DataSource`
* 将使用`DataSource`作为输入创建并注册一个`SqlSessionFactory`示例

* 将创建并注册从`SqlSessionFactory`中获取的`SqlSessionTemplate`实例
* 自动扫描映射器，将它们链接到`SqlSessionTemplate`并注册到Spring上下文中，所以可以直接注入使用

## 配置

在`application.properties`中配置，

| Property                                 | Description                                                  |
| :--------------------------------------- | :----------------------------------------------------------- |
| `config-location`                        | Location of MyBatis xml config file.                         |
| `check-config-location`                  | Indicates whether perform presence check of the MyBatis xml config file. |
| `mapper-locations`                       | Locations of Mapper xml config file.                         |
| `type-aliases-package`                   | Packages to search for type aliases. (Package delimiters are “`,; \t\n`”) |
| `type-aliases-super-type`                | The super class for filtering type alias. If this not specifies, the MyBatis deal as type alias all classes that searched from `type-aliases-package`. |
| `type-handlers-package`                  | Packages to search for type handlers. (Package delimiters are “`,; \t\n`”) |
| `executor-type`                          | Executor type: `SIMPLE`, `REUSE`, `BATCH`                    |
| `default-scripting-language-driver`      | The default scripting language driver class. This feature requires to use together with mybatis-spring 2.0.2+. |
| `configuration-properties`               | Externalized properties for MyBatis configuration. Specified properties can be used as placeholder on MyBatis config file and Mapper file. For detail see the [MyBatis reference page](http://www.mybatis.org/mybatis-3/configuration.html#properties). |
| `lazy-initialization`                    | Whether enable lazy initialization of mapper bean. Set `true` to enable lazy initialization. This feature requires to use together with mybatis-spring 2.0.2+. |
| `configuration.*`                        | Property keys for `Configuration` bean provided by MyBatis Core. About available nested properties see the [MyBatis reference page](http://www.mybatis.org/mybatis-3/configuration.html#settings). **NOTE**: This property cannot be used at the same time with the `config-location`. |
| `scripting-language-driver.thymeleaf.*`  | Property keys for `ThymeleafLanguageDriverConfig` bean provided by MyBatis Thymeleaf. About available nested properties see the [MyBatis Thymeleaf reference page](http://www.mybatis.org/thymeleaf-scripting/user-guide.html#_configuration_properties). |
| `scripting-language-driver.freemarker.*` | Properties keys for `FreeMarkerLanguageDriverConfig` bean provided by MyBatis FreeMarker. About available nested properties see the [MyBatis FreeMarker reference page](http://www.mybatis.org/freemarker-scripting/#Configuration). This feature requires to use together with mybatis-freemarker 1.2.0+. |
| `scripting-language-driver.velocity.*`   | Properties keys for `VelocityLanguageDriverConfig` bean provided by MyBatis Velocity. About available nested properties see the [MyBatis Velocity reference page](http://www.mybatis.org/velocity-scripting/#Configuration). This feature requires to use together with mybatis-velocity 2.1.0+. |