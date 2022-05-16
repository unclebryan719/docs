### Mybatis笔记

#### 开启sql输出

方案1：

- 将ibatis log4j运行级别调到DEBUG可以在控制台打印出ibatis运行的sql语句

- 添加如下语句

  ```bash
  ###显示SQL语句部分
  log4j.logger.com.ibatis=DEBUG
  log4j.logger.com.ibatis.common.jdbc.SimpleDataSource=DEBUG
  log4j.logger.com.ibatis.common.jdbc.ScriptRunner=DEBUG
  log4j.logger.com.ibatis.sqlmap.engine.impl.SqlMapClientDelegate=DEBUG
  log4j.logger.Java.sql.Connection=DEBUG
  log4j.logger.java.sql.Statement=DEBUG
  log4j.logger.java.sql.PreparedStatement=DEBUG　
  ```

方案2：在mybatis.config.xml中增加如下配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD SQL Map Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
 
<configuration>
    <settings>
<setting name="logImpl" value="STDOUT_LOGGING" />
  </settings>
</configuration>
```

在SpringBoot中，修改application.yml文件

```yaml
mybatis:
configuration:
log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

```

