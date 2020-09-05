# Angular2-book

## 路由

### 内嵌路由

``` javaScript
const routes:Routes=[
  {path:'',redirectTo:'home',pathMatch:'full'},
  {path:'home',component: HomeComponent},
  {path:'products',component:ProductsComponent,children:childRoutes}//
];

const routes:Routes =[
  {path:'',redirectTo:'main'},
  {path:'main',component:MainComponent},
  {path:':id',component:ByIdComponent},
];
```



## Dependency Injection

`@NgModule` 中`providers`的类可以在整个应用中被注入到任何一个组件中。

### 注入的方式

- 单例注入
- 函数注入，调用函数将函数的返回值注入(对象工厂),最强大的依赖注入
- 值注入
- 别名注入

#### 对象工厂

```javaScript
{
  provide : MyComponent,
  useFactory:()=>{
    if(loggedIn){
      return new MyLoggedComponent();
    }
    return new MyComponent();
  }
}


{
  provide : MyComponent,
  useFactory:(user)=>{
    if(user.loggedIn()){
      return new MyLoggedComponent(user);
    }
    return new MyComponent();
  },
  deps: [User]
}

```

运行时改变注入对象：
```javaScript
useInjectors():void{
  let injector: any = ReflectionInjector.resolveAndCreate([
    ViewPortService,
    {
      provide: 'OtherSizeService',
      userFactory:(viewport: any) =>{
        return viewport.determineService();
      },
      deps: [ViewPortService]
    }
  ]);
  let sizeService: any = injector.get('OhterSizeService');
  sizeService.run();
}
```

### `NgModule`

1. 组件定义的合法标签
2. 被注入的对象来自哪里

## Data Architecture in Angular2

### Observables-Service

A few big ideas about streams
1. Promise emit a single value whereas streams emit many values
2. Imperative code pulls data whereas reactive streams push Data
3. RxJS is functional
4. Streams are composable


`BehaviorSubject` -stores the last value.

## `Forms` in Angular 2

`#v=thing` 在视图中创建了local变量

`ngModel` 在 `FormControl` 中单独使用,而非 `ngModel="whateve"` 含义:

1. 单向绑定
2. 创建了一个 FormControl 使用了 `input` 的 `name` 属性值

`ngForm` and `ngModel` create a new `FormGroup` and `FormControlls`;

`formGroup` and `formControl` bind to an existing `FormGroup` and `FormControl`

