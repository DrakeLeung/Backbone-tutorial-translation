# 什么是model?
由于网络上关于[MVC]()的定义十分混乱，所以很难准确地告诉你究竟什么是*model*。Backbone.js的作者有一个很清晰的定义，对于Backbone.js里面的model。

> Models是任何应用的核心，他包括了交互的数据，以及一大部分关于他的logic: conversions(转换), validation(验证), computed properties, 以及access control.

那么，让我们来创建一个model吧.
```javascript
var Person = Backbone.Model.extend({
  initialize: function() {
    alert('Welcome to this world');
  }
});

var person = new Person();
```
`initialize()`会被触发，无论你什么时候给model(models, collections, views同理)创建一个新的实例(instance)。当然，你也可以选择不定义`initialize()`。但是，你会发现你是需要经常用到他的。

## Setting Attributes
现在我们需要传一些参数，当我们创建一个model instance的时候。
```javascript
var Person = Backbone.Model.extend({
  initialize: function() {
    alert('welcom to this world');
  }
});

var person = new Person({
  name: 'Thomas',
  age: 67
});

// 当然，我们也可以像下面这样设置Attributes
var person = new Person();
person.set({
  name: 'Thomas',
  age: 67
});
```
所以，传一个JavaScript对象给我们的constructor跟调用`model.set()`是一样的。现在这些model都设置了attributes，所以我们可以获取(retrieve)他们。

## Getting attributes
使用`model.get()`方法，我们可以在任何时候访问model的properties。
```javascript
var Person = Backbone.Model.extend({
  initialize: function() {
    alert('Welcome to this world');
  }
});

var person = new Person({
  name: 'Thomas',
  age: 67
});

var age = person.get('age'); // 67
var name = person.get('name'); // 'Thomas'
```

## Setting Model Defaults
有时候，你会想你的model包含默认值。这是很容易实现的，只要你在model declaration的时候设置一个`defaults` property。
```javascript
var Person = Backbone.Model.extend({
  defaults: {
    name: 'Fetus',
    age: 0,
    child: ''
  }
});

var person = new Person({
  name: 'Thomas',
  age: 67,
  child: 'Ryan'
});

var age = person.get('age'); // 67
var name = person.get('name'); // 'Thomas'
var child = person.get('child'); // Ryan
```

## 操作model attributes
Models可以自定义你想要操作attributes的方法，默认情况下这些方法都是public。
```javascript
var Person = Backbone.Model.extend({
  defaults: {
    name: 'Fetus',
    age: 0,
    child: ''
  },

  initialize: function() {
    alert('Welcome to this world');
  },

  // custom method
  adopt: function(newChildName) {
    this.set({
      child: newChildName
    });
  }
});

var person = new Person({
  name: 'Thomas',
  age: 67,
  child: 'Ryan'
});

person.adopt('John Resig');
var child = person.get('child'); // 'John Resig'
```
因此，我们可以实现这些方法来get/set，或者用我们的model的attributes来进行某些计算。

## 监听model的改变
下面是Backbone比较有用的地方之一。所有model的attributes都可以绑定一个listener，用来检测他们的值是否发生了变化。在我们的`iinitialize()`中，我们打算绑定一个函数。当我们的attributes发生改变的时候就去调用他。在下面的例子中，当我们`person`的`name`发生改变了，我们就`alert`那个新的name。
```javascript
var Person = Backbone.Model.extend({
  initialize: function() {
    alert('Welcome to the world');

    this.on('change:name', function(model) {
      var name = model.get('name'); // 'Stewie Griffin'
      alert('Changed my name to ' + name);
    });
  }
});

var person = new Person({
  name: 'Thomas',
  age: 67
});

// This will trigger a change of 'name'
person.set({
  name: 'Stewie Griffin'
});
```
综上，我们可以给单独的一个attribute绑定change listener，也可以给所有attributes绑定。像这样：
```javascript
this.on('change', function(model) {
  // 只要model随便一个attribute改变，都会调用这个函数
});
```

## 和服务器交互
models其实代表这服务器的数据，以及你对数据进行的操作会被转换成RESTful operations。

