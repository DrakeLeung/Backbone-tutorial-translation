**原文链接:https://cdnjs.com/libraries/backbone.js/tutorials/organizing-backbone-using-modules**

# 用require.js模块化你的代码
很遗憾，Backbone.js并没有告诉你如何组织你的代码，因此会让许多开发者陷入如何加载脚本(load scripts)和搭建他们的开发环境的黑暗中。

这是和其他MVC框架一个很不同的地方，因为他们都赞成给开发者搭建好开发环境的理论。

但是，本文会让你搭建一个健壮的项目，充分分离界面(design)和代码之间的concern(耦合)。

下面我们会让你结合[AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)(Async Module Definitions)来使用Backbon.js。

## 什么是AMD?
AMD的设计理念是在浏览器或者服务器中**异步加载模块化的代码**。他其实是**Common.js**规范的一个实现。许多脚本加载器(script laoder)都是基于AMD来实现的，由此可见AMD是JavaScript未来模块化的基础。

我们会使用[Require.js](http://requirejs.org/)来实现模块化，有组织的Backbone应用。

**我强烈建议使用AMD来开发应用**。

快速浏览

- 模块化(modular)
- 可扩展(Scalable)
- 编译好(查看[r.js](http://requirejs.org/docs/optimization.html))
- 市场(Dojo 1.6已经全面转到AMD)

## Why Require.js
`Require.js`有一个活跃的社区(community)，而且他成长很迅速。[James Burke](http://tagneto.blogspot.com/)，也就是Require.js的作者，已经全身投入到Require.js且不断等待用户的feedback。他是脚本加载的专家，也是AMD规范的contributor(贡献者)。

## 开始吧
为了理解这个教程，你应该直接跳到这个例子，看看他大概是怎样子的。
- [Code on GitHub](https://github.com/thomasdavis/backbonetutorials/tree/gh-pages/examples/modular-backbone)
- [Demo Online](http://thomasdavis.github.io/backbonetutorials/examples/modular-backbone)

这个教程是很松散的，所以你需要上面的例子来帮助你理解。

这个例子不是很生动，但是可以给你一个大概的印象。

## 文件结构
根据项目大小和类型，文件目录的安排有很多种方法。在下面的例子中，`views`和`templates`的目录结构是相似的。`Collections`和`Model`的结构就有点像ORM。

```
|—— index.html
|—— imgs
|—— css
|   |__ style.css
|
|—— templates
|   |—— projects
|   |   |—— list.html
|   |   |__ edit.html
|   |
|   |__ users
|       |—— list.html
|       |__ edit.html
|__ js
    |—— libs
    |   |—— jquery
    |       |—— jquery.min.js
    |   |—— backbone
    |       |—— backbone.min.js
    |   |—— underscore
    |       |—— underscore.min.js
    |
    |—— models
    |   |—— users.js
    |   |—— projects.js
    |
    |—— collections
    |   |—— users.js
    |   |__ projects.js
    |
    |—— views
    |   |—— projects
    |   |   |—— list.js
    |   |   |__ edit.js
    |   |
    |   |—— users
    |       |—— list.js
    |       |__ edit.js
    |
    |—— router.js
    |—— app.js
    |—— main.js  // bootstrap
    |—— order.js // Require.js plugin
    |__ text.js // Require.js plugin
```

## 启动(bootstrap)你的app
我们用Require.js给我们的`index.html`定义了一个单一的入口点(entry point)。我们应该把任何有用的东西放在我们的Backbone views里面。

**Note**: `data-main`这个attribute会告诉Require.js去加载位于`js/mian.js`的脚本。他自动在后面补上`.js`后缀。

```html
<!-- index.html -->

<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <!-- 加载'js/main.js'作为我们的entry point(入口点) -->
  <script data-main="js/main" src="js/libs/require/require.js"></script>
</head>

<body>
  <section id="container">
    <div id="menu"></div>
    </div id="content"></div>
  </section>

</body>
</html>
```
你应该尽量保持你的`index.html`轻量(light weight)。你可以把他serve到服务器，其他的文件就serve到CDN(比如Clondfront)，这样你就可以大大利用cache了。

## bootstrap到底长什么样子的
我们的bootstrap文件负责配置`Require.js`，同时也初始化地加载一些依赖(dependencies)。

我们会请求一个叫`app`的模块，他会包含整个web应用的logic。

**Note**: 模块会相对于bootstrap文件(main.js)来加载，同时会自动补全`.js`后缀。所以加载`app`就相当于加载跟bootstrap文件同一个目录的`app.js`文件。

在下面的例子中，我们通过创建一个常用脚本的别名(alias)来配置Require.js。
```javascript
// main.js

// Require.js允许我们给常用的脚本配置alias
require.config({
  paths: {
    jquery: 'libs/jquery/jquery.min',
    underscore: 'libs/underscore/underscore.min',
    backbone: 'libs/backbone/backbone.min'
  }
});

require([
  // 加载app.js模块
  'app'
], function(App) {
  App.initialize();
})
```

## 我们应该怎么编排(lay out)我们的外部脚本?
我们开发web应用的所用到的模块都会使用AMD/Require.js来异步加载。

我们已经加载了jQuery, Underscore以及Backbone。但是很不幸的是，这些library都会同步加载，并且都在全局命名空间(gloable namespace)中，且相互依赖。

## 一个叫'boiler plate'的模块
在我们开始构建我们的应用之前，让我们快速看一遍我们将会经常用到的代码(boiler plate code)。

为了方便，我都会保存一个叫`boilerplate.js`的文件在我的项目根目录中，所以我可以复制粘贴。

```javascript
// boilerplate.js

define([
  // 这是我们在bootstrap文件中已经配置好的alias
  'jquery', // lib/jquery/jquery
  'underscore',
  'backbone'
], function($, _, Backbone) {
  // 我们return的对象都可以被其他模块使用
  return {};
});
```
`define()`函数的第一个参数是一个数组，里面放的是我们所需要依赖的模块。

## App.js构建我们应用的主要模块
我们应用的主要模块(`app.js`)应该尽量保持轻量。这个教程只是建立了一个Backbone Router，以及初始化他。

```javascript
// app.js

define([
  'jquery',
  'underscore',
  'backbone',
  'router'
], function($, _, Backbone, Router) {
  var initialize = function() {
    Router.initialize();
  };

  return {
    initialize: initialize
  };
});
```

```javascript
// router.js

define([
  'jquery', // lib/jquery/jquery
  'underscore',
  'backbone',
  'views/projects/list',
  'views/users/list'
], function($, _, Backbone, ProjectListView, UserListView) {
  var AppRouter = Backbone.Router.extend({
    routes: {
      '/projects': 'showProjects',
      '/users': 'showUsers',
      '*actions': 'defaultAction'
    }
  });

  var initialize = function() {
    var app_router = new AppRouter();

    app_router.on('showProjects', function() {
      var projectListView = new ProjectListView();
      projectListView.render();
    });

    app_router.on('showUsers', function() {
      var userListView = new UserListView();
      userListView.render();
    });

    app_router.on('defaultAction', function(actions) {
      console.log('No route: ', actions);
    });
  };

  return {
    initialize: initialize
  };
});
```

## 模块化的Backbone View
Backbone views会经常和DOM交互。用了新的模块系统，即是`Require.js text!`这个插件(plug-in)，我们就可以加载JavaScript templates，

```javascript
// views/projects/list
define([
  'jquery', // lib/jquery/jquery
  'underscore',
  'backbone',
  // 使用了Require.js text!这个插件，我们就可以加载raw text
  'text!templates/projects/list.html'
], function($, _, Backbone, projectListTemplate) {
  var ProjectListView = Backbone.View.extend({
    el: $('#container'),
    render: function() {
      var data = {};
      var compiledTemplate = _.template(projectListTemplate)(data);

      this.$el.append(compiledTemplate);
    }
  });

  return projectListTemplate;
});
```
JavaScript模版引擎可以让我们分离界面(design)和逻辑(logic)，通过把我们所有的HTML都放在template文件夹里。

## 模块化Colletion, Model以及View
现在就让我们把Model, Collection以及View串连起来吧，这也是构建Backbone View的一个常用场景。

首先，我们会定义一个model:
```javascript
// models/project

define([
  'underscore',
  'backbone'
], function(_, Backbone) {
  var ProjectModel = Backbone.Model.extend({
    defaults: {
      name: 'Harry Potter'
    }
  });

  return ProjectModel;
});
```

现在我们已经有了一个model，而我们的collection是依赖他的。为了加载这个模块，我们会给collection设置`model`这个attribute。
```javascript
// collections/projects

define([
  'underscore',
  'backbone',
  'models/project'
], function(_, Backbone, ProjectModel) {
  var ProjectCollection = Backbone.Collection.extend({
    model: ProjectModel
  });

  return ProjectCollection;
});
```

现在，我们就可以把collection注入到我们的template。
```javascript
// views/projects/list

define([
  'jquery', // lib/jquery/jquery
  'underscore',
  'backbone',
  // 加载我们的collection
  'collections/projects'
  'text!templates/projects/list.html'
], function($, _, Backbone, ProjectsCollection, projectListTemplate) {

  var ProjectListView = Backbone.View.extend({
    el: $('#container'),

    initialize: function() {
      this.collection = new ProjectsCollection();
      this.collection.add({
        name: 'Ginger Kid'
      });

      var compiledTemplate = _.template(projectListTemplate)(this.colletion.models);
      this.$el.html(compiledTemplate);
    }

  });

  return projectListTemplate;
});;
```

## 相关链接
- [Organizing Your Backbone.js Application With Modules](http://bocoup.com/weblog/organizing-your-backbone-js-application-with-modules/)


**原文链接:https://cdnjs.com/libraries/backbone.js/tutorials/organizing-backbone-using-modules**
