
# 三万字带你彻底吃透MyBatis源码！！

原创 冰河

[↓↓↓](javascript:)  

冰河技术  

[↑↑↑](javascript:)

_2021-01-04 00:00_

收录于合集 #MyBatis系列 7个

点击上方蓝色“冰河技术”，关注并选择“设为星标”

持之以恒，贵在坚持，每天进步一点点！

  

![图片](assets/1700476395-a0eb9bd0735ee6a77d2aba9046ba3cb3.png)

  

作者个人研发的在高并发场景下，提供的简单、稳定、可扩展的延迟消息队列框架，具有精准的定时任务和延迟队列处理功能。自开源半年多以来，已成功为十几家中小型企业提供了精准定时调度方案，经受住了生产环境的考验。为使更多童鞋受益，现给出开源框架地址：

https://github.com/sunshinelyz/mykit-delay  

PS: 欢迎各位Star源码，也可以pr你牛逼哄哄的代码。      

## 写在前面

随着互联网的发展，越来越多的公司摒弃了Hibernate，而选择拥抱了MyBatis。而且，很多大厂在面试的时候喜欢问MyBatis底层的原理和源码实现。总之，MyBatis几乎成为了Java开发人员必须深入掌握的框架技术，今天，我们就一起来深入分析MyBatis源码。文章有点长，建议先收藏后慢慢研究。整体三万字左右，全程高能，小伙伴们可慢慢研究。

文章已收录到：

https://github.com/sunshinelyz/technology-binghe

https://gitee.com/binghe001/technology-binghe

## MyBatis源码解析

大家应该都知道Mybatis源码也是对Jbdc的再一次封装,不管怎么进行包装,还是会有获取链接、preparedStatement、封装参数、执行这些步骤的。

## 配置解析过程

```plain
String resource = "mybatis-config.xml";
//1.读取resources下面的mybatis-config.xml文件
InputStream inputStream = Resources.getResourceAsStream(resource);
//2.使用SqlSessionFactoryBuilder创建SqlSessionFactory
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
//3.通过sqlSessionFactory创建SqlSession
SqlSession sqlSession = sqlSessionFactory.openSession();
```

### Resources.getResourceAsStream(resource)读取文件

```plain
public static InputStream getResourceAsStream(String resource) throws IOException {
 return getResourceAsStream(null, resource);
} 
//loader赋值为null
public static InputStream getResourceAsStream(ClassLoader loader, String resource) throws IOException {
 InputStream in = classLoaderWrapper.getResourceAsStream(resource, loader);
 if (in == null) {
  throw new IOException("Could not find resource " + resource);
 } 
 return in;
}
//classLoader为null
public InputStream getResourceAsStream(String resource, ClassLoader classLoader) {
 return getResourceAsStream(resource, getClassLoaders(classLoader));
} 
//classLoader类加载
InputStream getResourceAsStream(String resource, ClassLoader[] classLoader) {
 for (ClassLoader cl : classLoader) {
  if (null != cl) {
   //加载指定路径文件流
   InputStream returnValue = cl.getResourceAsStream(resource);
   // now, some class loaders want this leading "/", so we'll add it and try again if we didn't find the resource
   if (null == returnValue) {
    returnValue = cl.getResourceAsStream("/" + resource);
   } 
   if (null != returnValue) {
    return returnValue;
   }
  }
 } 
 return null;
}
```

总结：主要是通过ClassLoader.getResourceAsStream()方法获取指定的classpath路径下的Resource 。

### 通过SqlSessionFactoryBuilder创建SqlSessionFactory

```java
//SqlSessionFactoryBuilder是一个建造者模式
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
public SqlSessionFactory build(InputStream inputStream) {
 return build(inputStream, null, null);
}
//XMLConfigBuilder也是建造者模式
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
 try {
  XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
  return build(parser.parse());
 } catch (Exception e) {
  throw ExceptionFactory.wrapException("Error building SqlSession.", e);
 } finally {
  ErrorContext.instance().reset();
  try {
   inputStream.close();
  } catch (IOException e) {
   // Intentionally ignore. Prefer previous error.
  }
 }
}
//接下来进入XMLConfigBuilder构造函数
public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
 this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);
}
//接下来进入this后,初始化Configuration
private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
 super(new Configuration());
 ErrorContext.instance().resource("SQL Mapper Configuration");
 this.configuration.setVariables(props);
 this.parsed = false;
 this.environment = environment;
 this.parser = parser;
}
//其中parser.parse()负责解析xml,build(configuration)创建SqlSessionFactory
return build(parser.parse());
```

