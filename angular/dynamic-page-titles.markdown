# Angular 2+ 监听路由变化动态设置页面标题
>现在很多web网站都采用了SPA单页应用，单页面有很多优点：用户体验好、应用响应快、对服务器压力小 等等。同时也有一些缺点：首次加载资源太多，不利于SEO，前进、后退、地址栏需要手动管理。今天我们实现`Angular`单页面应用中路由变化设置页面标题，来优化用户的用户体验。可以先去掘金看下效果。[稀土掘金](https://juejin.im/)

在AngularJS（1.x）中动态设置页面标题通常是通过一个全局$rootScope对象来完成的，通过$rootScope对象监听路由变化获取当前路由信息并映射到页面标题。在Angular（v2 +）中，解决起来要比1.x容易得多，我们可以通过注入一个provider，在路由变化事件中使用provider提供的API来动态更新页面标题。

##Title Service

在angular中，我们可以通过Title来设置页面标题。我们从`platform-browser`导入`Title`, 同时也导入`Router`。
```
import { Title } from '@angular/platform-browser';
import { Router } from '@angular/router';
```

导入之后，我们在组件的构造函数中注入他们

```
@Component({
  selector: 'app-root',
  templateUrl: `
    <div>
      Hello world!
    </div>
  `
})
export class AppComponent {
  constructor(private router: Router, private titleService: Title) {}
}

```

在使用`Title`之前，我们先看下`Title`是如何定义的
```
export class Title {
  /**
   * Get the title of the current HTML document.
   * @returns {string}
   */
  getTitle(): string { return getDOM().getTitle(); }

  /**
   * Set the title of the current HTML document.
   * @param newTitle
   */
  setTitle(newTitle: string) { getDOM().setTitle(newTitle); }
}

```

`Title`类有两个方法，一个用来获取页面标题`getTitle`， 一个是用来设置页面标题的`setTitle`

要更新页面标题，我们可以简单的调用`setTitle`方法:

```
@Component({...})
export class AppComponent implements OnInit {
  constructor(private router: Router, private titleService: Title) {}
  ngOnInit() {
    this.titleService.setTitle('My awesome app');
  }
}

```
这样就可以设置我们的页面标题了，但是很不优雅。我们接着往下看。

在AngularJS中，我们可以使用ui-router为每个路由添加一个自定义对象，自定义的对象在路由器的状态链中继承：

```
// AngularJS 1.x + ui-router
.config(function ($stateProvider) {
  $stateProvider
    .state('about', {
      url: '/about',
      component: 'about',
      data: {
        title: 'About page'
      }
    });
});

```

在Angular2+中，我们也可以为每个路由定义一个data对象，然后再在监听路由变化时做一些额外的逻辑处理就可以实现动态设置页面标题。首先，我们定义一个基本的路由：
```
const routes: Routes = [{
  path: 'calendar',
  component: CalendarComponent,
  children: [
    { path: '', redirectTo: 'new', pathMatch: 'full' },
    { path: 'all', component: CalendarListComponent },
    { path: 'new', component: CalendarEventComponent },
    { path: ':id', component: CalendarEventComponent }
  ]
}];
```
在这里定义一个日历应用，他有一个路由`/calendar`， 还有三个子路由， `/all`对应日历列表页，`new`对应新建日历，`:id`对应日历详情。现在，我们定义一个`data`对象然后设置一个`title`属性来作为每个页面的标题。
```
const routes: Routes = [{
  path: 'calendar',
  component: CalendarComponent,
  children: [
    { path: '', redirectTo: 'new', pathMatch: 'full' },
    { path: 'all', component: CalendarListComponent, data: { title: 'My Calendar' } },
    { path: 'new', component: CalendarEventComponent, data: { title: 'New Calendar Entry' } },
    { path: ':id', component: CalendarEventComponent, data: { title: 'Calendar Entry' } }
  ]
}];
```
好了，路由定义完了，现在我们看下如何监听路由变化

##Routing events

Angular路由配置非常简单，但是路由通过Observables使用起来也非常强大。
我们可以在根组件中全局监听路由的变化:
```
ngOnInit() {
  this.router.events
    .subscribe((event) => {
      // example: NavigationStart, RoutesRecognized, NavigationEnd
      console.log(event);
    });
}
```
我们要做的就是在导航结束时获取到定义的数据然后设置页面标题，可以检查 [NavigationStart](https://angular.cn/docs/ts/latest/api/router/index/NavigationStart-class.html), [RoutesRecognized](https://angular.cn/docs/ts/latest/api/router/index/RoutesRecognized-class.html), [NavigationEnd](https://angular.cn/docs/ts/latest/api/router/index/NavigationEnd-class.html) 哪种事件是我们需要的方式，理想情况下`NavigationEnd`，我们可以这么做：
```
this.router.events
  .subscribe((event) => {
    if (event instanceof NavigationEnd) { // 当导航成功结束时执行
      console.log('NavigationEnd:', event);
    }
  });
```
这样我们就可以在导航成功结束时做一些逻辑了，因为Angular路由器是`reactive`响应式的，所以我们可以使用 [RxJS](https://github.com/Reactive-Extensions/RxJS) 实现更多的逻辑，我们来导入以下操作符：

```
import 'rxjs/add/operator/filter';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/mergeMap';
```

现在我们已经添加了 `filter`，`map` 和 `mergeMap` 三个操作符，我们可以过滤出导航结束的事件：
```
this.router.events
  .filter(event => event instanceof NavigationEnd)
  .subscribe((event) => {
    console.log('NavigationEnd:', event);
  });
```
其次，因为我们已经注入了Router类，我们可以使用 [routerState](https://angular.cn/docs/ts/latest/api/router/index/RouterState-interface.html) 来获取路由状态树得到最后一个导航成功的路由：

```
this.router.events
  .filter(event => event instanceof NavigationEnd)
  .map(() => this.router.routerState.root)
  .subscribe((event) => {
    console.log('NavigationEnd:', event);
  });
```
然而，一个更好的方式就是使用 [ActivatedRoute](https://angular.cn/docs/ts/latest/api/router/index/ActivatedRoute-interface.html) 来代替 `routerState.root`,  我们可以将其`ActivatedRoute`注入类中：
```
import { Router, NavigationEnd, ActivatedRoute } from '@angular/router';

@Component({...})
export class AppComponent implements OnInit {
  constructor(
    private router: Router,
    private activatedRoute: ActivatedRoute,
    private titleService: Title
  ) {}
  ngOnInit() {
    // our code is in here
  }
}
```
注入之后我们再来优化下：
```
this.router.events
  .filter(event => event instanceof NavigationEnd)
  .map(() => this.activatedRoute)
  .subscribe((event) => {
    console.log('NavigationEnd:', event);
  });
```
我们使用 `map` 转换了我们观察到的内容，返回一个新的对象 `this.activatedRoute` 在 `stream` 流中继续执行。 我们使用 `filter(过滤出导航成功结束)` 和 `map(返回我们的路由状态树)` 成功地返回我们想要的事件类型 `NavigationEnd`。

接下来是最有意思的部分，我们将创建一个while循环遍历状态树得到最后激活的 route，然后将其作为结果返回到流中：
```
this.router.events
  .filter(event => event instanceof NavigationEnd)
  .map(() => this.activatedRoute)
  .map(route => {
    while (route.firstChild) route = route.firstChild;
    return route;
  })
  .subscribe((event) => {
    console.log('NavigationEnd:', event);
  });
```
接下来我们可以通过路由配置的属性来获取相应的页面标题。然后，我们还需要另外两个运算符：
```
this.router.events
  .filter(event => event instanceof NavigationEnd)
  .map(() => this.activatedRoute)
  .map(route => {
    while (route.firstChild) route = route.firstChild;
    return route;
  })
  .filter(route => route.outlet === 'primary')  // 过滤出未命名的outlet，<router-outlet>
  .mergeMap(route => route.data)                // 获取路由配置数据
  .subscribe((event) => {
    console.log('NavigationEnd:', event);
  });
```

现在我们 `titleService` 只需要实现：
```
.subscribe((event) => this.titleService.setTitle(event['title']));

```
下面看一下最终代码：
```
import 'rxjs/add/operator/filter';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/mergeMap';

import { Component, OnInit } from '@angular/core';
import { Router, NavigationEnd, ActivatedRoute } from '@angular/router';
import { Title } from '@angular/platform-browser';

@Component({...})
export class AppComponent implements OnInit {
  constructor(
    private router: Router,
    private activatedRoute: ActivatedRoute,
    private titleService: Title
  ) {}
  ngOnInit() {
    this.router.events
      .filter(event => event instanceof NavigationEnd)
      .map(() => this.activatedRoute)
      .map(route => {
        while (route.firstChild) route = route.firstChild;
        return route;
      })
      .filter(route => route.outlet === 'primary')
      .mergeMap(route => route.data)
      .subscribe((event) => this.titleService.setTitle(event['title']));
  }
}
```
本文翻译自[dynamic-page-titles-angular-2-router-events](https://toddmotto.com/dynamic-page-titles-angular-2-router-events), 本人水平有限，如果有翻译不好的地方欢迎大家联系我