model的`id` attribute表明了如何在数据库查找他，通常会映射到[surrogate key](https://en.wikipedia.org/wiki/Surrogate_key)。

为了教程的目的，我们不妨假设我们有一个叫做`Users`的mysql table，以及叫做`id`, `name`, `email`的3个column。

并且，服务器已经实现了RESTful URL `/user`，因此我们可以通过他来实现交互。

那么，我们的model应该是这样定义的:
```javascript
var UserModel = Backbone.Model.extend({
  urlRoot: '/user',
  defaults: {
    name: '',
    email: ''
  }
});
```
### 创建新的Model
如果我们想在服务器上面创建一个用户的话，我们需要实例化(instantiate)一个新的`UserModel`，然后调用`call()`方法。如果`id` attribute是`null`的话，Backbone.js就会发送一个`POST`请求到服务器的`urlRoot`。
```javascript
var UserModel = Backbone.Model.extend({
  urlRoot: '/user',
  defaults: {
    name: '',
    email: ''
  }
});

var user = new UserModel();

// 注意到我们并没有设置`id`
var userDetails = {
  name: 'Thomas',
  email: 'thomasalwyndavis@gmail.com'
};

// 因为我们并没有设置`id`，所以发送`post` /user请求
user.save(userDetails, {
  success: function(user) {
    alert(user.toJSON());
  }
})
// 服务器会保存userDetails，以及return一个响应，这个响应会包含一个新的`id`
```
在数据库的table中，我们应该有这样的值:
```
1, 'Thomas', 'thomasalwyndavis@gmail.com'
```

### 获取model
现在，我们已经有了一个新的user model，我们可以从服务器那里获取(fetch)他。我们可以知道`id`是`1`，从上面的例子中。

如果我们实例化了一个带有`id`的model，Backbone.js会自动向`urlRoot` + `/id`进行一个`get`请求。(考虑到RESTful convensions)

```javascript
var user = new UserModel({ id: 1});

// 下面的fetch会进行`GET /user/1` 请求
// success callback函数会返回一个user model，包含了id, name, email
user.fetch({
  success: function(user) {
    alert(user.toJSON());
  }
})
```

### 更新model
既然我们的model已经在server上面，那么我们就可以通过一个`PUT`请求来进行更新(update)。我们会使用`save()`这个API，如果有`id`的话，他就会发送一个`PUT`请求，而不是`POST`。
```javascript
var user = new UserModel({
  id: 1,
  name: 'Thomas',
  email: 'thomasalwyndavis@gmail.com'
})

// 我们改变了`name`，然后更新服务器
// 返回: {name: 'Davis', ...}
user.save({
  name: 'Davis'
}, {
  success: function(user) {
    alert(user.toJSON());
  }
})
```

### 删除model
同理，我们可以通过一个`id`来删除一个model。我们调用`destroy`这个方法，他会触发一个`DELETE /user/id`请求。
```javascript
var user = new UserModel({
  id: 1,
  name: 'Thomas',
  email: 'thomasalwyndavis@gmail.com'
});

user.destroy({
  success: function() {
    alert('Destroyed');
  }
})
```

## Tips & Tricks
获取model所有的attributes:
```javascript
var person = new Person({
  name: 'Thomas',
  age: 67
});

// 这个会返回一个attributes的copy
var attributes = person.toJSON();

// 这个是对attributes的一个引用，你应该要小心使用他。
// 最佳实践是你应该使用`model.set()`来修改model的attribute，因为这样可以利用Backbone的listener
var attributes = person.attributes;
```

检验(validate)数据，当你在set或者save他的时候:
```javascript
var Person = Backbone.Model.extend({
  var Person = Backbone.Model.extend({
    // 如果你在validate function返回了一个string的话
    // Backbone就会抛出一个错误
    validate: function(attributes) {
      if (attributes.age < 0 && attributes.name != 'Dr Manhatten') {
        return 'You cant be negative years old';
      }
    },

    initialize: function() {
      alert('Welcome to this world');
      this.bind('error', function(model, error) {
        // 我们会在这里收到一个error
        alert(error);
      });
    }
  });
});

var person = new Person();

// 下面将会触发一个错误，然后会alert这个错误
person.set({
  name: 'Mary Poppins',
  age: -1
});

person.set({
  name: 'Dr Manhatten',
  age: -1
});
```
