# Spring的学习-day02

## 1.applicationContext.xml中的配置

- 对级联属性赋值

  ```xml
  	<!-- 1.对级联属性赋值 -->
  	<bean class="cn.itheima.entity.Grade" id="grade">
  		<property name="id" value="1"></property>
  		<property name="gradeName" value="高三一班"></property>
  	</bean>
  	<bean class="cn.itheima.entity.Student" id="student">
  		<property name="username" value="小明"></property>
  		<property name="sex" value="男"></property>
  		<property name="age" value="20"></property>
  		<property name="grade" ref="grade"></property>
  		<!-- 级联属性赋值 -->
  		<property name="grade.gradeName" value="高三二班"></property>
  	</bean>
  ```

- 基于xml的自动装配

  ```xml
  	<!-- 2.基于xml的自动装配（很少使用，了解就行）
  			注意：自动装配分两种：byName(通过名字)、byType(通过类型)
  			byName:在IOC容器中寻找和属性名一样的对象，寻找到后自动注入
  			byType:在IOC容器中寻找和属性类型一致的对象，寻找到后自动注入，如果匹配到多个就会报错
  	 -->
  	 <!-- byName -->
  	<bean class="cn.itheima.entity.Student" id="student1" autowire="byName">
  		<property name="username" value="小刚"></property>
  		<property name="age" value="18"></property>
  		<property name="sex" value="男"></property>
  	</bean>
  	<!-- byType -->
  	<bean class="cn.itheima.entity.Student" id="student2" autowire="byType">
  		<property name="username" value="小美"></property>
  		<property name="age" value="18"></property>
  		<property name="sex" value="女"></property>
  	</bean>
  ```

- bean之间的关系详解

  ```xml
  <!-- 3.bean之间的关系:继承、依赖
  			注意:（1）如果对象abstract属性被设置为true，则该对象是模板类，不会被实例化
  				（2）利用parent属性实现继承
  				（3）如要要定义模板类，这个时候不用指定class，属性随便设置就行，一定要设置                         abstract属性为true
  				（4）如果要实现依赖可以使用：depends-on，这个时候被依赖的对象会先被创建，且要存                     在
  -->	 
  	 <!-- 3.1继承 -->
  	 <!-- 父类的定义，不用指定class属性，设置abstract属性为true -->
  	 <bean id="parent" abstract="true">
  	 	<property name="username" value="小智"></property>
  	 	<property name="sex" value="男"></property>
  	 	<property name="age" value="20"></property>
  	 </bean>
  	 <!-- 利用parent属性指定继承的父类 -->
  	 <bean class="cn.itheima.entity.Student" id="student3" parent="parent">
  	 	<property name="grade" ref="grade"></property>
  	 </bean>
  	 
  	 <!-- 3.2依赖 ，depends-on属性设置依赖-->
  	 <bean class="cn.itheima.entity.Student" id="student4" depends-on="school">
  	 	<property name="username" value="小智"></property>
  	 	<property name="sex" value="男"></property>
  	 	<property name="age" value="20"></property>
  	 </bean>
  	 <!-- 被依赖的对象 -->
  	 <bean class="cn.itheima.entity.School" id="school"></bean>
  ```

- bean的作用域

  ```xml
   <!-- 4.bean的作用域:scope、singleton,默认为single(单例)
  	 			注意：（1）如果需要修改成单例或者多例，可以设置scope属性
   -->
  	  <!-- 4.1单例
  	  		注意:属性设置为singleton，该对象在初始化IOC容器的时候创建，且每次从IOC容器中获取的                  对象都是一样的
  	  -->
  	<bean class="cn.itheima.entity.Student" id="student5" scope="singleton"></bean>
  	<!-- 4.2多例
  			注意：属性设置为prototype，该对象是在使用的时候创建，并且每次获取的对象都不一样
  	 -->
  	 <bean class="cn.itheima.entity.Student" id="student6" scope="prototype">            </bean>
  	
  ```

- 加载外部属性文件

  ```xml
      <!-- 5.加载外部属性文件 -->
  	<context:property-placeholder location="classpath:db.properties"/>
  	<!-- 调用外部properties中的属性 -->
  	<bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
  		<property name="user" value="${jdbc.user}"></property>
  		<property name="password" value="${jdbc.password}"></property>
  		<property name="jdbcUrl" value="${jdbc.url}"></property>
  		<property name="driverClass" value="${jdbc.driverClass}"></property>
  		<property name="initialPoolSize" value="${jdbc.initialPoolSize}">	               </property>
  		<property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
  		<property name="minPoolSize" value="${jdbc.minPoolSize}"></property>
  	</bean>
  ```

- sqEl表达式语言的使用

  ```xml
  <!-- 6.spEl表达式语言
  			注意:(1)可以利用它引用对象
  				(2)可以调用对象的方法
  				(3)可以使用三元运算符
  				(4)使用规则:#{内容}，除了调用静态方法是#{T(全类名).静态方法}，其他的都是前者
  			
   -->
  	 <bean class="cn.itheima.entity.Example" id="example">
  	 	<!-- 调用静态方法 -->
  	 	<property name="id" value="#{T(cn.itheima.entity.Example).getTotal()}">		   </property>
  	 	<!-- 使用三元运算符 -->
  	 	<property name="flag" value="#{student.sex=='男'?'true':'false'}">                   </property>
  	 	<!-- 引用对象 -->
  	 	<property name="student" value="#{student}"></property>
  	 	<!-- 调用非静态方法 -->
  	 	<property name="value" value="#{example.getNum()}"></property>
  	 </bean>
  ```

