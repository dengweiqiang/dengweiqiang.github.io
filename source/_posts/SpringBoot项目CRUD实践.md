---
title: SpringBoot项目CRUD实践
copyright: true
date: 2020-04-14 20:57:50
tags: 学习
---

使用SpringBoot做的一个CRUD的手脚架，并且提供了数据库密码的加密功能。

<!--more-->

### 项目准备

1. 进入“https://start.spring.io/”构建SpringBoot项目；
2. 选择“Web”、“MySQL”、“Mybatis”模块；
3. 导入项目到idea；

## 配置数据库

1. 新建数据库；

   ```shell
   create database test;
   use test;
   ```

2. 新建表

   ```sql
   -- create table `account`
   DROP TABLE `account`;
   CREATE TABLE `account` (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `name` varchar(20) NOT NULL,
     `money` double DEFAULT NULL,
     PRIMARY KEY (`id`)
   ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
   INSERT INTO `account` VALUES ('1', 'aaa', '1000');
   INSERT INTO `account` VALUES ('2', 'bbb', '1000');
   INSERT INTO `account` VALUES ('3', 'ccc', '1000');
   ```

### 配置数据库连接

application.properties文件

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC
# ENC() 是数据库加密函数
spring.datasource.username=ENC(YMoG7LZvcKq3DbW3dc9Ltg==)
spring.datasource.password=ENC(RgqVyviuib6ScIcgcQr9SA==)
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
# jasypt加密的盐值
jasypt.encryptor.password=erp
# 开启debug
debug=true
# 控制台打印SQL
logging.level.top.dengwq.springboot_crud.mapper=debug
```

### Entity

``` java
@Data
@ToString
@EqualsAndHashCode
public class Account {
    private int id ;
    private String name ;
    private double money;
}
```

### Mapper

```java
@Mapper
public interface AccountMapper {
    @Insert("insert into account(name, money) values(#{name}, #{money})")
    int add(@Param("name") String name, @Param("money") double money);

    @Update("update account set name = #{name}, money = #{money} where id = #{id}")
    int update(@Param("name") String name, @Param("money") double money, @Param("id") int  id);

    @Delete("delete from account where id = #{id}")
    int delete(int id);

    @Select("select id, name as name, money as money from account where id = #{id}")
    Account findAccount(@Param("id") int id);

    @Select("select id, name as name, money as money from account")
    List<Account> findAccountList();
}
```

### Service

```java
@Service
public class AccountService {
    @Autowired
    private AccountMapper accountMapper;

    public int add(String name, double money) {
        return accountMapper.add(name, money);
    }
    public int update(String name, double money, int id) {
        return accountMapper.update(name, money, id);
    }
    public int delete(int id) {
        return accountMapper.delete(id);
    }
    public Account findAccount(int id) {
        return accountMapper.findAccount(id);
    }
    public List<Account> findAccountList() {
        return accountMapper.findAccountList();
    }
}
```

### Controller

```java
@RestController
@RequestMapping("/account")
public class AccountController {
    @Autowired
    AccountService accountService;

    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public List<Account> getAccounts() {
        return accountService.findAccountList();
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public Account getAccountById(@PathVariable("id") int id) {
        return accountService.findAccount(id);
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    public String updateAccount(@PathVariable("id") int id, @RequestParam(value = "name", required = true) String name,
                                @RequestParam(value = "money", required = true) double money) {
        int t= accountService.update(name,money,id);
        if(t==1) {
            return "success";
        }else {
            return "fail";
        }

    }

    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    public String delete(@PathVariable(value = "id")int id) {
        int t= accountService.delete(id);
        if(t==1) {
            return "success";
        }else {
            return "fail";
        }

    }

    @RequestMapping(value = "", method = RequestMethod.POST)
    public String postAccount(@RequestParam(value = "name") String name,
                              @RequestParam(value = "money") double money) {

        int t= accountService.add(name,money);
        if(t==1) {
            return "success";
        }else {
            return "fail";
        }

    }
}
```

### 加密组件

1. 依赖

   ```xml
   <dependency>
       <groupId>com.github.ulisesbocchio</groupId>
       <artifactId>jasypt-spring-boot-starter</artifactId>
       <version>2.1.1</version>
   </dependency>
   ```

2. 明文加密

   ```java
   /**
   * ulisesbocchio 加密组件
   */
   @Test
   public void getPass() {
       String name = encryptor.encrypt("root");
       String password = encryptor.encrypt("123456");
       System.out.println("Name加密后明文: " + name);
       System.out.println("Password加密后明文: " + password);
       Assert.assertTrue(name.length() > 0);
       Assert.assertTrue(password.length() > 0);
   }
   ```

3. 使用（application.properties）

   ```java
   # ENC() 是数据库加密函数
   spring.datasource.username=ENC(YMoG7LZvcKq3DbW3dc9Ltg==)
   spring.datasource.password=ENC(RgqVyviuib6ScIcgcQr9SA==)
   ```

   