#### parser.parse()解析xml

```plain
public Configuration parse() {
 //判断是否重复解析
 if (parsed) {
  throw new BuilderException("Each XMLConfigBuilder can only be used once.");
 } 
 parsed = true;
 //读取配置文件一级节点configuration
 parseConfiguration(parser.evalNode("/configuration"));
 return configuration;
}
```

```plain
private void parseConfiguration(XNode root) {
 try {
  //properties 标签,用来配置参数信息，比如最常见的数据库连接信息
  propertiesElement(root.evalNode("properties"));
  Properties settings = settingsAsProperties(root.evalNode("settings"));
  loadCustomVfs(settings);
  loadCustomLogImpl(settings);
  //实体别名两种方式:1.指定单个实体;2.指定包
  typeAliasesElement(root.evalNode("typeAliases"));
  //插件
  pluginElement(root.evalNode("plugins"));
  //用来创建对象(数据库数据映射成java对象时)
  objectFactoryElement(root.evalNode("objectFactory"));
  objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
  reflectorFactoryElement(root.evalNode("reflectorFactory"));
  settingsElement(settings);
  // read it after objectFactory and objectWrapperFactory issue #631
  //数据库环境
  environmentsElement(root.evalNode("environments"));
  databaseIdProviderElement(root.evalNode("databaseIdProvider"));
  //数据库类型和Java数据类型的转换
  typeHandlerElement(root.evalNode("typeHandlers"));
  //这个是对数据库增删改查的解析
  mapperElement(root.evalNode("mappers"));
 } catch (Exception e) {
  throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
 }
}
```

总结:parseConfiguration完成的是解析configuration下的标签

```plain
private void mapperElement(XNode parent) throws Exception {
 if (parent != null) {
   for (XNode child : parent.getChildren()) {
   //解析<package name=""/>
   if ("package".equals(child.getName())) {
    String mapperPackage = child.getStringAttribute("name");
    //包路径存到mapperRegistry中
    configuration.addMappers(mapperPackage);
   } else {
    //解析<mapper url="" class="" resource=""></mapper>
    String resource = child.getStringAttribute("resource");
    String url = child.getStringAttribute("url");
    String mapperClass = child.getStringAttribute("class");
    if (resource != null && url == null && mapperClass == null) {
     ErrorContext.instance().resource(resource);
     //读取Mapper.xml文件
     InputStream inputStream = Resources.getResourceAsStream(resource);
     XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream,
     configuration, resource, configuration.getSqlFragments());
     mapperParser.parse();
    } else if (resource == null && url != null && mapperClass == null) {
     ErrorContext.instance().resource(url);
     InputStream inputStream = Resources.getUrlAsStream(url);
     XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream,
     configuration, url, configuration.getSqlFragments());
     mapperParser.parse();
    } else if (resource == null && url == null && mapperClass != null) {
     Class<?> mapperInterface = Resources.classForName(mapperClass);
     configuration.addMapper(mapperInterface);
    } else {
     throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
    }
   }
  }
 }
}
```

总结: 通过解析configuration.xml文件,获取其中的Environment、Setting,重要的是将下的所有解析出来之后添加到 Configuration,Configuration类似于配置中心,所有的配置信息都在这里。

#### mapperParser.parse()对 Mapper 映射器的解析

