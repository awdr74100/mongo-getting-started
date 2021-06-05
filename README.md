### 添加環境變數

- Windows

```shell
$ $old = (Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment' -Name path).path
$ $new = $old + "C:\Program Files\MongoDB\Server\4.4\bin"
$ Set-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment' -Name path -Value $new
```

### 子進程啟動

- Windows

```shell
$ net start mongodb
$ net stop mongodb
```

- Mac

```shell
$ sudo mongod --fork --dbpath="C:\db\data" --logpath="C:\db\log\mongod.log" --logappend
$ sudo db.shutdownServer()
```

### 指定配置文件

```shell
$ mongod -f "C:\db\mongod.cfg"
```

### 啟動外殼

```shell
$ mongo
```

### 列出所有 Database

```shell
$ show dbs
```

### 切換指定 Database

```shell
$ use shop
```

### 列出目前 Database

```shell
$ db
```

### 刪除目前 Database

```shell
$ db.dropDatabase()
```

### 查看統計 Database

```shell
$ db.stats()
```

### 列出所有 Collection

```shell
$ show collections
```

### 刪除指定 Collection

```shell
$ db.users.drop()
```

### 查看統計 Collection

```shell
$ db.users.stats()
```

### 重命名 Collection

```shell
$ db.users.renameCollection("groups")
```

### 指定 Collection 插入 Document

> 重複 id 時會引發錯誤，Document 不會被插入

```shell
$ db.users.insertOne({})
$ db.users.insertOne({ name: "Ian", age: 18 })
$ db.users.insertOne({ _id: "k1", name: "Jack", age: 19 })

---

$ db.users.insertMany([{ name: "Jack", age: 18, weight: 66, height: 180 }])
$ db.users.insertMany([{ name: "Jack", age: 18 }, { name: "Owen", age: 19, online: true }])

---

$ db.users.insert({ name: "Ian", age: 18 }) # 不建議使用，參考 insertOne
$ db.users.insert([{ name: "Ian", age: 18 }, { name: "Eric", age: 20 }]) # 不建議使用，參考 insertMany

---

$  db.users.insertMany([{ _id: 444, name: "Tidy" }, { _id: 222, name: "Owen" }], { ordered: false })
```

### 指定 Collection 讀取 Document

