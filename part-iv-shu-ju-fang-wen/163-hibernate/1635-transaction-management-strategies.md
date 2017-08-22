### 16.3.5 事务管理策略

无论是`TransactionTemplate`或者是`TransactionInterceptor`都将实际的事务处理代理到`PlatformTransactionManager`实例上来进行处理的，这个实例的实现可以是一个`HibernateTransactionManager`\(包含一个Hibernate的`SessionFactory`通过使用`ThreadLocal`的`Session`\)，也可以是`JatTransactionManager`\(代理到容器的JTA子系统\)。开发者甚至可以使用一个自定义的`PlatformTransactionManager`的实现。现在，如果应用有需求需要部署分布式事务的话，只是一个配置变化，就可以从本地Hibernate事务管理切换到JTA。简单地用Spring的JTA事务实现来替换Hibernate事务管理器即可。因为引用的`PlatformTransactionManager`的是通用事务管理API，事务管理器之间的切换是无需修改代码的。

对于那些跨越了多个Hibernate会话工厂的分布式事务，只需要将`JtaTransactionManager`和多个`LocalSessionFactoryBean`定义相结合即可。每个DAO之后会获取一个特定的`SessionFactory`引用。如果所有底层JDBC数据源都是事务性容器，那么只要使用`JtaTransactionManager`作为策略实现，业务服务就可以划分任意数量的DAO和任意数量的会话工厂的事务。

无论是`HibernateTransactionManager`还是`JtaTransactionManager`都允许使用JVM级别的缓存来处理Hibernate，无需基于容器的事务管理器查找，或者JCA连接器（如果开发者没有使用EJB来实例化事务的话）。

`HibernateTransactionManager`可以为指定的数据源的Hibernate JDBC的`Connection`转成为纯JDBC的访问代码。如果开发者仅访问一个数据库，则开发者完全可以不使用JTA，通过Hibernate和JDBC数据访问进行高级别事务划分。如果开发者已经通过`LocalSessionFactoryBean`的`dataSource`属性与`DataSource`设置了传入的`SessionFactory`，`HibernateTransactionManager`会自动将Hibernate事务公开为JDBC事务。或者，开发者可以通过`HibernateTransactionManager`的`dataSource`属性的配置以确定公开事务的类型。
