# Mybatis

`mybatis`中的 jdbcType, javaType, jdbcTypeHandler:
- jdbcType 数据库中的类型
- javaType Java对象属性类型
- jdbcTypeHandler 以上2种关系的映射处理

**最佳实践**:不要讲数据库中的列全部映射为java 的 String，特别是数字，选择合适的java类型

自定义 jdbcTypeHandler 的使用:
1. 实现 `TypeHandler` 接口,或者继承 `BaseTypeHandler` 抽象类
2. 注册TypeHandler    
  - `@MappedJdbcTypes`注解
  - `xml` 的 `<typeHandler>`
3. 使用