```plain
public void parse() {
 if (!configuration.isResourceLoaded(resource)) {
  //解析所有的子标签
  configurationElement(parser.evalNode("/mapper"));
  configuration.addLoadedResource(resource);
  //把namespace（接口类型）和工厂类绑定起来
  bindMapperForNamespace();
 }
 parsePendingResultMaps();
 parsePendingCacheRefs();
 parsePendingStatements();
} 
//这里面解析的是Mapper.xml的标签
private void configurationElement(XNode context) {
 try {
  String namespace = context.getStringAttribute("namespace");
  if (namespace == null || namespace.equals("")) {
   throw new BuilderException("Mapper's namespace cannot be empty");
  } 
  builderAssistant.setCurrentNamespace(namespace);
  //对其他命名空间缓存配置的引用
  cacheRefElement(context.evalNode("cache-ref"));
  //对给定命名空间的缓存配置
  cacheElement(context.evalNode("cache"));
  parameterMapElement(context.evalNodes("/mapper/parameterMap"));
  //是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象
  resultMapElements(context.evalNodes("/mapper/resultMap"));
  //可被其他语句引用的可重用语句块
  sqlElement(context.evalNodes("/mapper/sql"));
  //获得MappedStatement对象(增删改查标签)
  buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
 } catch (Exception e) {
  throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
 }
}
//获得MappedStatement对象(增删改查标签)
private void buildStatementFromContext(List<XNode> list) {
 if (configuration.getDatabaseId() != null) {
  buildStatementFromContext(list, configuration.getDatabaseId());
 } 
 buildStatementFromContext(list, null);
}
//获得MappedStatement对象(增删改查标签)
private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
 //循环增删改查标签
 for (XNode context : list) {
  final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
  try {
   //解析insert/update/select/del中的标签
   statementParser.parseStatementNode();
  } catch (IncompleteElementException e) {
   configuration.addIncompleteStatement(statementParser);
  }
 }
}
public void parseStatementNode() {
 //在命名空间中唯一的标识符,可以被用来引用这条语句
 String id = context.getStringAttribute("id");
 //数据库厂商标识
 String databaseId = context.getStringAttribute("databaseId");
 if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
  return;
 } 
 String nodeName = context.getNode().getNodeName();
 SqlCommandType sqlCommandType =
 SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
 boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
 //flushCache和useCache都和二级缓存有关
 //将其设置为true后,只要语句被调用,都会导致本地缓存和二级缓存被清空,默认值:false
 boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
 //将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true
 boolean useCache = context.getBooleanAttribute("useCache", isSelect);
 boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);
 // Include Fragments before parsing
 XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
 includeParser.applyIncludes(context.getNode());
 //会传入这条语句的参数类的完全限定名或别名
 String parameterType = context.getStringAttribute("parameterType");
 Class<?> parameterTypeClass = resolveClass(parameterType);
 String lang = context.getStringAttribute("lang");
 LanguageDriver langDriver = getLanguageDriver(lang);
 // Parse selectKey after includes and remove them.
 processSelectKeyNodes(id, parameterTypeClass, langDriver);
 // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
 KeyGenerator keyGenerator;
 String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
 keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
 if (configuration.hasKeyGenerator(keyStatementId)) {
  keyGenerator = configuration.getKeyGenerator(keyStatementId);
 } else {
  keyGenerator = context.getBooleanAttribute("useGeneratedKeys", configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType)) ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
 } 
 SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
 StatementType statementType =
 StatementType.valueOf(context.getStringAttribute("statementType",
 StatementType.PREPARED.toString()));
 Integer fetchSize = context.getIntAttribute("fetchSize");
 Integer timeout = context.getIntAttribute("timeout");
 String parameterMap = context.getStringAttribute("parameterMap");
 //从这条语句中返回的期望类型的类的完全限定名或别名
 String resultType = context.getStringAttribute("resultType");
 Class<?> resultTypeClass = resolveClass(resultType);
 //外部resultMap的命名引用
 String resultMap = context.getStringAttribute("resultMap");
 String resultSetType = context.getStringAttribute("resultSetType");
 ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);
 String keyProperty = context.getStringAttribute("keyProperty");
 String keyColumn = context.getStringAttribute("keyColumn");
 String resultSets = context.getStringAttribute("resultSets");
 builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
 fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
 resultSetTypeEnum, flushCache, useCache, resultOrdered,
 keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
}
public MappedStatement addMappedStatement(
 String id,
 SqlSource sqlSource,
 StatementType statementType,
 SqlCommandType sqlCommandType,
 Integer fetchSize,
 Integer timeout,
 String parameterMap,
 Class<?> parameterType,
 String resultMap,
 Class<?> resultType,
 ResultSetType resultSetType,
 boolean flushCache,
 boolean useCache,
 boolean resultOrdered,
 KeyGenerator keyGenerator,
 String keyProperty,
 String keyColumn,
 String databaseId,
 LanguageDriver lang,
 String resultSets) {
 if (unresolvedCacheRef) {
  throw new IncompleteElementException("Cache-ref not yet resolved");
 } 
  id = applyCurrentNamespace(id, false);
  boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
  MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration,
  id, sqlSource, sqlCommandType)
  .resource(resource)
  .fetchSize(fetchSize)
  .timeout(timeout)
  .statementType(statementType)
  .keyGenerator(keyGenerator)
  .keyProperty(keyProperty)
  .keyColumn(keyColumn)
  .databaseId(databaseId)
  .lang(lang)
  .resultOrdered(resultOrdered)
  .resultSets(resultSets)
  .resultMaps(getStatementResultMaps(resultMap, resultType, id))
  .resultSetType(resultSetType)
  .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
  .useCache(valueOrDefault(useCache, isSelect))
  .cache(currentCache);
  ParameterMap statementParameterMap = getStatementParameterMap(parameterMap,
  parameterType, id);
  if (statementParameterMap != null) {
   statementBuilder.parameterMap(statementParameterMap);
  } 
  MappedStatement statement = statementBuilder.build();
  //持有在configuration中
  configuration.addMappedStatement(statement);
  return statement;
}
public void addMappedStatement(MappedStatement ms){
//ms.getId = mapper.UserMapper.getUserById
//ms = MappedStatement等于每一个增删改查的标签的里的数据
 mappedStatements.put(ms.getId(), ms);
}
//最终存放到mappedStatements中,mappedStatements存放的是一个个的增删改查
protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection").conflictMessageProducer((savedValue, targetValue) ->
". please check " + savedValue.getResource() + " and " + targetValue.getResource());
```

