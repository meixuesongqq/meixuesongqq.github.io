---
layout: post
title: "Ember笔记"
date: 2015-05-02 10:18:49 +0800
comments: true
categories: 
- javascript
---

Ember是一个JavaScript框架。近日参与了一个采用Ember构建组件的开发工作。此处记录学习Ember的笔记。

<!--more-->

## 1. Getting Start
### Installation
注意不要使用sudo.

```
# install brew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
#install node
brew install node
brew install watchman
#install ember
npm install -g ember-cli
npm install -g phantomjs
```

### TESTING YOUR INSTALLATION
建立一个简单应用验证安装是否成功：

```
ember new my-app
cd my-app
ember server
```

## 2. Concepts
### 2.1 Core Concepts
#### Templates
用于UI，采用Handlebars templating language. Each template is backed by a model, Model变化时自动更新UI.

Templates可以包含:

1. Expressions： {{firstName}}
2. Outlets：包含其它Templates. You can put outlets into your template using the {{outlet}} helper.
3. Components, custom HTML elements that you can use to clean up repetitive templates or create reusable controls.

#### Router
完成URL与Template的映射。

#### Components
可重用的组件，用JavaScript定义行为，Template定义UI。

#### Models
存储对象状态。

#### Route
用于告诉Template应该显示哪个Model。

### 2.2 Naming Conventions
#### The Application

* app/app.js
* app/controllers/application.js
* app/templates/application.hbs
* ...

#### Simple Routes
每个Route都有一个Controller和一个Template. 当访问/favorites时，Ember.js将会查找以下类：

* app/routes/favorites.js
* app/controllers/favorites.js
* app/templates/favorites.hbs

对于route:favorites，可以指定Model:

```javascript
export default Ember.Route.extend({
  model: function() {
    // the model is an Array of all of the posts
    // fetched from this url
    return $.ajax('/a/service/url/where/posts/live');
  }
});
```

#### DYNAMIC SEGMENTS
指URL中包含参数。

```javascript
export default Ember.Router.extend().map(function(){
  this.route('post', { path: '/posts/:post_id' });
});

//app/routes/post.js
export default Ember.Route.extend({
  model: function(params) {
    return $.ajax('/my-service/posts/' + params.post_id);
  },

  serialize: function(post) {
    return { post_id: Ember.get(post, 'id') };
  }
});
```

由于这种模式很常见，因此约定：

* If your dynamic segment ends in _id, the default model hook will convert the first part into a model class on the application's namespace (post looks for app/models/post.js). It will then call find on that class with the value of the dynamic segment.
* The default behaviour of the serialize hook is to replace the route's dynamic segment with the value of the model object's id property.

#### ROUTE, CONTROLLER AND TEMPLATE DEFAULTS
If you don't specify a route handler for the post route (app/routes/post.js), Ember.js will still render the app/templates/post.hbs template with the app's instance of app/controllers/post.js.

If you don't specify the controller (app/controllers/post.js), Ember will automatically make one for you based on the return value of the route's model hook. If the model is an Array, you get an ArrayController. Otherwise, you get an ObjectController.

If you don't specify a post template, Ember.js won't render anything!

#### NESTING

```javascript
export default Ember.Router.extend().map(function(){
  this.route('posts', function() {    // the `posts` route
    this.route('favorites');          // the `posts.favorites` route
    this.route('post');               // the `posts.post` route
  });
});
```

#### THE INDEX ROUTE
At every level of nesting (including the top level), Ember.js automatically provides a route for the / path named index.

```javascript
export default Ember.Router.extend().map(function(){
  this.route('index', { path: '/' });  //可以省略，默认自带
  this.route('favorites');
});
```

## 3. The Object Model
### 3.1 Classes and Instances

```javascript
Person = Ember.Object.extend({
  say: function(thing) {
    var name = this.get('name');
    alert(name + " says: " + thing);
  }
});

Soldier = Person.extend({
  say: function(thing) {
    this._super(thing + ", sir!");
  }
});

var yehuda = Soldier.create({
  name: "Yehuda Katz"
});

yehuda.say("Yes"); // alerts "Yehuda Katz says: Yes, sir!"
```

>For performance reasons, note that you cannot redefine an instance's computed properties or methods when calling create(), nor can you define new ones. You should only set simple properties when calling create(). 

init方法会在对象创建时自动调用，因此可用于初始化一些设置。如果是继承自框架类，如Ember.View or Ember.ArrayController,一定要在init中调用this._super()：

```javascript
Person = Ember.Object.extend({
  init: function() {
    var name = this.get('name');
    alert(name + ", reporting for duty!");
  }
});
```

### 3.2 COMPUTED PROPERTIES

```javascript
Person = Ember.Object.extend({
  // these will be supplied by `create`
  firstName: null,
  lastName: null,

  fullName: function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  }.property('firstName', 'lastName')
  //property方法用于指定依赖的属性, Ember扩展了function prototype。
});

var ironMan = Person.create({
  firstName: "Tony",
  lastName:  "Stark"
});

ironMan.get('fullName'); // "Tony Stark"
```

如果 [disabling prototype extensions guide](http://guides.emberjs.com/v1.11.0/configuring-ember/disabling-prototype-extensions/)，可以改成：

```javascript
fullName: Ember.computed('firstName', 'lastName', function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  })
```

**CHAINING COMPUTED PROPERTIES**，计算字段可以再依赖计算字段。

```javascript
  fullName: function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  }.property('firstName', 'lastName'),

  description: function() {
    return this.get('fullName') + '; Age: ' + this.get('age') + 
    	'; Country: ' + this.get('country');
  }.property('fullName', 'age', 'country')
```

任何计算字段的依赖项发生变化时，都会自动触发计算字段重新计算。

**SETTING COMPUTED PROPERTIES**

```javascript
Person = Ember.Object.extend({
  firstName: null,
  lastName: null,

  fullName: function(key, value, previousValue) {
    // setter
    if (arguments.length > 1) {
      var nameParts = value.split(/\s+/);
      this.set('firstName', nameParts[0]);
      this.set('lastName',  nameParts[1]);
    }

    // getter
    return this.get('firstName') + ' ' + this.get('lastName');
  }.property('firstName', 'lastName')
});


var captainAmerica = Person.create();
captainAmerica.set('fullName', "William Burnside");
captainAmerica.get('firstName'); // William
captainAmerica.get('lastName'); // Burnside
```

### 3.3 COMPUTED PROPERTIES AND AGGREGATE DATA WITH @EACH

`@each`instructs Ember.js to update bindings and fire observers for this computed property. 下面的例子中，当isDone属性、数组元素数量发生变化、todos换成另一个数组时，都会触发更新。

```
export default Ember.Controller.extend({
  todos: [
    Ember.Object.create({ isDone: true }),
    Ember.Object.create({ isDone: false }),
    Ember.Object.create({ isDone: true })
  ],

  remaining: function() {
    var todos = this.get('todos');
    return todos.filterBy('isDone', false).get('length');
  }.property('todos.@each.isDone')
});
```

### 3.4 Observers













参考：[Ember Guide](http://guides.emberjs.com/v1.11.0/)