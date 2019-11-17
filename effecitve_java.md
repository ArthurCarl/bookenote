# Effective Java Third Edition

## 对象创建和销毁
### 静态工厂/构造器
1. 名称
2. 单例对象
3. 返回子类对象
4. 入参不同返回不同对象
5. 客户端服务端代码解耦

缺点:
1. 没有`public` `protected` 构造器则不能有子类
2. 难以发现

### builder -构造器有很多参数

### 私有构造器/枚举类型 加强单例的属性

### 私有构造器加强工具类的非实例化

### 依赖注入-不确定资源

### 避免创建不必要的对象
`Map.keySet()`  
原型优先

### 消除不必要的对象引用
1. 内存泄露-类自己管理内存时
2. 缓存
3. 监听器/其他回调