#### 解析bindMapperForNamespace()方法

把 namespace（接口类型）和工厂类绑定起来

```plain
private void bindMapperForNamespace() {
 //当前Mapper的命名空间
 String namespace = builderAssistant.getCurrentNamespace();
 if (namespace != null) {
  Class<?> boundType = null;
  try {
   //interface mapper.UserMapper这种
   boundType = Resources.classForName(namespace);
  } catch (ClassNotFoundException e) {
  } 
  if (boundType != null) {
   if (!configuration.hasMapper(boundType)) {
    configuration.addLoadedResource("namespace:" + namespace);
    configuration.addMapper(boundType);
   }
  }
 }
}
public <T> void addMapper(Class<T> type) {
 mapperRegistry.addMapper(type);
} 
public <T> void addMapper(Class<T> type) {
 if (type.isInterface()) {
  if (hasMapper(type)) {
   throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
  } 
  boolean loadCompleted = false;
  try {
   //接口类型(key)->工厂类
   knownMappers.put(type, new MapperProxyFactory<>(type));
   MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
   parser.parse();
   loadCompleted = true;
  } finally {
   if (!loadCompleted) {
    knownMappers.remove(type);
   }
  }
 }
}
```

#### 生成SqlSessionFactory对象

XMLMapperBuilder.parse()方法,是对 Mapper 映射器的解析里面有两个方法：

（1）configurationElement()解析所有的子标签,最终解析Mapper.xml中的insert/update/delete/select标签的id(全路径)组成key和整个标签和数据连接组成MappedStatement存放到Configuration中的 mappedStatements这个map里面。

（2）bindMapperForNamespace()是把接口类型(interface mapper.UserMapper)和工厂类存到放MapperRegistry中的knownMappers里面。

## SqlSessionFactory的创建

```plain
public SqlSessionFactory build(Configuration config) {
 return new DefaultSqlSessionFactory(config);
}
```

直接把Configuration当做参数,直接new一个DefaultSqlSessionFactory。

## SqlSession会话的创建过程

