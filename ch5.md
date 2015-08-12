# 什么是一个router?
Backbone routers是用来route我们应用的URL，当使用hash tags(`#`)的时候。在传统的MVC观念中，这个在语义上(semantics)是不符合的。并且，如果你阅读了[什么是view?]()的话，那篇文章也阐述了这个观点。尽管如此，Backbone的'router'对于许多需要URL routing/history capabilites的应用来说，他是十分有用的。

定义过的routers应该总是包含了至少一个route，以及映射到(map)这个route的函数。在下面的例子中，我们将会定义一个总是被调用的route。

同样需要注意的是，route会解释(interpret)在URL中`#`tag后面的所有东西。在你应用中所有的链接都应该像这样: `#/action`或者`#action`。(在hash tag后面加上个forward slash(`/`)似乎好看一点, `http://example.com/#/user/help`)

```html
<script>
  var AppRouter = Backbone.Router.extend({
    routes: {
      '*actions': 'defaultRoute' // 匹配http://example.co/#anything-here
    }
  });

  // 初始化router
  var app_router = new AppRouter();

  app_router.on('route:defaultRoute', function(actions) {
    alert(actions);
  });

  // 开始Backbone history，对于bookmarkable(可保存为书签的)URL是需要设置的
  Backbone.history.start();
</script>
```
点击链接后，注意地址栏URL的变化。
```html
<a href="#actions"> Activate route </a>
<a href="#/route/actions"> Activate another route </a>
```

## 动态的routing
大多数框架都会允许你在定义route的时候，可以混合静态和动态的route paramters。比如说，你也会想使用一个比较友好的URL string去获取你的post，通过一个post id变量，就像这样: `http://example.com/#/posts/12`。一旦route被激活之后，你就想通过URL string来访问post的`id`。

```javascript
var AppRouter = Backbone.Router.extend({
  routes: {
    // 最上面的route首先被匹配
    'posts/:id': 'getPost',
    '*actions': 'defaultRoute'
  }
});

// 实例化router
var app_router = new AppRouter();

app_router.on('route:getPost', function(id) {
  // 定义在route的参数变量可以被传到这里
  alert('Get post number ' + id);
});

app_router.on('route:defaultRoute', function(action) {
  alert('actions');
});

Backbone.history.start();
```

```html
<a href="#/posts/120"> Post 120 </a>
```

## 动态的routing cont. `:params` and `*splats`
Backbone在实现routes的时候有2种变量风格。第一种就是`:params`，他会匹配在slashs(`/`)中的所有URL components。然后，`splats`风格会匹配任何数量的URL components。因为是`splat`，所以自然而然他总是你URL的最后一个变量，因此他总是会匹配所有的components。

```javascript
routes: {
  // example.com/#/posts/121
  'posts/:id': 'getPost',

  // example.com/#/download/user/images/hey.gif
  'download/*path': 'downloadFile',

  // example.com/#/dashboard/graph
  ':route/:action': 'loadView'
}

app_router.on('route:getPost', function(id) {
  alert(id) // 121
});

app_router.on('route:downloadFile', function(path) {
  alert(path); // user/images/hey.gif
});

app_router.on('route:loadView', function(route, action) {
  alert(route + '_' + action); // dashboard_graph
});
```

Routes是很强大的，但是在理想情况下你的应用不应该包含太多routes。如果你需要在hash tags下实现SEO的话，你可以google `google seo hashtags`。或者看看[Seo Server](http://seo.apiengine.io/)。

## 相关链接
- [Backbone.js official router docs](http://documentcloud.github.io/backbone/#Router)
- [Using routes and understanding the hash tag](http://thomasdavis.github.io/2011/02/07/making-a-restful-ajax-app.html)
