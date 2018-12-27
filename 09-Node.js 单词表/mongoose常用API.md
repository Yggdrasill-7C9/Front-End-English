# Mongoose 常用API

## 1.connect

 用于数据库的链接

```javascript
var mongoose = require('mongoose'),
    DB_URL = 'mongodb://localhost:27017/test';

/**
 * 连接
 */
mongoose.connect(DB_URL);
```

## 2.connection

 connection是mongoose模块的默认引用，返回一个Connetion对象。因为connect()方法并不能监听数据库连接情况，所以，一般情况下此方法跟connet()方法搭配使用：

```javascript
var db = mongoose.connection;//获取connection实例
//使用Connetion监听连接状态
db.on('connected',function(err){
    if(err){
        console.log('连接数据库失败：'+err);
    }else{
        console.log('连接数据库成功！');
    }
});
```

## 3.schema

schema可以理解为mongoose对表结构的定义(不仅仅可以定义文档的结构和属性，还可以定义文档的实例方法、静态模型方法、复合索引等)，每个schema会映射到mongodb中的一个collection，schema不具备操作数据库的能力

```javascript
var studentsSechma=new mongoose.Schema({
    name:String,
    age:Number
})
```

#### 实例方法

 实例方法就是给model的实例使用的方法，在Schema上使用methods定义

```javascript
var mongoose=require("mongoose");
mongoose.connect("mongodb://localhost/test");
var animalSchema=new mongoose.Schema({
    name:String,
    type:String
});
//Schema定义的方法,model的实例就可以直接使用
animalSchema.methods.findSimilarType=function (cb) {
    //那个实例调用的，这个this就指向谁
    this.model("Animal").find({type:this.type},cb)
}
var Animal=mongoose.model("Animal",animalSchema);

var dog=new Animal({
    name:"小狗",
    type:"dog"
});

dog.save();

dog.findSimilarType(function (err,result) {
    console.log(result);
})
```

#### 静态方法

 静态方法是给model使用的，在Schema上使用statics定义

```javascript
var mongoose=require("mongoose");
mongoose.connect("mongodb://localhost/test");
var animalSchema=new mongoose.Schema({
    name:String,
    type:String
});
//Schema定义的静态方法
animalSchema.statics.findSimilarType=function (type,cb) {
    //这里的this指的是model
   this.find({type:type},cb)
}
var Animal=mongoose.model("Animal",animalSchema);

var dog=new Animal({
    name:"小狗",
    type:"dog"
});
Animal.findSimilarType("dog",function (err,result) {
    console.log(result);
})
```

## 4.Model

 　
 定义好了Schema，接下就是生成Model。
 model是由schema生成的模型，可以对数据库的操作

```javascript
var mongoose=require("mongoose");
mongoose.connect("mongodb://localhost/test");
var animalSchema=new mongoose.Schema({
    name:String,
    type:String
});
var Animal=mongoose.model("Animal",animalSchema);
```

## 5.Entity

 由model创建的实体，使用save方法保存数据，Model和Entity都能影响数据库的操作，但Model和Entity更具操作性

```javascript
var TestEntity = new TestModel({
       name : "Lenka",
       age  : 36,
       email: "lenka@qq.com"
});
console.log(TestEntity.name); // Lenka
console.log(TestEntity.age); // 36
```

**下面就来介绍一下数据库的常用操作**（操作数据库的都是model的实例切记）

### 1.**插入**

```javascript
// Model#save([fn])
var Animal=mongoose.model("Animal",animalSchema);

var dog=new Animal({
    name:"小狗",
    type:"dog"
});

dog.save();
```

### 2.**查询**

####  find

```javascript
Model.find(conditions, [fields], [options], [callback])
 User.find({'user' : 'ws'}, function(err, res){
        if (err) {
            console.log("Error:" + err);
        }
        else {
            console.log("Res:" + res);
        }
    })
```

第二个字段可以设置要查询的字段，1表示输出该字段，0表示不输出该字段

```javascript
 User.find({'user' : 'ws'},{"_id",0},function(err, res){
        if (err) {
            console.log("Error:" + err);
        }
        else {
            console.log("Res:" + res);
        }
    })
```

表示返回的结果即res里面没有_id字段

条件查询中常用属性

```php
$or　　　　#或关系
$nor　　　　#或关系取反
$gt　　　　#大于
$gte　　　　#大于等于
$lt　　　　#小于
$lte　　　　#小于等于
$ne　　　　#不等于
$in　　　　#在多个值范围内
$nin　　　　#不在多个值范围内
$all　　　　#匹配数组中多个值
$regex　　　　#正则，用于模糊查询
$size　　　　#匹配数组大小
$maxDistance　　　　#范围查询，距离（基于LBS）
$mod　　　　#取模运算
$near　　　　#邻域查询，查询附近的位置（基于LBS）
$exists　　　　#字段是否存在
$elemMatch　　　　#匹配内数组内的元素
$within　　　　#范围查询（基于LBS）
$box　　　　#范围查询，矩形范围（基于LBS）
$center　　　　#范围醒询，圆形范围（基于LBS）
$centerSphere　　　　#范围查询，球形范围（基于LBS）
$slice　　　　#查询字段集合中的元素（比如从第几个之后，第N到第M个元素
```