mybatis操作的时候跟数据库的每一次连接,都需要创建一个会话,我们用openSession()方法来创建。这个会话里面需要包含一个Executor用来执行 SQL。Executor又要指定事务类型和执行器的类型。

### 创建Transaction(两种方式)

| 属性  | 产生工厂类 | 产生事务 |
| --- | --- | --- |
| JDBC | JbdcTransactionFactory | JdbcTransaction |
| MANAGED | ManagedTransactionFactory | ManagedTransaction |

*   如果配置的是 JDBC,则会使用Connection 对象的 commit()、rollback()、close()管理事务。
    
*   如果配置成MANAGED,会把事务交给容器来管理,比如 JBOSS,Weblogic。
    

```plain
SqlSession sqlSession = sqlSessionFactory.openSession();
```

```plain
public SqlSession openSession() {
 //configuration中有默认赋值protected ExecutorType defaultExecutorType = ExecutorType.SIMPLE
 return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
}
```

```plain
<environments default="development">
<environment id="development">
<transactionManager type="JDBC"/>
<dataSource type="POOLED">
<property name="driver" value="${driver}"/>
<property name="url" value="${url}"/>
<property name="username" value="${username}"/>
<property name="password" value="${password}"/>
</dataSource>
</environment>
</environments>
```

### 创建Executor

```plain
//ExecutorType是SIMPLE,一共有三种SIMPLE(SimpleExecutor)、REUSE(ReuseExecutor)、BATCH(BatchExecutor)
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
 Transaction tx = null;
 try {
  //xml中的development节点
  final Environment environment = configuration.getEnvironment();
  //type配置的是Jbdc所以生成的是JbdcTransactionFactory工厂类
  final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
  //Jdbc生成JbdcTransactionFactory生成JbdcTransaction
  tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
  //创建CachingExecutor执行器
  final Executor executor = configuration.newExecutor(tx, execType);
  //创建DefaultSqlSession属性包括 Configuration、Executor对象
  return new DefaultSqlSession(configuration, executor, autoCommit);
 } catch (Exception e) {
  closeTransaction(tx); // may have fetched a connection so lets call
  close()
  throw ExceptionFactory.wrapException("Error opening session. Cause: " + e, e);
 } finally {
  ErrorContext.instance().reset();
 }
}
```

## 获得Mapper对象

```plain
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
```

```plain
public <T> T getMapper(Class<T> type) {
 return configuration.getMapper(type, this);
}
```

mapperRegistry.getMapper是从MapperRegistry的knownMappers里面取的,knownMappers里面存的是接口类型(interface mapper.UserMapper)和工厂类(MapperProxyFactory)。

```plain
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
 return mapperRegistry.getMapper(type, sqlSession);
}
```

从knownMappers的Map里根据接口类型(interface mapper.UserMapper)取出对应的工厂类。

```plain
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
 final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>)
 knownMappers.get(type);
 if (mapperProxyFactory == null) {
  throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
 } 
 try {
  return mapperProxyFactory.newInstance(sqlSession);
 } catch (Exception e) {
  throw new BindingException("Error getting mapper instance. Cause: " + e, e);
 }
}
public T newInstance(SqlSession sqlSession) {
 final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
 return newInstance(mapperProxy);
}
```

这里通过JDK动态代理返回代理对象MapperProxy(org.apache.ibatis.binding.MapperProxy@6b2ea799)

```plain
protected T newInstance(MapperProxy<T> mapperProxy) {
 //mapperInterface是interface mapper.UserMapper 
 return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new
 Class[] { mapperInterface }, mapperProxy);
}
```

```plain
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
```

## 执行SQL

```plain
User user = userMapper.getUserById(1);
```

### 调用invoke代理方法

由于所有的 Mapper 都是 MapperProxy 代理对象，所以任意的方法都是执行MapperProxy 的invoke()方法

```plain
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
 try {
  //判断是否需要去执行SQL还是直接执行方法
  if (Object.class.equals(method.getDeclaringClass())) {
   return method.invoke(this, args);
   //这里判断的是接口中的默认方法Default等
  } else if (isDefaultMethod(method)) {
   return invokeDefaultMethod(proxy, method, args);
  }
 } catch (Throwable t) {
  throw ExceptionUtil.unwrapThrowable(t);
 } 
    //获取缓存,保存了方法签名和接口方法的关系
 final MapperMethod mapperMethod = cachedMapperMethod(method);
 return mapperMethod.execute(sqlSession, args);
}
```

