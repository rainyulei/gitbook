# mongDB

## mongoDB 一些特殊的地方

1. mongdb 中的主键 生成时间 自增长的数字和线程号 机器服务器号组合生成的
2. mongo 中操作条件就是在当前的数据解构中再加一层括号
3. mongo 中有几种层级
   - 1 db 库
   - 2 collection 表
   - 3 document 文档级别 也就是一次时候输入操作在 insert 语句括号中的整个插入对象
4. mongodb 事物支持不友好 需要做两次操作的时候 要自己实现事物，mongodb 的插入原子性都是基于方法来实现的
5. mongodb 主键 为\_id 默认的
6. 在查询字段的时候建议使用$ in 而不是$or
7. $elemMatch ， $slice ，以及 \$ 是用来指定返回数组中包含映射元素的 唯一 方式。不能使用数组索引来指定映射的特定数组元素；
   例如

   ```javascript
   db.users.find( { status: "A" }, { name: 1, status: 1, points: { $slice: -1 } } ) # 可以返回status 为A的 points数组中的的最后一个元素
    db.users.find( {} , "ratings.0": 1 })#映射不会映射到数组的第一个元素，这样写不对。
    db.users.find( { "badges.0": "black" } )#可以得到数组一个属性时black的
    # 也就是说$elemMatch ， $slice ，以及 \$  操作的是数组内映射关系，数组内元素关系 而不用这个几个操作符操作的是数组名

   ```

## mongoDB 增删改查 CRUD 操作

### 增 插入文档

mongo 中提供了三条命令来插入

    ```javascript
         # 返回 WriteResult 对象
      db.collection.insert()
      #version 3.2新增 插入一条  返回插入数据的 primary key  _id objectID
      db.collection.insertOne()
      # version 3.2 新增同时插入多条数据 一条失败前面的成功后面的全部失败
      db.collection.insertMany({document数组}，
         {writeConcern ： < document > ，
       ordered ： < boolean })
       #writeConcern 在安全写情况下,你可以指定MongoDB写操作要求的确认级别
       # ordered ： boolean 有序或者无序插入 默认为true的 如果设置为false 会提高性能
       # w: "majority", wtimeout: 100 插入的时间要求 超过时间判定失败
    ```

### 删除方法

    ```javascript
    db.collection.remove() #删除符合条件的 可由 justone属性指定 删除一条
    db.collection.deleteOne()#删除第一条符合条件的 3.2
    db.collection.deleteMany() #删除多条条件选中 3.2

    ```

### 查询的方法

1.  {}代表并且条件
2.  ObjectId 是一个 12 字节 BSON 类型数据，有以下格式：

        - 前 4 个字节表示时间戳
        - 接下来的 3 个字节是机器标识码
        - 紧接的两个字节由进程 id 组成（PID）
        - 最后三个字节是随机数

3.  \$ 后面接条件符号

    ```javascript
    db.user.find({age:12}) # 表明查询年龄等于12岁的
    db.user.find({age:{$gt:12}}) # 表明查询年龄大于12岁的
    db.user.find({age:{$gt:12，$lt:30}}) # 表明查询年龄大于12岁并且小于30 的
    db.user.find({age:{$or：[{$gt:12},{$lt:30}]}}) # 表明查询年龄大于12岁并且小于30 的
    db.user.find({age:{$exists:true}}) # 查询年龄字段存在
    ```

4.  数组的查询 用：方式表示数组的并行属于

    ```javascript
    db.user.find({数组;'数组内数据'})
    ```

5.  内嵌对象的查询，用点的方式 表示内嵌

    ```javascript
    db.user.find({对象.子对象;'要查询的内容'})
    ```

6.  数组内储存对象

    ```javascript
    #数组名：[{属性1：值},{},{}]
    db.user.find({"数组名:属性1"：值})可查出
    ```

7.  \$elemMatch 指定数组内多项条件同时过滤，数组内查询时的并且条件
8.  指定内置数据的位置

    ```javascript
    db.user.find({ '数组名.下标.属性': '值' })
    ```

9.  正则 不推荐使用正则筛选一个大体量的数据

    ```javascript
    db.user.find({name:{\$regex:"正则"}})
    ```

10. \$size 查询大小 但是会把数组中含有对象和含有字符串的全部查询出来因此数据的一致性有挑战

11. find 的第二个参数 显示字段

    ```javascript
    db.user.find({条件}，{name:1,age:1})
    # 1 显示  0  不显示此字段 ID 要不显示必须手动指定为0，此方法在显示数组内字段会有缺陷
    # 这样查询出来的就会值显示其 ID name 和age  三个字段  其他的字段全部隐藏
    # 这个可以显示内嵌在对象或者数组中的字段
    ```

12. 查询空字段
    如果在 mngodb 中查询某一个字段的值为空的时候需要指定`{ name : { $type: 10 } }`查询 仅仅 匹配那些包含值是 null 的 name 字段的文档,亦即 条目字段的值是 BSON 类型中的 Null (即 10 ):
13. 存在检查
    `{ name : { $exists: false } }` 检查是否存在该属性

### 更改

1. update 的\$set

   ```javascript
   db.user.update({条件}，{name:1,age:1})
   # 如果不写$set 会把user的匹配所有字段全部洗掉 变成后面的对象
   # 而加入$set后会自动查找匹配的字段 进行替换
   ```

2. update 的 \$unset 把字段删除 相当于删除某匹配字段

3. update 的第三个选项 {returnnewdocument：true} {motli:true}

   ```javascript
   db.user.findOneAndupdate({条件}，{name:1,age:1},{new:true})
   # {motli:true} 一起处理  多个一起
   #returnnewdocument：true 返回更改过后的新文档
   # upsert  有就更改   没有就插入  原子性的
   #
   ```

### 批量操作

bulkWrite（）方法 在使用的时候由两种模式

- 1 有序插入 通过选项 ordered true 默认
- 2 无序插入 设置为 ordered false

```javascript
    #bulkWrite（）支持以下写操作
       # insertOne()
       # updateOne()
       # updateMany()
       # replaceOne()
       # deleteOne()
       # deleteMany()
       #使用方法
       try {
   db.characters.bulkWrite(
      [
         { insertOne :
            {
               "document" :
               {
                  "_id" : 4, "char" : "Dithras", "class" : "barbarian", "lvl" : 4
               }
            }
         },
         { updateOne :
            {
               "filter" : { "char" : "Eldon" },
               "update" : { $set : { "status" : "Critical Injury" } }
            }
         },
         { deleteOne :
            { "filter" : { "char" : "Brisbane"} }
         },
         { replaceOne :
            {
               "filter" : { "char" : "Meldane" },
               "replacement" : { "char" : "Tanys", "class" : "oracle", "lvl" : 4 }
            }
         }
      ]
   );
}
catch (e) {
   print(e);
}
```

## 数据结构

浮点数的经度丢失需要使用 decimal 格式来处理 ，包括 java 和 c 和 javascript 都在使用而 mongoode 默认就会使用这个格式 ，
mongo 可以直接存 js 代码指定其类型
mongo 排序

## 聚合

aggregate 聚合操作 对表中的文档进行相应的处理
$match  相当于mysql的have
$group 相当于 groupby
\$unwide 拆分数组

```javascript
db.suer.aggregate()
```

## 索引

### 创建索引

db.user.createIndex({字段名：1}，{unique:1}) 索引 是不是唯一索引
删除索引 db.user.dropindex()
有索引之后根据索引查询速度会特别快 ，但是插入会很慢，使用空间来换时间，不会读取文档而是读取索引
