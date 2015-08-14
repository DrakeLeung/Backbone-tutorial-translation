**原文链接**: https://cdnjs.com/libraries/backbone.js/tutorials/what-is-a-collection

# 什么是collection?
Backbone的collection简单来说就是models的有序表(ordered set)，所以他会在下面的场景中使用:

- Model: Student, Collection: ClassStudents
- Model: Todo Item, Collection: Todo List
- Model: Animals, Collection: Zoo

你的collection通常情况下都只会使用同一种类型的model，虽然允许不同类型的model。

下面是一个常用的例子:
```javascript
var Song = Backbone.Model.extend({
  initialize: function() {
    console.log('Music is the answer');
  }
});

var Album = Backbone.Collection.extend({
  model: Song
});
```

## 创建一个collection
现在我们可以给collection注入(populate)一些有用的数据。
```javascript
var Song = Backbone.Model.extend({
  initialize: function() {
    console.log('Music is the answer');
  }
});

var Album = Backbone.Collection.extend({
  model: Song
});

var song1 = new Song({
  name: 'Sexual Healing',
  artist: 'Marvin Gaye'
});

var song2 = new Song({
  name: 'Talk It Over the Bed',
  artist: 'OMC'
});

var myAlbum = new Album([song1, song2]);

console.log(myAlbum.models); // [song1, song2]
```

**原文链接**: https://cdnjs.com/libraries/backbone.js/tutorials/what-is-a-collection