### 调用execute方法

这里使用的例子用的是查询所以走的是else分支语句。

```plain
public Object execute(SqlSession sqlSession, Object[] args) {
 Object result;
 //根据命令类型走不行的操作command.getType()是select
 switch (command.getType()) {
  case INSERT: {
   Object param = method.convertArgsToSqlCommandParam(args);
   result = rowCountResult(sqlSession.insert(command.getName(), param));
   break;
  } 
  case UPDATE: {
   Object param = method.convertArgsToSqlCommandParam(args);
   result = rowCountResult(sqlSession.update(command.getName(), param));
   break;
  } 
  case DELETE: {
   Object param = method.convertArgsToSqlCommandParam(args);
   result = rowCountResult(sqlSession.delete(command.getName(), param));
   break;
  } 
  case SELECT:
   if (method.returnsVoid() && method.hasResultHandler()) {
    executeWithResultHandler(sqlSession, args);
    result = null;
   } else if (method.returnsMany()) {
    result = executeForMany(sqlSession, args);
   } else if (method.returnsMap()) {
    result = executeForMap(sqlSession, args);
   } else if (method.returnsCursor()) {
    result = executeForCursor(sqlSession, args);
   } else {
    //将参数转换为SQL的参数
    Object param = method.convertArgsToSqlCommandParam(args);
    result = sqlSession.selectOne(command.getName(), param);
    if (method.returnsOptional()
    && (result == null ||
    !method.getReturnType().equals(result.getClass()))) {
     result = Optional.ofNullable(result);
    }
   }
   break;
  case FLUSH:
   result = sqlSession.flushStatements();
   break;
  default:
   throw new BindingException("Unknown execution method for: " + command.getName());
 } 
 if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
  throw new BindingException("Mapper method '" + command.getName() + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
 } 
 return result;
}
```

### 调用selectOne其实是selectList

selectOne查询一个和查询多个其实是一样的。

```plain
public <T> T selectOne(String statement, Object parameter) {
 // Popular vote was to return null on 0 results and throw exception on too many.
 List<T> list = this.selectList(statement, parameter);
 if (list.size() == 1) {
  return list.get(0);
 } else if (list.size() > 1) {
  throw new TooManyResultsException("Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
 } else {
  return null;
 }
}
```

```plain
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
 try {
  //从Configuration里的mappedStatements里根据key(id的全路径)获取MappedStatement 对象
  MappedStatement ms = configuration.getMappedStatement(statement);
  return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
 } catch (Exception e) {
  throw ExceptionFactory.wrapException("Error querying database. Cause: " + e, e);
 } finally {
  ErrorContext.instance().reset();
 }
}
```

#### mappedStatements对象如图

![图片](assets/1700476395-2db55a40aba332a79301b9c0a972ec25.jpg)

#### MappedStatement对象如图

![图片](assets/1700476395-b310248475486e4a1f1e2e37924f62d0.jpg)

### 执行query方法

#### 创建CacheKey

从 BoundSql 中获取SQL信息,创建 CacheKey。这个CacheKey就是缓存的Key。

```plain
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
 //创建缓存Key
 BoundSql boundSql = ms.getBoundSql(parameterObject);
 //key = -575461213:-771016147:mapper.UserMapper.getUserById:0:2147483647:select * from test_user where id = ?:1:development
 CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
 return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

```plain
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
 Cache cache = ms.getCache();
 if (cache != null) {
  flushCacheIfRequired(ms);
  if (ms.isUseCache() && resultHandler == null) {
   ensureNoOutParams(ms, boundSql);
   @SuppressWarnings("unchecked")
   List<E> list = (List<E>) tcm.getObject(cache, key);
   if (list == null) {
    list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    tcm.putObject(cache, key, list); // issue #578 and #116
   } 
   return list;
  }
 }
 return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

#### 清空本地缓存

