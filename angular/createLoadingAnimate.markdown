# 自定义 Angular 4 首屏加载动画

>默认情况下，Angular应用程序在首次加载根组件时，会在浏览器的显示一个`loading... `我们可以轻松地将loading修改成我们自己定义的动画。

#### 这是我们要实现首次加载的效果:

![loading](https://d33wubrfki0l68.cloudfront.net/c6d0349488d9b7732f8378dbf70025a0b4cb20f1/01e91/images/angular/loading-screen/app-loading.gif)

#### 根组件标签中的内容

请注意，在你的入口文件index.html中，默认的`loading...`只是插入到根组件标签之间：

```
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Fancy Loading Screen</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>

  <app-root>Loading...</app-root>

</body>
</html>

```

如果您在加载完根组件检查应用程序，则无法找到`loading...` 的文字，因为它在应用加载完成后被我们自己定义的组件替换掉。

这意味着我们可以在这些标签之间放置任何内容，包括样式定义，一旦`Angular`加载完根组件，就可以完全清除它们。

```
<app-root>
  <style>
    app-root {
      color: purple;
    }
  </style>
  I'm a purple loading message!
</app-root>

```

我们不必担心这些样式会影响我们的应用程序加载后的内容，因为一切都被完全替换掉。

现在你可以在那里随意的做任何事情。使用`css`或者`svg`实现自定义加载动画。

在我们的示例中，我们给页面一个粉红色的背景，我们使用`Flexbox` 将loaidng设置居中，给它设置一个更漂亮的字体，我们甚至在省略号上添加一个自定义动画：

```
<app-root>
  <style>
  app-root {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;

    color: pink;
    text-transform: uppercase;
    font-family: -apple-system,
        BlinkMacSystemFont,
        "Segoe UI",
        Roboto,
        Oxygen-Sans,
        Ubuntu,
        Cantarell,
        Helvetica,
        sans-serif;
    font-size: 2.5em;
    text-shadow: 2px 2px 10px rgba(0,0,0,0.2);
  }
  body {
    background: salmon;
    margin: 0;
    padding: 0;
  }

  @keyframes dots {
    50% {
      transform: translateY(-.4rem);
    }
    100% {
      transform: translateY(0);
    }
  }

  .d {
    animation: dots 1.5s ease-out infinite;
  }
  .d-2 {
    animation-delay: .5s;
  }
  .d-3 {
    animation-delay: 1s;
  }
  </style>

  Loading<span class="d">.</span><span class="d d-2">.</span><span class="d d-3">.</span>
</app-root>

```

分享几个loading效果的在线素材网：

* [loading.io](https://loading.io/)
* [css-loaders](https://projects.lukehaas.me/css-loaders/)
* [cssload](http://cssload.net/)

好了，去创建属于你自己的loading吧!

这样我们就实现了上图的加载效果了，[点击这里查看原文](https://alligator.io/angular/custom-loading-screen/)

