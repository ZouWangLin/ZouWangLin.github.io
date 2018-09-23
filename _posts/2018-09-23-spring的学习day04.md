# Spring的学习-day03

## 1.applicationContext.xml中的配置

- 基于注解的连接池

  ```properties
  jdbc.user=root
  jdbc.password=root
  jdbc.jdbcUrl=jdbc:mysql://localhost:3306/online-test
  jdbc.driverClass=com.mysql.jdbc.Driver
  jdbc.maxPoolSize=10
  jdbc.minPoolSize=5
  jdbc.initialPoolSize=5
  ```

  ```xml
  <!-- 扫描外部文件 -->
  <context:property-placeholder location="classpath:db.properties"/>
  
  <!-- 开启扫描注解 -->
  <context:component-scan base-package="cn.itheima"/>
  
  <!-- 配置数据源（即连接池） -->
  <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
  	<property name="user" value="${jdbc.user}"></property>
  	<property name="password" value="${jdbc.password}"></property>
  	<property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
  	<property name="driverClass" value="${jdbc.driverClass}"></property>
  	<property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
  	<property name="minPoolSize" value="${jdbc.minPoolSize}"></property>
  	<property name="initialPoolSize" value="${jdbc.initialPoolSize}"></property>
  </bean>
  ```

- 基于注解的事务管理

  ```xml
   <!-- 1.配置事务管理器  -->
  <bean class="org.springframework.jdbc.datasource.init.DataSourceInitializer" id="transactionManager">
  	<property name="dataSource" ref="dataSource"></property>
  </bean>
   
   <!-- 2.开启注解事务支持，默认引用的transaction-manager是transactionManager -->
   <tx:annotation-driven transaction-manager="transactionManager"/>
   
   <!-- 注解完成事务详解：@Transaction注解 ，该注解可用加在方法上，也可以加在类上
   		@Transaction的详解:
   			(1)propagation属性:用来设置事务的传播行为
   			   事务的传播行为:一个方法运行在一个开启了事务的方法中时，当前方法是使用原有的事务
   			   还是开启新的事务？这就叫事务的传播行为
   			 propagation的默认值为:Propagation.REQUIRED(默认使用现有的事务)
   			 propagation的值:Propagation.REQUIRED_NEW(不使用现有的事务，而开启自身的事务)
   			(2)isolation:用来设置事务的隔离级别
   			   -REPEATABLE_READ:可重复读，MySql默认
   			   -READ_COMMITTED:读已提交，Oracle默认
   			   总结：一般使用READ_COMMITTED
   			(3)rollbackFor:遇到什么异常回滚，值是异常的类型
   			   rollbackForClassName:遇到什么异常回滚，值是异常的名字
   			   noRollbackFor:设置不回滚，值是异常的类型
   			   noRollbackForClassName:设置不回滚，值是异常的名字
   			(4)readonly:设置只读属性，通常情况下如果是查询的操作，该属性
   			设置readonly为true，如果不是可用设置readonly为false   
   			(5)timeout:用来设置超时的时间，单位为秒
    -->
  ```

- 基于xml的事务管理

  ```xml
  <!-- 3.配置事务管理类 -->
  <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
  	<property name="dataSource" ref="dataSource"></property>
  </bean>
  
  <!-- 4.配置事务通知 -->
  <tx:advice transaction-manager="transactionManager" id="txAdvice">
  	<tx:attributes>
  		<!-- 配置方法对应的隔离级别以及事务的传播行为 -->
  		<tx:method name="save" isolation="READ_COMMITTED" propagation="REQUIRED"/>
  	</tx:attributes>
  </tx:advice>
  
  <!-- 5.配置切入点 -->
  <aop:config>
  	<aop:pointcut expression="execution(* cn.itheima.transaction.*.*(..))"                  id="pointcut"/>
  	<aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
  </aop:config>
  ```