```plain
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
 ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
 if (closed) {
  throw new ExecutorException("Executor was closed.");
 } 
 //queryStack 用于记录查询栈，防止递归查询重复处理缓存
 //flushCache=true 的时候，会先清理本地缓存（一级缓存）
 if (queryStack == 0 && ms.isFlushCacheRequired()) {
  //清空本地缓存
  clearLocalCache();
 } 
 List<E> list;
 try {
  queryStack++;
  list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
  if (list != null) {
   handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
  } else {
   //如果没有缓存,会从数据库查询：queryFromDatabase()
   list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
  }
 } finally {
  queryStack--;
 } 
 if (queryStack == 0) {
  for (DeferredLoad deferredLoad : deferredLoads) {
  deferredLoad.load();
  } 
  // issue #601
  deferredLoads.clear();
  //如果 LocalCacheScope == STATEMENT,会清理本地缓存
  if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
   // issue #482
   clearLocalCache();
  }
 } 
 return list;
}
```

#### 从数据库查询

```plain
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
 List<E> list;
 //先在缓存用占位符占位
 localCache.putObject(key, EXECUTION_PLACEHOLDER);
 try {
  //执行Executor 的 doQuery(),默认是SimpleExecutor
  list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
 } finally {
  //执行查询后,移除占位符
  localCache.removeObject(key);
 } 
 //从新放入数据
 localCache.putObject(key, list);
 if (ms.getStatementType() == StatementType.CALLABLE) {
  localOutputParameterCache.putObject(key, parameter);
 } 
 return list;
}
```

#### 执行doQuery

```plain
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
 Statement stmt = null;
 try {
  Configuration configuration = ms.getConfiguration();
  StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
  stmt = prepareStatement(handler, ms.getStatementLog());
  return handler.query(stmt, resultHandler);
 } finally {
  closeStatement(stmt);
 }
}
```

## 源码总结

总体上来说，MyBatis的源码还是比较简单的，只要大家踏下心来，花个两三天仔细研究下，基本上都能弄明白源码的主体脉络。

**好了，今天就到这儿吧，我是冰河，大家有啥问题可以在下方留言，也可以加我微信，一起交流技术，一起进阶，一起牛逼~~**

![图片](assets/1700476395-8cae56536dcbcac03e98f6fb2df73810.jpg)

## 冰河原创PDF

关注 **冰河技术** 微信公众号：

回复 “**并发编程**” 领取《深入理解高并发编程（第1版）》PDF文档。

回复 “**并发源码**” 领取《并发编程核心知识（源码分析篇 第1版）》PDF文档。

回复 ”**限流**“ 领取《亿级流量下的分布式解决方案》PDF文档。

回复 “**设计模式**” 领取《深入浅出Java23种设计模式》PDF文档。

回复 “**Java8新特性**” 领取 《Java8新特性教程》PDF文档。

回复 “**分布式存储**” 领取《跟冰河学习分布式存储技术》 PDF文档。

回复 “**Nginx**” 领取《跟冰河学习Nginx技术》PDF文档。

回复 “**互联网工程**” 领取《跟冰河学习互联网工程技术》PDF文档。

## 写在最后

如果你觉得冰河写的还不错，请微信搜索并关注「 **冰河技术** 」微信公众号，跟冰河学习高并发、分布式、微服务、大数据、互联网和云原生技术，「 **冰河技术** 」微信公众号更新了大量技术专题，每一篇技术文章干货满满！不少读者已经通过阅读「 **冰河技术** 」微信公众号文章，成功跳槽到大厂；也有不少读者实现了技术上的飞跃，成为公司的技术骨干！如果你也想像他们一样提升自己的能力，实现技术能力的飞跃，进大厂，升职加薪，那就关注「 **冰河技术** 」微信公众号吧，每天更新超硬核技术干货，让你对如何提升技术能力不再迷茫！

![图片](assets/1700476395-c8c487065c4cf8013aa1cdc349090b9d.png)

收录于合集 #MyBatis系列

 7个

上一篇MyBatis源码分析之——面试官问我如何获得Mapper对象？我是这样说的！下一篇三天撸完了MyBatis，各位随便问！！
