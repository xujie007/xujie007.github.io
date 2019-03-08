---
layout:     post
title:      PGSQL使用JSON格式存储
subtitle:   完结
date:       2018-12-02
author:     xujie
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - PGSQL
---

> 正所谓前人栽树，后人乘凉。
> 感谢[Huxpro](https://github.com/huxpro)提供的博客模板,同时也感谢BY!
> 
> [我的的博客](http://my.happy-coding.cn)

# 前言
> 我这里使用的架构是SSM架构，所以这里主要讲mybatis里面怎么只用JSON格式

# 创建JSONTypeHandlerPg工具类,继承BaseTypeHandler<Object>

```
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.postgresql.util.PGobject;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JSONTypeHandlerPg extends BaseTypeHandler<Object> {

    private static final PGobject jsonObject = new PGobject();

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
        jsonObject.setType("json");
        jsonObject.setValue(parameter.toString());
        ps.setObject(i, jsonObject);
    }

    @Override
    public Object getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return rs.getString(columnIndex);
    }

    @Override
    public Object getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return cs.getString(columnIndex);
    }

    @Override
    public Object getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return rs.getString(columnName);
    }

}
```

# 配置XML文件，指定字段为JSON格式
```
    <result column="j1" property="j1" jdbcType="OTHER" javaType="Object" typeHandler="cn.lexiaotongvip.www.utils.JSONTypeHandlerPg" />
    <result column="j2" property="j2" jdbcType="OTHER" javaType="Object" typeHandler="cn.lexiaotongvip.www.utils.JSONTypeHandlerPg"/>
    <result column="j3" property="j3" jdbcType="OTHER" javaType="Object" typeHandler="cn.lexiaotongvip.www.utils.JSONTypeHandlerPg"/>
```

### 到这里教程基本结束，使用方式和普通的类型也是一样。


