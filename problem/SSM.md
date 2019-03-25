# **SSM框架**
## **1、事务配置失效**
applicationContext.xml配置如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       "
        >
      <!--扫描-->
       <context:component-scan base-package="com.unis"/>
       <!--开启注解-->
       <context:annotation-config />
       <!-- 数据访问层配置 -->
       <import resource="classpath:/spring-comm-dao.xml" />
    <!--init-->
    <bean class="com.unis.service.InitListener"/>


</beans>
```
项目结构如下：


![项目结构](../pic/pic1.png "包结构")

spring-comm-dao.xml配置如下：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans" default-lazy-init="true"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:property-placeholder location= "classpath:mysql-jdbc.properties" />
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close"
          p:driverClassName="${jdbc.driverClassName}"
          p:url="${jdbc.url}"
          p:username="${jdbc.username}"
          p:password="${jdbc.password}"
          p:maxActive="${jdbc.maxActive}"
          p:maxWait="${jdbc.maxWait}"
          p:maxIdle="${jdbc.maxIdle}"
          p:validationQuery="select 1"
          p:testOnBorrow="true"
          p:defaultAutoCommit="true"/>


    <!-- 配置Jdbc模板  -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate"
          p:dataSource-ref="dataSource" />

    <!-- myBatis文件 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis-mapper-config.xml"/>

    </bean>
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 非常重要。。。
       在mybatis 中扫描,并且注解 Repository 在代码中才能使用 @Autowired Mapper 接口,在SPRING 中扫描没用-->
        <property name="basePackage" value="com.unis.dao.mybatis"/>
        <property name="annotationClass" value="org.springframework.stereotype.Repository"/>

    </bean>
    <!-- 使用SESSION 直接操作 -->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory" />
    </bean>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
          p:dataSource-ref="dataSource" />

    <!--两种事务配置 下面第一种是需要注解,第2种是全部,但必须抛出RuntimeException 才会回滚-->

        <!--启用事务-->
    <!--<tx:annotation-driven transaction-manager="transactionManager"/>-->

    <!-- 通过AOP配置提供事务增强，让service包下所有Bean的所有方法拥有事务 -->
    <aop:config proxy-target-class="true">
        <aop:pointcut id="serviceMethod"
                      expression=" execution(* com.unis.service..*(..))" />
        <aop:advisor pointcut-ref="serviceMethod" advice-ref="txAdvice" />
    </aop:config>
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" />
        </tx:attributes>
    </tx:advice>
</beans>
```
发现事务配置失效，service中并没有回滚。问题在于applicationContext.xml中
` <context:component-scan base-package="com.unis"/>`配置路径太广了，直接定位到com.unis.service即可解决。过大的扫描路径会导致后续扫描包配置覆盖前面扫描包配置的情况，参见：[扫描所有包导致事务失效](https://tianqing-525.iteye.com/blog/1767353).

---



























```