- IOC容器中对象的生命周期

  ```xml
   <!-- 7.IOC容器中对象的生命周期
  	 			注意:可以设置init-method给对象初始化调用的方法，这里的初始化方法应该在构造器方                     法之后
  	 				可以设置destory-method给对象设置销毁后调用的方法
  	 				可以设置bean的后置处理器BeanPostProcessor
   -->
  	  <!-- 配置init方法和destory方法 -->
  	  <bean class="cn.itheima.entity.People" id="people" init-method="init"                      destroy-method="destory">
  	  		<property name="username" value="小红"></property>
  	  </bean>
  	  <!-- 配置bean的后置处理器，它是面向所有bean的 -->
  	  <bean class="cn.itheima.beanpostprocess.MyBeanPostProcessor"                              id="myBeanPostProcessor"></bean>
  ```

  ```java
     /**
  	 * spring的第八个案例:bean的生命周期
  	 * 		注意：如果是通过set属性赋值的单例对象的生命周期
  	 * 				（1）默认调用无参构造器
  	 * 				（2）调用set属性赋值
  	 * 				（3）后置处理器的前方法
  	 * 				（4）调用init方法
  	 * 				（5）后置处理器的后方法
  	 * 				（6）容器IOC销毁调用destory方法
  	 */
  ```

  ```java
  /**
   * 自定义后置处理器
   */
  public class MyBeanPostProcessor implements BeanPostProcessor{
  ```

- 通过工厂方法创建bean

  ```xml
   <!-- 8.通过工厂方法创建bean -->
  	  <!-- 通过静态工厂创建bean
  	  		注意：（1）factory-method属性设置调用工厂的哪个方法，只写方法名就行
  	  				，如果有参数，需要使用constructor-arg设置参数
  	  			（2）factory-method:调用哪个方法
  	  			（3）factory-bean:调用哪个工厂bean
  	  			（4）如果只设置factory-method就调用当前class类的对应方法
  	   -->
  	  <bean class="cn.itheima.entity.MyFactory" id="myFactory" factory-                         method="getBean">
  	  	<constructor-arg value="1"></constructor-arg>
  	  </bean>
  	  <!-- 通过实例工厂创建bean -->
  	  <bean class="cn.itheima.entity.MyFactory" id="factory"></bean>
  	  <bean class="cn.itheima.entity.Student" factory-bean="factory" factory-                   method="getBeanByBean" id="studentele">
  	  	<constructor-arg value="1"></constructor-arg>
  	  </bean>
  ```

- 通过spring提供的FactoryBean创建对象

  ```xml
  <!-- 9.通过spring提供的FactoryBean创建对象
  	  	注意：（1）使用的时候需要写一个类实现FactoryBean<T>，T写什么类型就创建什么对象
  -->
  	   <bean class="cn.itheima.entity.SpringFactoryBean" id="springFactoryBean">          </bean>
  ```

  ```java
  /**
   * 通过spring提供的FactoryBean创建对象
   */
  public class SpringFactoryBean implements FactoryBean<Student> {
  	/**
  	 * 该方法返回对象
  	 */
  	@Override
  	public Student getObject() throws Exception {
  		return new Student("小邹",18);
  	}
  	/**
  	 * 该方法返回对象类型
  	 */
  	@Override
  	public Class<Student> getObjectType() {
  		return null;
  	}	
  	/**
  	 * isSingleton方法可以设置创建对象的是否是单例或者多例
  	 */
  	@Override
  	public boolean isSingleton() {
  		//返回true为多例
  		return true;
  	}
  }
  ```

- 配置扫描注解的基本包

  ```xml
        <!-- 配置扫描注解的基本包 -->
  	  <context:component-scan base-package="cn.itheima"/>   
  ```

## 2.注解详解

- 注入对象的注解

  ```java
  /**
   * spring中常见的注解
   * 1.@Repository	一般用于将dao对象注入IOC容器
   * 2.@Service		一般用于将service对象注入IOC容器
   * 3.@Controller	一般用于将controller对象注入IOC容器
   * 4.@Component		 用于将任何对象注入IOC容器中
   * 上面的注入方式：注入的对象id默认为类名首字母小写
   * 如果要指定注入对象的id，可以在注解上添加value属性
   */
  @Repository//将当前对象加入容器中
  ```

- 获取IOC容器中对象的注解

  ```java
  	/**
  	 * Autowired注解详解：
  	 * 	（1）首先从IOC容器中找当前类型的对象，找到后自动注入
  	 * 	（2）如果找到多个就按照用户写的变量找，例子中的即找(userDao)
  	 * 	（3）如果用户需要精确找到某个id的对象，可以添加Qualifier注解
  	 */
  	@Autowired
      @Qualifier(value="xxx")
  ```


