---
title: "PostgreSql 快速入门"
description: "Springboot 结合 MybatisPlus 快速入门 PostgreSql。"
tags: ["PostgreSql","Quick Start"]
categories: ["PostgreSql"]
date: 2024-12-24T16:29:18+08:00
image: "postgresql-quick-start-cover.png"
---

## 数据类型

### 数值型

|   名称   | 存储长度 |         描述         |                 范围                  |
| :------: | :------: | :------------------: | :-----------------------------------: |
| smallint |  2字节   |      小范围整数      | -2<sup>16 </sup>到 2<sup>16</sup> - 1 |
| integer  |  4字节   |      常用的整数      | -2<sup>32 </sup>到 2<sup>32</sup> - 1 |
|  bigint  |  8字节   |      大范围整数      | -2<sup>64 </sup>到 2<sup>64</sup> - 1 |
| decimal  |  可变长  | 用户指定的精度，准确 | 小数点前 131072 位；小数点后 16383 位 |
| numeric  |  可变长  | 用户指定的精度，准确 | 小数点前 131072 位；小数点后 16383 位 |
|   real   |  4字节   |   可变精度，不准确   |          6 位十进制数字精度           |
|  double  |  8字节   |   可变精度，不准确   |          15 位十进制数字精度          |

### 字符串类型

|    名称    | 存储长度 |         描述         |  示例   |
| :--------: | :------: | :------------------: | :-----: |
|    char    |  1字节   |     固定长度字符     | 'A','z' |
| varchar(n) |  可变长  | 定长字符，最大长度n  | 'hello' |
|    text    |  可变长  | 不限长度的字符串类型 | 'hello' |

### 时间类型

|    名称     | 存储长度 |         描述         |            示例             |
| :---------: | :------: | :------------------: | :-------------------------: |
|    data     |  4字节   |         日期         |        ‘2024-01-01'         |
|    time     |  8字节   |         时间         |          '14:30:00          |
|  timestamp  |  8字节   |       日期时间       |    '2024-01-01 14:30:00'    |
| timestamptz |  8字节   | 带时区的日期时间组合 | '2024-01-01 14:30:00+08:00' |
|  interval   |  16字节  | 时间间隔，年/月/日等 |  '1 year 2 months 3 days'   |

### json 类型

| 名称  | 存储长度 |       描述        |            示例             |
| :---: | :------: | :---------------: | :-------------------------: |
| json  |  可变长  |   JSON 数据存储   | {"name": "John", "age": 30} |
| jsonb |  可变长  | 二进制格式的 JSON | {"name": "John", "age": 30} |

### 其他数据类型

| 名称  | 存储长度 |         描述         |        示例        |
| :---: | :------: | :------------------: | :----------------: |
| bool  |  1字节   |        布尔值        |    true, false     |
| enum  |  可变长  | 用户自定义的枚举类型 |   'red', 'blue'    |
| bytea |  可变长  |    存储二进制数据    |    '\xDEADBEEF'    |
| array |  可变长  |     数组存储类型     |    '{1, 2, 3}'     |
| uuid  |  16字节  |         uuid         | 550e8400-e29b-41d4 |
| cidr  |  8字节   |  IPv4 或 IPv6 网络   |  '192.168.0.0/24'  |
| inet  |  8字节   |       IP 地址        |   '192.168.1.1'    |

## 实践使用

建表 sql：

```postgresql
CREATE TABLE "users"
(
    user_id       SERIAL PRIMARY KEY,                  -- 用户ID，自动递增
    user_name     VARCHAR(50)         NOT NULL,        -- 用户名
    email         VARCHAR(100) UNIQUE NOT NULL,        -- 用户的邮箱，唯一且不能为空
    password      VARCHAR(255)        NOT NULL,        -- 用户的密码
    date_of_birth DATE,                                -- 用户的出生日期
    phone_number  VARCHAR(20),                         -- 用户的电话号码
    address       TEXT,                                -- 用户的地址
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- 用户创建时间
    updated_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP  -- 用户更新时间
);

-- 创建触发器函数
CREATE OR REPLACE FUNCTION update_updated_at()
    RETURNS TRIGGER AS
$$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP; -- 更新为当前时间
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 创建触发器
CREATE TRIGGER set_updated_at
    BEFORE UPDATE
    ON "user"
    FOR EACH ROW
EXECUTE FUNCTION update_updated_at();
```

UserMapper.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mincai.study.postgresql.mapper.UserMapper">

    <resultMap id="BaseResultMap" type="com.mincai.study.postgresql.domain.User">
            <id property="userId" column="user_id" jdbcType="INTEGER"/>
            <result property="userName" column="user_name" jdbcType="VARCHAR"/>
            <result property="email" column="email" jdbcType="VARCHAR"/>
            <result property="password" column="password" jdbcType="VARCHAR"/>
            <result property="dateOfBirth" column="date_of_birth" jdbcType="DATE"/>
            <result property="phoneNumber" column="phone_number" jdbcType="VARCHAR"/>
            <result property="address" column="address" jdbcType="VARCHAR"/>
            <result property="createdAt" column="created_at" jdbcType="TIMESTAMP"/>
            <result property="updatedAt" column="updated_at" jdbcType="TIMESTAMP"/>
    </resultMap>

    <sql id="Base_Column_List">
        user_id,user_name,email,
        password,date_of_birth,phone_number,
        address,created_at,updated_at
    </sql>
</mapper>
```

测试代码

```java
@SpringBootTest
class PostgresqlApplicationTests {

    @Resource
    UserService userService;

    @Test
    void testUserService() {
        List<User> userList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setUserName("用户" + i);
            user.setEmail(i + "@qq.com");
            user.setPassword(UUID.randomUUID().toString());
            user.setDateOfBirth(DateTime.now());
            user.setPhoneNumber(String.valueOf(i));
            user.setAddress("111");
            userList.add(user);
        }

        // 增
        userService.saveBatch(userList);

        // 删除
        userService.removeById(10);

        // 改
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.ge(User::getPhoneNumber, "3");
        User user = new User();
        user.setUserName("new id");
        userService.update(user, queryWrapper);
        userService.update(user, queryWrapper);

        // 查
        List<User> users = userService.list(queryWrapper);
        System.out.println(users);
    }
}
```

