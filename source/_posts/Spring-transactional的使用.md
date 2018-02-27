---
title: Spring @transactional的使用
date: 2018-02-27 15:22:01
tags: Spring
---

# @Transactional 注解管理事务的实现步骤
使用@Transactional 注解管理事务的实现步骤分为两步。第一步，在 xml 配置文件中添加如清单 1 的事务配置信息。除了用配置文件的方式，@EnableTransactionManagement 注解也可以启用事务管理功能。这里以简单的 DataSourceTransactionManager 为例。

```xml
<tx:annotation-driven />
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```
将@Transactional 注解添加到合适的方法上，并设置合适的属性信息。@Transactional 注解的属性信息如表 1 展示。

|属性名|	说明|
|:--|:--|
|name|	当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。|
|propagation|	事务的传播行为，默认值为 REQUIRED。|
|isolation|	事务的隔离度，默认值采用 DEFAULT。|
|timeout|	事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。|
|read-only|	指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。|
|rollback-for|	用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。|
|no-rollback-for|	抛出 no-rollback-for 指定的异常类型，不回滚事务。|

除此以外，@Transactional 注解也可以添加到类级别上。当把@Transactional 注解放在类级别时，表示所有该类的公共方法都配置相同的事务属性信息。见清单 2，EmployeeService 的所有方法都支持事务并且是只读。当类级别配置了@Transactional，方法级别也配置了@Transactional，应用程序会以方法级别的事务属性信息来管理事务，换言之，方法级别的事务属性信息会覆盖类级别的相关配置信息。

# Spring 的注解方式的事务实现机制
在应用系统调用声明@Transactional 的目标方法时，Spring Framework 默认使用 AOP 代理，在代码运行时生成一个代理对象，根据@Transactional 的属性配置信息，这个代理对象决定该声明@Transactional 的目标方法是否由拦截器 TransactionInterceptor 来使用拦截，在 TransactionInterceptor 拦截时，会在在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器AbstractPlatformTransactionManager 操作数据源 DataSource 提交或回滚事务。

Spring AOP 代理有 CglibAopProxy 和 JdkDynamicAopProxy 两种，以 CglibAopProxy 为例，对于 CglibAopProxy，需要调用其内部类的 DynamicAdvisedInterceptor 的 intercept 方法。对于 JdkDynamicAopProxy，需要调用其 invoke 方法。

正如上文提到的，事务管理的框架是由抽象事务管理器 AbstractPlatformTransactionManager 来提供的，而具体的底层事务处理实现，由 PlatformTransactionManager 的具体实现类来实现，如事务管理器 DataSourceTransactionManager。不同的事务管理器管理不同的数据资源 DataSource，比如 DataSourceTransactionManager 管理 JDBC 的 Connection。

# 注解方式的事务使用注意事项
当您对 Spring 的基于注解方式的实现步骤和事务内在实现机制有较好的理解之后，就会更好的使用注解方式的事务管理，避免当系统抛出异常，数据不能回滚的问题。

## 正确的设置@Transactional 的 propagation 属性
需要注意下面三种 propagation 可以不启动事务。本来期望目标方法进行事务管理，但若是错误的配置这三种 propagation，事务将不会发生回滚。

1. TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
2. TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
3. TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

## 正确的设置@Transactional 的 rollbackFor 属性
默认情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常）或者 Error，则 Spring 将回滚事务；除此之外，Spring 不会回滚事务。

如果在事务中抛出其他类型的异常，并期望 Spring 能够回滚事务，可以指定 rollbackFor。例：

@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class)
若在目标方法中抛出的异常是 rollbackFor 指定的异常的子类，事务同样会回滚。

## @Transactional 只能应用到 public 方法才有效
只有@Transactional 注解应用到 public 方法，才能进行事务管理。这是因为在使用 Spring AOP 代理时，Spring 在调用在图 1 中的 TransactionInterceptor 在目标方法执行前后进行拦截之前，DynamicAdvisedInterceptor（CglibAopProxy 的内部类）的的 intercept 方法或 JdkDynamicAopProxy 的 invoke 方法会间接调用 AbstractFallbackTransactionAttributeSource（Spring 通过这个类获取表 1. @Transactional 注解的事务属性配置属性信息）的 computeTransactionAttribute 方法。

这个方法会检查目标方法的修饰符是不是 public，若不是 public，就不会获取@Transactional 的属性配置信息，最终会造成不会用 TransactionInterceptor 来拦截该目标方法进行事务管理。

## 避免 Spring 的 AOP 的自调用问题
在 Spring 的 AOP 代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理，这会造成自调用问题。若同一类中的其他没有@Transactional 注解的方法内部调用有@Transactional 注解的方法，有@Transactional 注解的方法的事务被忽略，不会发生回滚。
```java
@Service
public class OrderService {
    private void insert() {
        insertOrder();
    }
    
    @Transactional
    public void insertOrder() {
        //insert log info
        //insertOrder
        //updateAccount
    }
}
```

@Transactional 注解只应用到 public 方法和自调用问题，是由于使用 Spring AOP 代理造成的。为解决这两个问题，使用 AspectJ 取代 Spring AOP 代理

需要将下面的 AspectJ 信息添加到 xml 配置信息中。
```xml
<tx:annotation-driven mode="aspectj" />
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    </bean class="org.springframework.transaction.aspectj.AnnotationTransactionAspect" factory-method="aspectOf">
        <property name="transactionManager" ref="transactionManager" />
    </bean>
</tx:annotation-driven>
```
同时在 Maven 的 pom 文件中加入 spring-aspects 和 aspectjrt 的 dependency 以及 aspectj-maven-plugin。
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.8.9</version>
</dependency>
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.9</version>
    <configuration>
        <showWeaveInfo>true</showWeaveInfo>
        <aspectLibraries>
            <aspectLibrary>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
            </aspectLibrary>
        </aspectLibraries>
    </configuration>
    <executions>
        <execution>
            <goals>
            <goal>compile</goal>
            <goal>test-compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