如要查询阅读量大于300小于400的文章

```javascript
Article.find({views : {$gte : 300 , $lte : 400}})
```

#### findById

```javascript
// findById 用来通过id查询单条文档
// Model.findById(id, [fields], [options], [callback])
 var id = '11111111aa359cb73';
    
    User.findById(id, function(err, res){
        if (err) {
            console.log("Error:" + err);
        }
        else {
            console.log("Res:" + res);
        }
    })
```

#### 模糊查询

```javascript
 User.find({'user':{$regex:/w/g}}, function(err, res){
        if (err) {
            console.log("Error:" + err);
        }
        else {
            console.log("Res:" + res);
        }
    })
```

全局查找有带w字符的用户么，$regex后面写的就是javascript正则表达式

#### findOne

 findOne 用来通过条件查询单条文档

重点
 population
 有很多场景都需要通过外键与另一张表建立关联，populate可以很方便的实现

首先我们创建两张表一个用户，一个评论

```javascript
var mongoose=require("mongoose");
mongoose.Promise=require("bluebird");
mongoose.connect("mongodb://localhost/population");

var Schema=mongoose.Schema;
var userSchema=new Schema({
    name:String,
    age:Number,
    comment:[{
        type:Schema.Types.ObjectId,
        ref:"Comment"
    }]
});
var User=mongoose.model("User",userSchema);

var commentSchema=new Schema({
    title:String,
    content:String,
    author:{
        type:Schema.Types.ObjectId,
        ref:"User"
    }
});

var Comment=mongoose.model("Comment",commentSchema);

var tom=new User({name:"tom",age:19});
var pl=new Comment({title:"pl",content:"this is a content"});

tom.save().then(function (user) {
    pl.author=user;
    return Promise.all([pl.save(),user]);
}).spread(function (comment,user) {
    user.comment.push(comment);
    return new Promise(function (resolve,reject) {
        resolve(user.save());
    });
}).then(function () {
    console.log("成功");
}).catch(function (err) {
    console.log(err);
})
```

如果我们使用

```javascript
User.find({name:"tom"}).then(function (result) {
    console.log(result);
});
```

输出的是

```json
[ { comment: [ 5b52eabcd5d53d1f34927ad3 ],
    _id: 5b52eabcd5d53d1f34927ad2,
    name: 'tom',
    age: 19,
    __v: 1 } ]
```

里面的comment只是一个id，并不能查到里面的东西

所以我们要使用population

语法

```javascript
// Query.populate(path, [select], [model], [match], [options])
// 1.path

// 指定要查询的表

// 2.select(可选)

// 指定要查询的字段

// 3.model(可选)

// 类型：Model，可选，指定关联字段的 model，如果没有指定就会使用Schema的ref。

// 4.match(可选)

// 类型：Object，可选，指定附加的查询条件。

// 5.options(可选)

// 类型：Object，可选，指定附加的其他查询选项，如排序以及条数限制等等。
User.find({name:"tom"}).populate("comment").exec(function (err,result) {
    console.log(result[0].comment[0]);
})
```

输出

```json
 { _id: 5b52eabcd5d53d1f34927ad3,
(node:10768) [DEP0079] DeprecationWarning: Custom inspection function on Objects via .inspect() is deprecated
  title: 'pl',
  content: 'this is a content',
  author: 5b52eabcd5d53d1f34927ad2,
  __v: 0 }
```

#### 分页查询

```javascript
// skip: (page - 1) * pageSize, //page : 当前页码, pageSize 每页显示条数
// limit: pageSize
var pageSize = 4;                   //一页多少条
    var currentPage =2;                //当前第几页
    var sort = {'logindate':-1};        //排序（按登录时间倒序）
    var condition = {};                 //条件
    var skipnum = (currentPage - 1) * pageSize;   //跳过数
    
    User.find(condition).skip(skipnum).limit(pageSize).sort(sort).exec(function (err, res) {
        if (err) {
            console.log("Error:" + err);
        }
        else {
            console.log("Res:" + res);
        }
    })
```

#### count

```javascript
// 语法Model.count(conditions, [callback])
 User.count({}, function(err, res){
        if (err) {
            console.log("Error:" + err);
        }
        else {
            console.log("Res:" + res);
        }
    })
```

### 3.**更新**

update

```javascript
// 语法Model.update(conditions, update, [options], [callback])

// $set` 指定字段的值，这个字段不存在就创建它。可以是任何MondoDB支持的类型。
Students.update({_id : id}, {$set : {name : "小文"...}})
    User.update({'user' : 'ws'}, {'pwd': '123'}, function(err, res){
        if (err) {
            console.log("Error:" + err);
        }
        else {
            console.log("Res:" + res);
        }
    })
```

### 4.**删除**

remove

```javascript
// 语法Model.remove(conditions, [callback])
 User.remove( {'user' : 'ws'}, function(err, res){
        if (err) {
            console.log("Error:" + err);
        }
        else {
            console.log("Res:" + res);
        }
    })
```

