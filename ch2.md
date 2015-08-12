# What is a View?
Backbone views是用于表现你应用的data models是怎样子的。他们也可以用于监听事件和响应事件。本教程不再讲解如何绑定(bind) model和collections到views，而是专注于view的功能，以及怎么结合templating engine(模版引擎)去使用view，尤其是Underscore.js的`_.template`。

我们将会使用jQuery 1.8.2来操作我们的DOM。你可以可以使用其他library，比如MooTools或者Sizzle。但是官方是推荐使用jQuery的，因为Backbone.View events如果不用jQuery的话，有可能work不起来。

我们讲实现一个search box来作为demo，[在线例子](http://jsfiddle.net/tBS4X/1/)可以在jsFiddle找到。或者较新代码可在[codepen](http://codepen.io/DrakeLeung/pen/bdJVRV?editors=101)(译者作)
```javascript
var SearchView = Backbone.View.extend({
  initialize: function() {
    alert('Alerts suck.');
  }
});

// initialize()在实现一个Backbone View的时候都会被调用
// 你可以把他看作一个class的constructor
var search_view = new SearchView();
```

## The 'el' Property
`el`这个property引用的是在浏览器创建的DOM对象。每个Backbone view都会有一个`el` property，如果没有定义他的话，那么Backbone.js就会construct他默认的，也就是一个空的`div`元素。

```html
<div id="search_container"></div>
<script>
  SearchView = Backbone.View.extend({
    initialize: function() {
      alert('Alerts suck.');
    }
  });

  var search_view  new SearchView({
    el: $('#search_container')
  })
</script>
```

## Loading a Template
Backbone.js依赖于`Underscore.js`的模版引擎，查看[Underscore.js's Docs](http://documentcloud.github.com/underscore/)获取更多信息。

让我们来实现一个`render()`函数，并且在view初始化(initialize)后调用他。`render()`会加载我们的template到`el`所指向DOM对象里，通过jQuery。

> 提示: 这里纠正了原版写法，因为underscore到1.7.0版本后，`_.template()`语法改变了

```html
<!-- 我们的Template -->
<script type="text/template" id="search_template">
  <label> Search </label>
  <input type="text" id="search_input">
  <input type="button" id="search_button" value="Search">
</script>

<div id="search_container"></div>

<script>
  SearchView = Backbone.View.extend({
    initialize: function() {
      this.render();
    },

    render: function() {
      // 用underscore来编译我们的template
      var template = _.template($("#search_template").html())();

      // 加载编译过的template到我们的`el`
      this.$el.html(template);
    }
  });

  var search_view = new SearchView({
    el: $('#search_container')
  });
</script>
```
**Tip**: 把你的template放在一个文件里，然后把他们serve在CDN。这可以利用cache，加快页面速度。

## Listening for Events
为了绑定(attach)一个listener给我们的view，我们会使用`event`这个property。记住，listener值可以被绑定在`el` property的child elements。让我们绑定一个`click` listener给我们的button。
```html
<script type="text/template" id="search_template">
  <label> Search </label>
  <input type="text" id="search_input">
  <input type="button" id="search_button" value="Search">
</script>

<div id="search_container"></div>

<script>
  var SearchView = Backbone.View.extend({
    initialize: function() {
      this.render();
    },

    render: function() {
      var template = _.template($('#search_template').html())();

      this.$el.html(template);
    },

    events: {
      'click input[type=button]': 'doSearch'
    },

    doSearch: function(event) {
      // 当点击button后，可以通过'event.currentTarget'来访问元素
      alert('Search for ' + $('#search_input').val());
    }
  });

  var search_view = new SearchView({
    el: $('#search_container')
  });
</script>
```
## Tips and Tricks
使用template variable
```html
<script type="text/template" id="search_template">
  <label> <%= search_label %> </label>
  <input type="text" id="search_input">
  <input type="button" id="search_button" value="Search">
</script>

<div id="search_container"></div>

<script>
  var SearchView = Backbone.View.extend({
    initialize: function() {
      this.render()
    },

    render: function() {
      // 将要注入template的数据
      var variables = { search_label: 'My Search' };

      // 用underscore来编译template
      // 这里纠正了原版写法，因为underscore到1.7.0版本后，_.template()语法改变了
      var template = _.template($('#search_template').html())(variables);

      // 加载编译后的HTML到Backbone 'el'
      this.$el.html(template);
    },

    events: {
      'click input[type=button]': 'doSearch'
    },

    doSearch: function(event) {
      alert('Search for ' + $('#search_input').val());
    }
  });

  var search_view = new SearchView({
    el: $('#search_container')
  });
</script>
```
### 相关链接
- [This example implemented With google API](http://thomasdavis.github.io/2011/02/05/backbone-views-and-templates.html)
- [This example exact code on jsFiddle.net](http://jsfiddle.net/thomas/C9wew/4/)
- [Another semi-complete example on jsFiddle](http://jsfiddle.net/thomas/dKK9Y/6/)
