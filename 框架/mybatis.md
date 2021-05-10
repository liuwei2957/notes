### Mybatis原理

1. sqlsessionFactoryBuilder生成sqlsessionFactory（单例）
2. 工厂模式生成sqlsession执行sql以及控制事务
3. Mybatis通过动态代理使Mapper（sql映射器）接口能运行起来即为接口生成代理对象将sql查询到结果映射成pojo

#### sqlSessionFactory构建过程

1. 解析并读取配置中的xml创建Configuration对象 （单例）
2. 使用Configruation类去创建sqlSessionFactory（builder模式）

### Mybatis一级缓存与二级缓存

 默认情况下一级缓存是开启的，而且是不能关闭的 

- 一级缓存是指 SqlSession 级别的缓存 原理：使用的数据结构是一个 map，如果两次中间出现 commit 操作 （修改、添加、删除），本 sqlsession 中的一级缓存区域全部清空
- 二级缓存是指可以跨 SqlSession 的缓存。是 mapper 级别的缓存；原理：是通过 CacheExecutor 实现的。CacheExecutor其实是 Executor 的代理对象