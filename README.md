# mybatis-enhance-actable-0.0.1

该项目是从之前写过的一个框架中抽取出来的，本身是对Mybatis做的增强功能，为了能够使习惯了hibernate框架的开发者能够快速的入手Mybatis，我给他取名叫做 “A.C.Table” 本意是自动建表的意思，A.C.Table是一个基于Spring和Mybatis的Maven项目，增强了Mybatis的功能，过配置model注解的方式来创建表，修改表结构，并且实现了共通的CUDR功能提升开发效率，目前仅支持Mysql，后续可能会扩展针对其他数据库的支持。

A.C.Table是采用了Spring、Mybatis技术的Maven结构，详细介绍如下：

 **######### mybatis增加功能自动创建表——A.C.Table版本说明################** 
1. 该版本修复了修改主键同时修改其类型引起的error
2. 该版本修复了根据model创建时没有创建父类中的字段的问题（ps：目前只支持扫描一层继承）
3. 该笨笨增加了对唯一约束的支持
4. 从原有的框架中剥离出来，支持任意结构的spring+mybatis的框架使用
5. 再次声明A.C.Table目前仅支持mysql数据库

 **使用步骤方法** 
1. 使用该功能的项目需要依赖mybatis-enhance-actable-0.0.1-SNAPSHOT.jar
2. 在你的web项目上创建个目录比如config下面创建个文件autoCreateTable.properties文件的内容如下：

	mybatis.table.auto=update
	mybatis.model.pack=com.sunchenbin.store.model
	
	本系统提供三种模式：

	2.1 当mybatis.table.auto=create时，系统启动后，会将所有的表删除掉，然后根据model中配置的结构重新建表，该操作会破坏原有数据。

	2.2 当mybatis.table.auto=update时，系统会自动判断哪些表是新建的，哪些字段要修改类型等，哪些字段要删除，哪些字段要新增，该操作不会破坏原有数据。
	
	2.3 当mybatis.table.auto=none时，系统不做任何处理。

	2.4mybatis.model.pack这个配置是用来配置要扫描的用于创建表的对象的包名
	
3. spring的配置文件中需要做如下配置：
	<!-- 自动扫描(自动注入mybatis-enhance-actable的Manager)必须要配置，否则扫描不到底层的mananger方法 -->
	<context:component-scan base-package="com.mybatis.enhance.store.manager.*" />
	
	<!-- 这是mybatis-enhance-actable的功能开关配置文件,其实就是将上面第2点说的autoCreateTable.properties文件注册到spring中，以便底层的mybatis-enhance-actable的方法能够获取到-->
	<bean id="configProperties" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
        <property name="locations">
            <list>
                <value>classpath*:config/autoCreateTable.properties</value>
            </list>
        </property>
    </bean>
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
        <property name="properties" ref="configProperties" />
    </bean>
	
	<!-- mybatis的配置文件中需要做两项配置，因为mybatis-enhance-actable项目底层是直接依赖mybatis的规范执行sql的，因此需要将其中的mapping和dao映射到一起 -->
	1. classpath*:com/mybatis/enhance/store/mapping/*/*.xml
	2. com.mybatis.enhance.store.dao.*
	
	举例这两个配置配置的详细位置
	
	<!-- myBatis文件 -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<!-- 自动扫描entity目录, 省掉Configuration.xml里的手工配置 -->
		<property name="mapperLocations">
			<array>
              <value>classpath*:com/sunchenbin/store/mapping/*/*.xml</value>
              <value>classpath*:com/mybatis/enhance/store/mapping/*/*.xml</value>
          	</array>
		</property>
		<property name="typeAliasesPackage" value="com.sunchenbin.store.model.*" />
		<!-- <property name="configLocation" value="classpath:core/mybatis-configuration.xml" /> -->
	</bean>
	
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.sunchenbin.store.dao.*;com.mybatis.enhance.store.dao.*" />
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
	</bean>
	
**代码用途讲解** 

    1.SysMysqlColumns.java这个对象里面配置的是mysql的数据类型，这里配置的类型越多，意味着创建表时能使用的类型越多

    2.LengthCount.java是一个自定义的注解，用于标记在SysMysqlColumns.java里面配置的数据类型上的，标记该类型需要设置几个长度，如datetime/varchar(1)/decimal(5,2)，分别是需要设置0个1个2个

    3.Column.java也是一个自定义的注解，用于标记model中的字段上，作为创建表的依据如不标记，不会被扫描到，有几个属性用来设置字段名、字段类型、长度等属性的设置，详细请看代码上的注释

    4.Table.java也是一个自定义的注解，用于标记在model对象上，有一个属性name，用于设置该model生成表后的表名，如不设置该注解，则该model不会被扫描到

    5.系统启动后会去自动调用SysMysqlCreateTableManagerImpl.java的createMysqlTable()方法，没错，这就是核心方法了，负责创建、删除、修改表。

 **共通的CUDR功能使用** 	
    1.使用方法很简单，大家在manager或者Controller中直接注入BaseMysqlCRUDManager这个接口然后给一个泛型就可以了
    2.注意：接口的泛型就是你要操作的表对应的model

 **demo代码的地址** 
    
    1.码云地址：http://git.oschina.net/sunchenbin/mybatis-enhance-actable-demo
    
    2.代码下载地址：https://git.oschina.net/sunchenbin/mybatis-enhance-actable-demo.git

 **之前的旧项目地址** 

    http://git.oschina.net/sunchenbin/Mybatis_BuildTable_V0.2