> 第二個參數可帶入 [projection](https://reurl.cc/L0Ov4a)

> find 回傳的對象為 [cursor](https://reurl.cc/WEmeR7)

```shell
$ db.users.findOne()
$ db.users.findOne({})
$ db.users.findOne({ verify: true })
$ db.users.findOne({ verify: true, age: 19 })
$ db.users.findOne({ _id: ObjectId("6086b8106fddb5a5c5959574") })

---

$ db.users.find()
$ db.users.find().pretty()
$ db.users.find({})
$ db.users.find({ verify: false })
$ db.users.find({ height: { $gt: 170 }}).pretty()
$ db.users.find({ weight: { $gte: 64 }}).pretty()
$ db.users.find({ weight: { $lt: 70 }}).pretty()
$ db.users.find({ weight: { $lte: 70 }}).pretty()
```

### 指定 Collection 更新 Document

> 屬性存在即更新，不存在即創建

> 根據頂級字段進行更新 (支持數組、嵌入文檔、文檔數組)

```shell
$ db.users.updateOne({}, { $set: { age: 23 } })
$ db.users.updateOne({ name: "Owen" }, { $set: { verify: false } })
$ db.users.updateOne({ verify: true, age: 19 }, { $set: { age: 40 }})
$ db.users.updateOne({ _id: ObjectId("6086b8106fddb5a5c5959574") }, { $set: { name: "Ian", verify: true }})

---

$ db.users.updateMany({}, { $set: { verify: false }})
$ db.users.updateMany({ age: 40 }, { $set: { name: "Owen" }})
$ db.users.updateMany({ age: 40 }, { $set: { online: true }})

---

$ db.users.update({ name: "Owen"}, { age: 22 }) # 不建議使用，參考 replaceOne

---

$ db.users.updateOne({ name: "Maria", age: 29 }, { $set: { isSporty: true }}, { upsert: true })
```

### 指定 Collection 刪除 Document

```shell
$ db.users.deleteOne({})
$ db.users.deleteOne({ _id: ObjectId("6086c1196fddb5a5c5959575") })
$ db.users.deleteOne({ age: 18, verify: false })

---

$ db.users.deleteMany({})
$ db.users.deleteMany({ age: 18, verify: false })
```

### 指定 Collection 取代 Document

```shell
$ db.users.replaceOne({ name: "Owen" }, {})
$ db.users.replaceOne({ _id: ObjectId("60870836596fc1c0e327e09f") }, { age: 23, name: "Owen" })
```

### 更新嵌套 Document

```shell
$ db.users.updateOne({ _id: ObjectId("60870836596fc1c0e327e09f") }, { $set: { "oauth.google.kind": ["aaa", "bbb"] }})
$ db.users.updateOne({ name: "Ian" }, { $set: { "oauth.google.kind.0": 30 }})
$ db.users.updateOne({ name: "Owen" }, { $set: { "hobbies.0.frequency": 6 }}) # 物件陣列內屬性存取方式

---

$ db.users.updateMany({}, { $set: { "oauth.google.kind.0": 48 }})
$ db.users.updateMany({}, { $set: { oauth: { google: { id: 123 }, github: { id: 456 }}}})
$ db.users.updateMany({}, { $set: { "oauth.google.displayName": "Ben" }})
```

### 存取嵌套 Document

> findOne 及大部分函式都是回傳 Document，與 find 回傳 cursor 不同，代表可直接進行存取

```shell
$ db.users.findOne({ name: "Eric" }).oauth.google
$ db.users.findOne({ name: "Ian" }).oauth.google.kind[0]

---

$ db.users.find({ "oauth.google.kind": ["aaa", "bbb"] }).pretty() # 匹配陣列 (完全符合陣列)
$ db.users.find({ "oauth.google.kind": { $all: ["bbb", "aaa"] }}).pretty() # 匹配陣列 (只在乎內容須都被包含)
$ db.users.find({ "oauth.google.kind": "ccc" }).pretty() # 匹配項目 (至少匹配一個項目)

---

$ db.users.find({ "oauth.google.kind.0": "aaa" }).pretty()
$ db.users.find({ "oauth.google.kind.0": { $gte: 30 }}).pretty()
$ db.users.find({ "oauth.google.kind": { $gte: 30 }}).pretty() # 匹配項目 (至少匹配一個項目)
$ db.users.find({ "hobbies.frequency": { $lte: 4 }}) # 物件陣列存取方式
```

### 關聯查詢

```shell
$ db.orders.insertMany([{ products: [1, 2, 3] }, { products: [4, 3, 1] }])
$ db.users.insertOne({ name: "Ian", orders: [ObjectId("60886133d0d873a28a88644f")]})
$ db.users.insertOne({ name: "Eric", orders: [ObjectId("60886133d0d873a28a886450")]})

---

$ db.users.aggregate({ $lookup: { from: "orders", localField: "orders", foreignField: "_id", as: "orderList" }}).pretty()
```

### Schema 驗證 (使用 createCollection 方法)

```shell
$ db.createCollection("posts", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title", "text", "tag", "creator", "comments"],
      properties: {
        title: {
          bsonType: "string",
          description: "must be a string and is required",
        },
        text: {
          bsonType: "string",
          description: "must be a string and is required",
        },
        tag: {
          bsonType: "array",
          description: "must be a array and is required",
          items: {
            bsonType: "string",
          },
        },
        creator: {
          bsonType: "objectId",
          description: "must be a objectId and is required",
        },
        comments: {
          bsonType: "array",
          description: "must be a array and is required",
          items: {
            bsonType: "object",
            required: ["text", "author"],
            properties: {
              text: {
                bsonType: "string",
                description: "must be a string and is required",
              },
              author: {
                bsonType: "objectId",
                description: "must be a objectId and is required",
              },
            },
          },
        },
      },
    },
  },
  validationAction: "warn",
})

---

$ db.posts.insertOne({
  title: "First Post",
  text: "money is power",
  tag: ["new", "tech"],
  creator: ObjectId("60891889606ebef65d76a898"),
  comments: [
    { text: "I like this post", author: ObjectId("60891889606ebef65d76a897") },
  ],
})
```

### Schema 驗證 (使用 runCommand 方法)

```shell
$ db.runCommand({
  collMod: "posts",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      additionalProperties: false,
      required: ["_id", "title", "text", "tag", "creator", "comments"],
      properties: {
        _id: {
          bsonType: "objectId",
        },
        title: {
          bsonType: "string",
          description: "must be a string and is required",
        },
        text: {
          bsonType: "string",
          description: "must be a string and is required",
        },
        tag: {
          bsonType: "array",
          description: "must be a array and is required",
          items: {
            bsonType: "string",
          },
        },
        creator: {
          bsonType: "objectId",
          description: "must be a objectId and is required",
        },
        comments: {
          bsonType: "array",
          description: "must be a array and is required",
          items: {
            bsonType: "object",
            required: ["text", "author"],
            properties: {
              text: {
                bsonType: "string",
                description: "must be a string and is required",
              },
              author: {
                bsonType: "objectId",
                description: "must be a objectId and is required",
              },
            },
          },
        },
      },
    },
  },
  validationAction: "error",
})

---

$ db.posts.insertOne({
  title: "First Post",
  text: "money is power",
  tag: ["new", "tech"],
  creator: ObjectId("60891889606ebef65d76a898"),
  comments: [
    { text: "I like this post", author: ObjectId("60891889606ebef65d76a897") },
  ],
})
```

---

### Write Concern

```shell
$ db.users.insertOne({ name: "Ian" }, { writeConcern: { w: 0 }})
$ db.users.insertOne({ name: "Eric" }, { writeConcern: { w: 1 }}) # default
$ db.users.insertOne({ name: "Jack" }, { writeConcern: { w: 1, j: false }}) # default
$ db.users.insertOne({ name: "Michael" }, { writeConcern: { w: 1, j: true }})
$ db.users.insertOne({ name: "Rebecca" }, { writeConcern: { w: 1, j: true, wtimeout: 200 }})
```

### 匯入資料

> 需額外下載 [MongoDB Database Tools](https://www.mongodb.com/try/download/database-tools)

```shell
$ mongoimport --file .\users.json --db shop --collection users --jsonArray --drop
```

```json
[
  {
    "name": "Ian",
    "age": 17,
    "skills": ["basketball", "football"],
    "site": {
      "href": "..."
    }
  },
  {
    "name": "Eric",
    "age": 19,
    "skills": ["baseball"],
    "site": {
      "href": "..."
    }
  }
]
```

---

### 比較運算符

```shell
$ db.movies.find({ runtime: { $eq: 60 }}) # 匹配等於指定值的值
$ db.movies.find({ runtime: { $ne: 60 }}) # 匹配不等於指定值的值
$ db.movies.find({ runtime: { $lt: 42 }}) # 匹配小於指定值的值
$ db.movies.find({ runtime: { $lte: 42 }}) # 匹配小於等於指定值的值
$ db.movies.find({ runtime: { $gt: 42 }}) # 匹配大於指定值的值
$ db.movies.find({ runtime: { $gte: 42 }}) # 匹配大於等於指定值的值
$ db.movies.find({ runtime: { $in: [30, 42] }}) # 匹配數組指定的任何值 (其次使用 $or)
$ db.movies.find({ runtime: { $nin: [30, 42] }}) # 不匹配數組指定的任何值 (其次使用 $or)

--- Array

$ db.movies.find({ genres: { $in: ["Drama", "Action"] }}) # 匹配數組指定的任何值 (如同 $or，不考慮順序)
$ db.movies.find({ genres: { $all: ["Drama", "Action"] }}) # 匹配數組指定的所有值 (如同 $and，不考慮順序)
```

### 邏輯運算符

```shell
$ db.movies.find({ $or: [{ "rating.average": { $lt: 5 }}, { "rating.average": { $gt: 9.3 }}] }) # 返回任一條件匹配 => (a || b)
$ db.movies.find({ $nor: [{ "rating.average": { $lt: 5 }}, { "rating.average": { $gt: 9.3 }}] }) # 返回所有條件均不匹配 => !(a || b)
$ db.movies.find({ $and: [{ "rating.average": { $gt: 9 }}, { genres: "Drama" }] }) # 返回全部條件匹配 => (a && b)
$ db.movies.find({ runtime: { $not: { $eq: 60 }}}) # 返回條件不匹配 (反轉表達式效果) => !a

--- Exception

$ db.movies.find({ "rating.average": { $gt: 9 }, genres: "Drama" }) # 同 $and
$ db.movies.find({ genres: "Horror", genres: "Drama" }) # 避免使用 (相同 key 會蓋掉前面的)
$ db.movies.find({ $and: [{ genres: "Drama" }, { genres: "Horror" }] }) # 使用 $and (推薦)
$ db.movies.find({ genres: { $all: ["Horror", "Drama"] }}) # 使用 $all (推薦)
```

### 元素運算符

```shell
$ db.users.find({ age: { $exists: true, $ne: null }}) # 確認元素是否存在 (empty 為不存在，null 為存在)
$ db.users.find({ phone: { $type: "string" } }) # 確認元素 BSON 型態 (接受別名、數字)
$ db.users.find({ phone: { $type: ["number", "string"] }}) # 確認元素 BSON 型態 (接受多個型態)
```

### 評估運算符

```shell
$ db.movies.find({ summary: { $regex: /musical/ }}) # 使用正規表達式搜尋
$ db.sales.find({ $expr: { $gt: ["$volume", "$target"] }}) # 使用聚合表達式

--- Advance

$ db.sales.find({ $expr: { $gt: [{ $cond: { if: { $gte: ["$volume", 190]}, then: { $subtract: ["$volume", 10] }, else: "$volume"}}, "$target"] }})
```

### 數組運算符

```shell
$ db.users.find({ hobbies: { $size: 2 }}) # 匹配陣列為指定長度 (無法處理大於或小於)
$ db.movies.find({ genre: { $all: ["action", "thriller"] }}) # 匹配陣列 (只在乎內容須都被包含)
$ db.users.find({ hobbies: { $elemMatch: { title: "Sports", frequency: { $gte: 6 }}}}) # 匹配同一項目的內容 (避免判斷到不同對象，參考下方)

--- Exception

$ db.users.find({ $and: [{ "hobbies.title": "Sports" }, { "hobbies.frequency": { $gte: 3 }}] }) # 判斷到不同對象
$ db.inventory.find({ dim_cm: { $gt: 15, $lt: 20 }}) # 各條件遍歷判斷 ([~] > 15 && [~] < 20)
$ db.inventory.find({ dim_cm: { $elemMatch: { $gt: 15, $lt: 20 }}}) # 所有條件遍歷判斷 ([1] > 15 && [1] < 20)
```

### 游標方法

> MongoDB Shell 只會返回前 20 個文檔 (性能考量，參考下方更改)

> print()、print(tojson())、printjson() 皆為 MongoDB Shell 方法

```shell
$ db.movies.find().count() # 返回文檔總數
$ db.movies.find().pretty() # 返回格式化結果
$ db.movies.find().toArray() # 返回數組化結果 (全部文檔)
$ db.movies.find().next() # 返回下一個文檔
$ db.movies.find().forEach(doc => printjson(doc)) # 遍歷文檔
$ db.movies.find().hasNext() # 確認下一個文檔是否存在 (通常用於變數)
$ db.movies.find().sort({ "rating.average": -1, runtime: -1 }) # 返回排序後的文檔 (1 代表升序，-1 代表降序)
$ db.movies.find().skip(2) # 返回跳過文檔數量後結果 (0 等同於沒有設置)
$ db.movies.find().limit(2) # 返回限制文檔數量後結果 (0 等效於沒有設置)
$ db.movies.find().batchSize(200) # 設置每批響應要返回的文檔數 (注意超時，可從 Wireshark 確認)
---

$ DBQuery.shellBatchSize = 30 # 設置 MongoDB Shell 批處理大小 (預設為 20)

---

$ db.users.find().sort({ index: -1, _id: -1 }).skip(5).limit(3) # 分頁實現
```

### 使用投影 (Projection)

> 除非 \_id 字段明確排除，否則皆返回 \_id 屬性

```shell
$ db.equipment.find({}, { name: 1, alias: true }) # 指定包含字段 (設置為 1 或 true)
$ db.equipment.find({}, { timing: 0, net: 0, logs: false, _id: 0 }) # 指定排除字段 (設置為 0 或 false)

--- Nested
$ db.equipment.find({}, { name: 1, "logs.member": 1 }) # 嵌入式文檔 (陣列)
$ db.users.find({}, { "name.first": 1 }) # 嵌入式文檔 (物件)

--- Exception

$ db.equipment.find({}, { name: 1, net: 0 }) # 錯誤！！ (參考下方例子)
$ db.equipment.find({}, { name: 1, _id: 0 }) # 包含與排除無法共用，_id 字段例外 (即設為排除)
$ db.equipment.find({}, { name: 0, _id: 1 }) # 包含與排除無法共用，_id 字段例外 (預設即為包含)
```

### 投影運算符

```shell
$ db.users.find({ array: "red" }, { "array.$": 1 }) # 投影數組中與查詢匹配的第一個元素 (查詢必須存在數組，無論自身或其他，返回空例外)
$ db.users.find({ array: { $all: ["red", "black"] }}, { "array.$": 1 }) # 同上 (元素可能不同)
$ db.users.find({ array: { $in: ["pink", "red"] }}, { "array.$": 1 }) # 同上 (元素可能不同)
$ db.equipment.find({ "logs.member": ObjectId("60a54ff1617882583771b983") }, { "logs.$": 1 }) # 嵌套

---

$ db.users.find({},{ array: { $elemMatch: {}}}) # 投影數組中與指定 $elemMatch 條件匹配的第一個元素 (此為皆不匹配)
$ db.users.find({},{ array: { $elemMatch: { $eq: "red" }}}) # 同上 (元素可能不同)
$ db.users.find({},{ array: { $elemMatch: { $in: ["pink", "red"] }}}) # 同上 (元素可能不同)
$ db.equipment.find({}, { logs: { $elemMatch: { member: ObjectId("60a54ff1617882583771b983") }}}) # 嵌套
$ db.equipment.find({}, { name: 1, logs: { $elemMatch: { member: ObjectId("60a54ff1617882583771b983"), startAt: { $gte: ISODate("2021-05-21T07:57:00.046Z") }}}}) # 嵌套

---

$ db.users.find({}, { array: { $slice: 2 }}) # 限制從數組投影的元素數量 (正數為從第一個開始計算)(包含或排除將影響其他字段)
$ db.users.find({}, { array: { $slice: -2 }}) # 同上 (負數為從最後一個開始計算)
$ db.users.find({}, { array: { $slice: [1, 2] }}) # 同上 (正數為從第一個開始計算，第二個參數填入數量，趨近於 ∞)
$ db.users.find({}, { array: { $slice: [-2, 1] }}) # 同上 (負數為從最後一個開始計算，第二個參數填入數量，趨近於 0)
```

---

### 字段更新運算符

> 均支持數組、嵌入文檔、文檔數組

```shell
$ db.users.updateOne({ name: "Chris" }, { $set: { name: "Sharon" }}) # 根據頂級字段更新其值 (不存在即創建)
$ db.users.updateMany({ "hobbies.title": "Sports" }, { $unset: { isSporty: "" } }) # 根據頂級字段刪除其值 (任意輸入值)(陣列項目變為 null)(不存在不做更動)
$ db.users.updateOne({ name: "Manuel" }, { $inc: { age: 1, qty: -2 }}) # 根據頂級字段增加其值 (接受正值和負值)(不存在即創建)
$ db.users.updateOne({ name: "Chris" }, { $min: { age: 33 }}) # 根據頂級字段變小其值 (僅當指定值小於現有字段值才更新其值)(不存在即創建)
$ db.users.updateOne({ name: "Chris" }, { $max: { age: 33, phone: 49884840 }}) # 根據頂級字段變大其值 (僅當指定值大於現有字段值才更新其值)(不存在即創建)
$ db.users.updateOne({ name: "Chris" }, { $mul: { age: 0.5 }}) # 根據頂級字段相乘其值 (將字段值相乘指定值)(不存在即創建，值為 0 並與乘數相同的數字型態)
$ db.users.updateMany({}, { $rename: { age: "totalAge" }}) # 根據頂級字段重命名名稱 (新名稱必須與現有名稱不同，否則報錯)(不存在不做更動)

---

$ db.users.updateOne({ name: "Manuel" }, { $inc: { age: -3 }, $unset: { num: "", count: "" }}) # 運算符可結合使用 (同字段會產生錯誤)
$ db.users.updateOne({ name: "Manuel" }, { $rename: { "oauth.github": "oauth.google" }}) # 嵌套重命名須包含完整路徑 (不支持文檔數組)

--- upsert

$ db.users.updateOne({ name: "Maria", age: 29 }, { $set: { isSporty: true }}, { upsert: true }) # 在過濾器無匹配時選擇插入文檔 (包含更新字段與唯一索引字段)(運算符以不存在處理)(預設為 false)
$ db.sports.updateMany({}, { $set: { title: "Football", requireTeam: true }}, { upsert: true }) # 同上 (同樣適用於 updateMany，在無匹配時插入一個新文檔)
$ db.sports.updateMany({ v: { $gt: 4 }}, { $set: { title: "Basketball" }}, { upsert: true }) # 同上 (v 不為唯一索引值，故單純插入 title)
$ db.users.updateOne({ name: "Maria" }, { $set: { name: "Sharon" }}, { upsert: true }) # 同上 (相同字段時，更新字段將覆蓋唯一索引字段)
```
