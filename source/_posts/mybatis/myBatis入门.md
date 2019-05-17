---
title: myBatis入门
date: 2018-04-11 20:12:04
tags: [myBatis]
categories: myBatis
---

## MyBatis说明
   - MyBatis是一款优秀的持久层框架，它支持定制化SQL、存储过程以及高级映射。
   MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。
   MyBatis可以使用简单的XML或注解来配置和映射原生信息，
   将接口和Java的POJO(Plain Old Java Objects，普通的Java对象)映射成数据库中的记录。
   
   - MyBatis探究将是一个系列。从这篇文章开始，我将开始探究MyBatis的相关技术，包括：
       + Mybatis使用
       + Mybatis源码分析
       + Mybatis与spring的结合
       + Mybatis与spring boot结合

## MyBatis的使用
   - #### 1 构建SqlSessionFactory
      + Mybatis主要是以SqlSessionFactory为中心，SqlSessionFactory的创建是由SqlSessionFactoryBuilder获取。
   而SqlSessionFactoryBuilder可以从XML配置文件或者一个java configuration实例构建出来SqlSessionFactory
   - #### 2 构建方式
      +  xml构建方式
      ```
         String resource = "mybatis-config.xml";
         InputStream inputStream = Resources.getResourceAsStream(resource);
         SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
         
         XML配置文件中包含了对MyBatis系统的核心设置，包含获取数据库连接实例的数据源(DataSource)和决定事务作用域和控制方式的事务管理器(TransactionManager)
         
         XML文件mybatis-config.xml配置如下：
                  <?xml version="1.0" encoding="UTF-8" ?>
                  <!DOCTYPE configuration
                          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                          "http://mybatis.org/dtd/mybatis-3-config.dtd">
                  <configuration>
                      <!--定义数据库信息，默认使用development数据库构建环境-->
                      <environments default="development">
                          <environment id="development">
                              <!--采用jdbc事务管理-->
                              <transactionManager type="JDBC"/>
                              <!--配置数据库连接信息-->
                              <dataSource type="POOLED">
                                  <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                                  <property name="url" value="jdbc:mysql://127.0.0.1:3306/demo2"/>
                                  <property name="username" value="root"/>
                                  <property name="password" value="rootroot"/>
                              </dataSource>
                          </environment>
                      </environments>
                      <!--定义映射器-->
                      <mappers>
                          <mapper resource="love/wangqi/dao/xml/UserMapper.xml"/>
                      </mappers>
                  </configuration>
      ```
      +  代码构建方式
      ```
         // 构建数据库连接池
         PooledDataSource dataSource = new PooledDataSource();
         dataSource.setDriver("com.mysql.cj.jdbc.Driver");
         dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/demo2");
         dataSource.setUsername("root");
         dataSource.setPassword("rootroot");
         // 构建数据库事务方式
         TransactionFactory transactionFactory = new JdbcTransactionFactory();
         // 创建数据库运行环境
         Environment environment = new Environment("development", transactionFactory, dataSource);
         // 构建Configuration对象
         Configuration configuration = new Configuration(environment);
         // 加入映射器
         configuration.addMappers("love.wangqi.dao");
         sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
      
      ```
      + 总结和对比
        + 可以看出xml和编码本质没有区别，需要注意的是采用这中方式，Mapper类与Mapper的xml映射文件必须放在在同一个package下，
        否则mybatis会找不到xml映射文件，此外该方式不推荐使用，因为会导致配置与编码耦合，不利于维护
   

   - #### 3 表结构与POJO定义
     ```
       表结构：
       
            CREATE TABLE `user` (
              `id` int(10) NOT NULL AUTO_INCREMENT,
              `user_name` varchar(10) DEFAULT NULL,
              `age` int(10) DEFAULT NULL,
              `password` varchar(10) DEFAULT NULL,
              `sex` tinyint(1) NOT NULL DEFAULT '1',
              PRIMARY KEY (`id`)
            )
            
       POJO类：
       
            public class User implements Serializable {
                private Integer id;
                private String userName;
                private Integer age;
                private String password;
                private Integer sex;
            
                public Integer getId() {
                    return id;
                }
            
                public void setId(Integer id) {
                    this.id = id;
                }
            
                public String getUserName() {
                    return userName;
                }
            
                public void setUserName(String userName) {
                    this.userName = userName;
                }
            
                public Integer getAge() {
                    return age;
                }
            
                public void setAge(Integer age) {
                    this.age = age;
                }
            
                public String getPassword() {
                    return password;
                }
            
                public void setPassword(String password) {
                    this.password = password;
                }
            
                public Integer getSex() {
                    return sex;
                }
            
                public void setSex(Integer sex) {
                    this.sex = sex;
                }
            }
            
        映射器UserMapper接口定义：
        
             public interface UserMapper {
                 User selectUser(@Param("pk") Integer pk);
                 void insetUser(User user);
             }
             
        UserMapper的sql映射文件(UserMapper.xml)如下：
        
             <?xml version="1.0" encoding="UTF-8" ?>
             <!DOCTYPE mapper
                     PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                     "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
             <mapper namespace="love.wangqi.dao.UserMapper">
                 <resultMap id="usermap" type="love.wangqi.domain.User">
                     <result column="id" property="id"/>
                     <result column="user_name" property="userName"/>
                     <result column="age" property="age"/>
                     <result column="password" property="password"/>
                     <result column="sex" property="sex"/>
                 </resultMap>
             
                 <cache/>
             
                 <select id="selectUser" resultMap="usermap">
                     select * from user where id = #{id}
                 </select>
             
                 <insert id="insetUser" useGeneratedKeys="true" keyProperty="id">
                     insert into user (user_name, age, password, sex) values (#{userName}, #{age}, #{password}, #{sex})
                 </insert>
             </mapper>
       
     ```
      + 映射器是由java接口和xml文件或者注解共同组成，它的作用如下：
        + 定义参数类型
        + 描述缓存
        + 描述SQL语句
        + 定义查询结果和POJO的映射关系
        
   - #### SqlSession
     - 有了上面的定义，我们再来看看如何使用mybatis来查询数据和插入数据：
     ```
        try (SqlSession session = sqlSessionFactory.openSession()) {
            UserMapper userMapper = session.getMapper(UserMapper.class);
            User user = userMapper.selectUser(pk);
        }
        
        try (SqlSession session = sqlSessionFactory.openSession()) {
            UserMapper userMapper = session.getMapper(UserMapper.class);
            userMapper.insetUser(user);
            session.commit();
        }
     
     ```
     - 可以看到，在定义好mapper和sql映射文件之后，mybatis的使用就非常便捷：
       + 首先通过SqlSessionFactory获得SqlSession的实例。
       + 通过SqlSession获得映射器实例(Mapper)
       + 执行映射器实例中的自定义方法
       + SqlSession接口类似于一个JDBC的Connection接口对象，我们需要保证每次用完正常关闭它
       + 数据库事务MyBatis是交由SqlSession去控制的，我们可以通过SqlSession提交(commit)或者回滚(rollback)
     
     - MyBatis组件思维导图及说明
       ![mybatis](/myBatis.png "Optional title")

[//]: 此编辑中，图片不显示，因为hexo默认将所有md文件放在_posts下，但该项目，进行了分类
[//]: 因为在生成中，已经统一将图片放在mybatis-myBatis使用目录下，结果已发布网页为准
