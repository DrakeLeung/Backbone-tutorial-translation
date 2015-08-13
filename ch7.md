# 使用Twitter API实现一个的简单的infinite Scrolling

**本教程已经deprecated(过时)，因为Twitter API现在需要oauth access only。尽管如此，我还是把代码留下来，给那些感兴趣的人。**

## 开始吧
在今次的例子中，我们打算创建一个widget。他可以刷新tweets。当用户的滚动条滚动到widget的底部时，我们就会re-sync到服务器，这样就会出现下一页的效果。

- [Demo Online](http://thomasdavis.github.io/backbonetutorials/examples/infinite-scroll/)
- [Code on GitHub](https://github.com/thomasdavis/backbonetutorials/tree/gh-pages/examples/infinite-scroll)

## Twitter collection
Twitter提供了一个浏览tweets的API。首先我们需要做的是在URL后面加上`&callback`，这样才能够发送跨域(cross-domain)的Ajax请求，这其实就是JSONP。

使用`q`和`page`这2个query参数，我们能够找到我们想要的东西。在定义collection的时候，我们有些默认值，但是都是可以被override。

由于Twitter的search API实际上会返回一大串东西，除了我们想要的，所以Backbone是无法正确解析的，因为他的Collection期望的是一个对象数组。因此，在我们定义Collections时，我们需要**override** Backbone默认的`parse()`函数。

```javascript
// collection/twitter.js

define([
  'jquery',
  'underscore',
  'backbone'
], function($, _, Backbone){
  var Tweets = Backbone.Collection.extend({
    url: function () {
      return 'http://search.twitter.com/search.json?q=' + this.query + '&page=' + this.page + '&callback=?'
    },
    // 因为twitter并没有返回一个model数组，所以我们需要重写他的parse()
    parse: function(resp, xhr) {
      return resp.results;
    },
    page: 1,
    query: 'backbone.js tutorials'
  });

  return Tweets;
});
```

你可以充分地利用Twitter返回的东西。比如，
```javascript
parse: function(resp, xhr) {
  this.completed_in = resp.completed_in
  return resp.results;
},
```

## 创建一个view
第一件需要做的事情就是加载我们的`Twitter` collection以及template到我们的`widget` model。这时，我们应该把我们的collection绑定到我们view的`initialize()`函数。而`loadRecords()`这个函数就负责调用我们`Twitter` collection的`fetch()`方法。如果成功的话，我们会把最新的结果填充到模版，然后append到我们的widget。

另外，我们会使用`event`来监听`el`元素的滚动(scroll)。如果当前的`scrollTop`是`el`元素的底部，那么我们就增加`Twitter` collections的`page` property，然后再次调用`loadRecords()`。

```javascript
// views/twitter/widget.js

define([
  'jquery',
  'underscore',
  'backbone',
  'vm',
  'collections/twitter',
  'text!templates/twitter/list.html'
], function($, _, Backbone, Vm, TwitterCollection, TwitterListTemplate){
  var TwitterWidget = Backbone.View.extend({
    el: '.twitter-widget',

    initialize: function () {
      // `isLoading`是一个很好的flag，这样我们就不会无谓地请求多次
      this.isLoading = false;
      this.twitterCollection = new TwitterCollection();
    },
    render: function () {
      this.loadResults();
    },
    loadResults: function () {
      var that = this;
      // 我们准备加载新的结果，所以设置`isLoading`为true
      this.isLoading = true;

      // `fetch`是Backbone.js的一个原生函数，调用后会解析collection的`url`
      this.twitterCollection.fetch({
        success: function (tweets) {
          // 让返回结果注入到我们的template
          $(that.el).append(_.template(TwitterListTemplate, {tweets: tweets.models, _:_}));

          that.isLoading = false;
        }
      });
    },

    // 这个会监听`el`元素的scroll动作
    events: {
      'scroll': 'checkScroll'
    },

    checkScroll: function () {
      var triggerPoint = 100; // 距离bottom 100px
        if( !this.isLoading && this.el.scrollTop + this.el.clientHeight + triggerPoint > this.el.scrollHeight ) {
          this.twitterCollection.page += 1; // 加载下一页
          this.loadResults();
        }
    }
  });
  return TwitterWidget;
});
```

## widget template
我们只需要使用underscore的`each()`函数来循环返回的tweets
```html
<ul class="tweets">
<% _.each(tweets, function(tweet) {
  <li> <%= tweet.get('text') &> </li>
<% }); &>
</ul>
```

## 总结
这个是很简单但是有很完整的infinite scroll。但在是UI/ux中使用这个效果是需要注意的。
