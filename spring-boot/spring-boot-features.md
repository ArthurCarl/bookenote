# spring-boot-features.md

## SpringApplication

### `FailureAnalyzer` - 启动失败分析;也可以在 `debug` 属性，`DEBUG` log  
```
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### 自定义Banner
1. 在类路径下添加`banner.txt` 文件
2. 设置 `spring.banner.location` 设置banner文件路劲

文件的形式可以是: `banner.gif` `banner.jpg` `banner.png`

Banner 变量
- `${application.version}`
- `${application.formatted-version}`
- `${spring-boot.version}`
- `${spring-boot.formatted-version}`
- `${Ansi.NAME}` 
- `${application.title}`

`spring.main.banner-mode` banner的模式:`System.out` `log` `off`

`SpringBootBanner` 类就是`Banner` 接口的实现；

### 自定义 SpringApplication


--
