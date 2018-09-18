# Mybatis的学习-day02

## 1.输入参数和输出参数详解

- 传入简单类型

  ```xml
  <!--根据id查找用户 -->
      <select id="selectUserById" parameterType="Integer" 
      resultType="cn.xiaozou.entity.User">
              select * from user where id=#{id}
  </select>
  ```

- 传入pojo对象

  ```xml
  	<!-- 增加用户 -->
  	<insert id="insertUser" parameterType="cn.xiaozou.entity.User">
  		<!-- 为了执行增加用户操作后（还没提交事务时），就获取当前 用户的id -->
  		<selectKey keyProperty="id" order="AFTER" keyColumn="id"
  			resultType="Integer">
  			select LAST_INSERT_ID();
  		</selectKey>
  		insert into user(id,username,sex,birthday,address)
  		values(#{id},#{username},#{sex},#{birthday},#{address})
  	</insert>
  ```

- 传入pojo包装对象

  ```xml
  <!-- 根据id数组查找对象 -->
  	<!-- 
  		foreach标签详解
  			1.collection:代表要遍历的集合，如果传入的对象是数组，
  				应该写array，如果是List写list，如果传入的是包装对象就可以写包装对象的属性
  			2.open:前缀
  			3.close:后缀
  			4.separator:分隔符
  			5.item:遍历的元素，可以随便写，这里写了之后，后面就要写对应的
  	 -->
  	<select id="selectByIds" parameterType="QueryVo" resultType="User">
  		select * from user
  		where id in
  		<foreach collection="ids" open="(" close=")" separator="," item="id">
  			#{id}
  		</foreach>
  	</select>
  ```

- 输出简单类型

  ```xml
  	<!-- 输出简单类型 -->
  	<select id="countNum" resultType="Integer">
  		select count(*) from user
  	</select>
  ```

- 输出pojo对象

  ```xml
  <select id="selectUserById" parameterType="Integer"
  		resultType="cn.xiaozou.entity.User">
  		select * from user where id=#{id}
  </select>
  ```

## 2.ResultMap详解

- resultMap:当数据库中表的字段和实体类的属性不一致时使用，还有进行一对多，或者多对一映射时使用

  ```xml
  <!-- 
  		resultMap的应用场景:
  		1.当数据中对应表的字段和对应实体的类属性不一致时使用
  		2.如果是单表，这个时候我们只需要将没有对应起来的属性字段匹配起来就行
  		3.type：映射到的类型，可以写别名
  		4.id:随便起的名字，一定要和下面resultMap的值一样，即名字
  	 -->
  	<resultMap type="orders" id="OrdersMapper">
  		<!-- 只有这一项数据不对应，所以只需要写这一个 -->
  		<result column="user_id" property="userId"/>
  	</resultMap>
  	<select id="selectById" parameterType="Integer" resultMap="OrdersMapper">
  		select * from orders where id=#{id}
  	</select>
  ```


## 3.动态SQL

- If标签：可以进行条件判断，set标签去除后逗号

  ```xml
  <!-- 
  		1.if标签:可以进行条件判断 
  		2.set标签：可以去除后逗号 
  	-->
  	<update id="updateUserBySelective" parameterType="User">
  		update user
  		<set>
  			<if test="username!=null">
  				username=#{username},
  			</if>
  			<if test="sex!=null">
  				sex=#{sex},
  			</if>
  			<if test="birthday!=null">
  				birthday=#{birthday},
  			</if>
  			<if test="address!=null">
  				address=#{address},
  			</if>
  		</set>
  		where id=#{id}
  	</update>
  ```

- foreach标签：遍历传入的集合或者数组

  ```xml
  <!-- foreach标签详解 
  			1.collection:代表要遍历的集合，如果传入的对象是数组，应该写array，如果是List写	     			list,如果是包装对象就可以写包装对象的属性 
  			2.open:前缀 
  			3.close:后缀 
  			4.separator:分隔符 
  			5.item:遍历的元素，可以随便写，这里写了之后，后面就要写对应的 
  -->
  	<select id="selectByIds" parameterType="QueryVo" resultType="User">
  		select * from user
  		where id in
  		<foreach collection="ids" open="(" close=")" separator=","
  			item="id">
  			#{id}
  		</foreach>
  	</select>
  ```

- where标签：去除前and

  ```xml
  	<!-- 
  		where标签：有where的作用，并且可以去除前and
  	 -->
  	<select id="selectByExample" parameterType="User" resultType="User">
  		select * from user 
  			<where>
  				<if test="id!=null">
  					and id=#{id}
  				</if>
  				<if test="username!=null">
  					and username like "%"#{username}"%"
  				</if>
  			</where>
  	</select>
  ```

- sql片段标签：可以将重复的sql提取出来，使用时用include即可，达到代码复用的效果

  ```xml
  	<sql id="selectAll">
  	 	select * from user
  	 </sql>
  	<select id="selectByExample" parameterType="User" resultType="User">
  		<include refid="selectAll"></include>
  			<where>
  				<if test="id!=null">
  					and id=#{id}
  				</if>
  				<if test="username!=null">
  					and username like "%"#{username}"%"
  				</if>
  			</where>
  	</select>
  ```

  ## 4.关联查询

  ### 4.1一对一查询

  ```xml
  <!-- 
  		配置一对一关系
  		1.不管对象属性和数据库属性有没有对应，如果需要获取值就要写映射
  		2.association的property属性的值为当前对象在对象中的名字，javaType：当前对象的类型
  		3.property:实体属性，column:数据库字段
  		4.id属性一定要用id标签
  	 -->
  	<resultMap type="Orders" id="SingleToSingle">
  		<id property="id" column="id"/>
  		<result property="number" column="number"/>
  		<result property="createtime" column="createtime"/>
  		
  		<association property="user" javaType="User">
  			<id property="id" column="user_id"/>
  			<result property="username" column="username"/>
  		</association>
  		
  	</resultMap>
  	<select id="selectSingelToSingle" resultMap="SingleToSingle">
  			select 
  				o.id,o.number,o.createtime,o.user_id,
  	  			u.username
  			from orders o
  			LEFT JOIN	user u
  			on o.user_id=u.id;
  	</select>
  ```

  ### 4.2一对多查询

  ```xml
  
  	<!-- 
  		配置一对多关系
  			1.id字段一定要用id标签
  			2.column指数据库中的字段，property指实体的属性
  			3.配置一对多关系，需要标签collection
  	 -->
  	<resultMap type="User" id="oneToManyMapper">
  		<id column="id" property="id"/>
  		<result column="username" property="username"/>
  		
  		<collection property="orders" javaType="Orders">
  			<id column="id" property="id"></id>
  			<result column="number" property="number"/>
  			<result column="createtime" property="createtime"/>
  		</collection>
  	</resultMap>
  	
  	<select id="selectOneToMany" resultMap="oneToManyMapper">
  		select 
  				u.id,u.username,
  		o.number,o.createtime,o.id
  			from user u
  			LEFT JOIN	orders o
  			on u.id=o.user_id;
  	</select>
  ```
