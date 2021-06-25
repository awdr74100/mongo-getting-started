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

$ db.users.insertMany([{ _id: 444, name: "Tidy" }, { _id: 222, name: "Owen" }], { ordered: false }) # 無序插入，失敗後仍會嘗試插入其他文檔 (預設為有序，失敗後終止插入後續文檔)
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

$ db.users.find({ "oauth.google.kind.0": "aaa" })
$ db.users.find({ "oauth.google.kind.0": { $gte: 30 }})
$ db.users.find({ "oauth.google.kind": { $gte: 30 }}) # 匹配項目 (至少匹配一個項目)
$ db.users.find({ "hobbies.frequency": { $lte: 4 }}) # 物件陣列存取方式

---

$ db.sports.find({ nums: [73, 85, 290] }) # 完全匹配陣列 (值、順序、長度)
$ db.sports.find({ nums: { $all: [73, 85] }}) # 部分匹配陣列 (值)
$ db.sports.find({ nums: { $eq: 370 }}) # 單條件查詢 (至少匹配一個項目)
$ db.sports.find({ nums: { $gte: 100, $lte: 130 }}) # 多條件查詢 (可能不同項目)
$ db.sports.find({ nums: { $elemMatch: { $gte: 100, $lte: 130 }}}) # 多條件查詢 (同一項目)
$ db.sports.find({ "nums.0": { $gte: 110 }}) # 指定索引查詢

$ db.sports.find({ colors: { color: "blue", v: 30 } }) # 完全匹配文檔陣列 (值、順序、長度)
$ db.sports.find({ "colors.v": { $eq: 30 } }) # 單條件查詢 (至少匹配一個項目)
$ db.sports.find({ "colors.v": { $gte: 25, $lte: 30 } }) # 多條件查詢 (可能不同項目)
$ db.sports.find({ "colors.v": { $lte: 30 }, "colors.color": "blue",}) # 多條件查詢 (可能不同項目)
$ db.sports.find({ colors: { $elemMatch: { v: { $gte: 25, $lte: 30 }}}}) # 多條件查詢 (同一項目)
$ db.sports.find({ "colors.0.v": { $lte: 5 }}) # 指定索引查詢
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

WiredTiger 使用 checkpoints (即將內存數據寫入磁盤) 來提供磁盤上數據的一致視圖，並允許 MongoDB 從上一個 checkpoint 進行恢復，但如果 MongoDB 在 checkpoint 之間意外退出，則需要使用日誌 ( 64 位系統下，預設已啟用) 來恢復上次 checkpoint 之後所發生的操作。客戶端發起的每個寫入操作都會在內存創建日誌。

- w：請求確認寫入操作已傳達到指定數量的 mongod 實例

  - 0：請求不確認，表示在發出寫入操作後立即返回
  - 1 (default)：請求確認，表示在發出寫入操作並實際寫入內存後返回，後續透過 `storage.syncPeriodSecs` 所設置間隔時間通過 `fsync` 將數據寫入磁盤

- j：請求確認寫入操作已寫入磁盤日誌

  - undefined (default)：當 `{ w: 1 }` 時，此值將設為 `false`
  - false：請求不確認，表示依照 `storage.journal.commitIntervalMs` 所設置間隔時間將內存日誌同步到磁盤
  - true：請求確認，表示將內存日誌主動同步到磁盤後返回

- wtimeout：指定寫入操作在限制時間內返回

  - undefined (default)：沒有限制
  - \<num>：限制在 \<num> ms 內返回，否則報錯

提示 (1)：日誌在不主動設置 `{ j: true }` 情況下，將根據 `storage.journal.commitIntervalMs` (預設 100 ms) 定期將內存日誌同步到磁盤。

提示 (2)：WiredTiger 在進行寫入操作時，實際上是將數據寫到內存上，接著透過 `storage.syncPeriodSecs` (預設 60 s) 定期通過 `fsync` 寫入到磁盤。

提示 (3)：為了保證資料持久性，建議一定要開啟日誌，這樣即使出現意外退出，也能從最近一次的 checkpoint 及日誌來恢復操作 (可使用 `{ j: true }` 將內存日誌立即同步)

```shell
$ db.users.insertOne({ name: "Ian" }, { writeConcern: { w: 0 }})
$ db.users.insertOne({ name: "Ian" }, { writeConcern: { w: 1 }})
$ db.users.insertOne({ name: "Ian" }, { writeConcern: { w: 1, j: true }})
$ db.users.insertOne({ name: "Ian" }, { writeConcern: { w: 1, j: true, wtimeout: 2000 }})
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
$ db.movies.find({ runtime: { $ne: 60 }}) # 匹配不等於指定值的值 (注意查詢數組的變化)
$ db.movies.find({ runtime: { $lt: 42 }}) # 匹配小於指定值的值
$ db.movies.find({ runtime: { $lte: 42 }}) # 匹配小於等於指定值的值
$ db.movies.find({ runtime: { $gt: 42 }}) # 匹配大於指定值的值
$ db.movies.find({ runtime: { $gte: 42 }}) # 匹配大於等於指定值的值
$ db.movies.find({ runtime: { $in: [30, 42] }}) # 匹配數組指定的任何值 (其次使用 $or)
$ db.movies.find({ runtime: { $nin: [30, 42] }}) # 不匹配數組指定的任何值 (其次使用 $or)(注意查詢數組的變化)

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
$ db.users.find({ hobbies: { $elemMatch: { title: "Sports", frequency: { $gte: 6 }}}}) # 匹配同一項目的內容 (避免判斷到不同對象，參考下方)(至少一個項目匹配所有條件)

--- Exception

$ db.users.find({ $and: [{ "hobbies.title": "Sports" }, { "hobbies.frequency": { $gte: 3 }}] }) # 判斷到不同對象
$ db.inventory.find({ dim_cm: { $gt: 15, $lt: 20 }}) # 各條件遍歷判斷 ([~] > 15 && [~] < 20)
$ db.inventory.find({ dim_cm: { $elemMatch: { $gt: 15, $lt: 20 }}}) # 所有條件遍歷判斷 ([1] > 15 && [1] < 20)

--- Trap

$ db.sports.find({ "colors.color": { $eq: "red" }}) # 單條件查詢 (數組任意值或文檔匹配即返回)
$ db.sports.find({ "colors.color": { $in: ["red", "pink"] }}) # 同上 (多個數值)
$ db.sports.find({ "colors.color": { $not: { $nin: ["red", "pink"] }}}) # 同上

$ db.sports.find({ "colors.color": { $ne: "red" }}) # 單條件查詢 (數組任意值或文檔匹配即不返回 -> 數組全部值或文檔皆不匹配即返回)
$ db.sports.find({ "colors.color": { $nin: ["red", "pink"] }}) # 同上 (多個數值)
$ db.sports.find({ "colors.color": { $not: { $in: ["red", "pink"] }}}) # 同上

$ db.sports.find({ "colors": { $elemMatch: { color: { $eq: "red" }}}}) # 與未使用 $elemMatch 相同 (限單條件查詢且未使用 $ne、$nin、$not)
$ db.sports.find({ "colors": { $elemMatch: { color: { $ne: "red" }}}}) # 與未使用 $elemMatch 不同 (數組任意值或文檔不匹配即返回)

$ db.test.find({ user_id: 1, imgs: { $elemMatch: { img_id: 1, "matches.img_id": { $nin: [3] }}}}) # 未使用 $elemMatch 處理 (結果不同，參考上方)
$ db.test.find({ user_id: 1, imgs: { $elemMatch: { img_id: 1, matches: { $elemMatch: { img_id: { $nin: [3] } }}}}}) # 使用 $elemMatch 處理 (結果不同，參考上方)
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
$ db.movies.find().sort({ "rating.average": -1, runtime: -1 }) # 返回排序後的文檔 (1 代表升序，-1 代表降序)(存在排序順序之分)(注意字段為數組時判斷，升序選擇項目間最小值、降序選擇項目間最大值，可嵌套選擇項目判斷字段)
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
$ db.users.find({ array: "red" }, { "array.$": 1 }) # 投影數組中與查詢匹配的第一個元素 (查詢必須存在數組，無論自身或其他，返回空例外)(每個查詢無法指定多個 $)(無法與 $elemMatch 同時使用)
$ db.users.find({ array: { $all: ["red", "black"] }}, { "array.$": 1 }) # 同上 (元素可能不同)
$ db.users.find({ array: { $in: ["pink", "red"] }}, { "array.$": 1 }) # 同上 (元素可能不同)
$ db.sports.find({ "colors.color": "red" }, { "colors.$": 1 }) # 嵌套
$ db.sports.find({ "colors.color": "red" }, { "colors.color.$": 1 }) # 嵌套 (指定字段)
$ db.sports.find({ nums: { $in: [24, 28] }, "colors.color": "blue" }, { "nums.$": 1, "colors.$": 1 }) # 錯誤 (每個查詢無法指定多個 $)

---

$ db.users.find({},{ array: { $elemMatch: {}}}) # 投影數組中與指定 $elemMatch 條件匹配的第一個元素 (假設為數組，此為不匹配；假設為文檔數組，此為匹配數組第一個元素)(單個文檔多個字段能使用)(無法與 $ 同時使用)
$ db.users.find({},{ array: { $elemMatch: { $eq: "red" }}}) # 同上 (元素可能不同)
$ db.users.find({},{ array: { $elemMatch: { $in: ["pink", "red"] }}}) # 同上 (元素可能不同)
$ db.equipment.find({}, { logs: { $elemMatch: { member: ObjectId("60a54ff1617882583771b983") }}}) # 嵌套
$ db.equipment.find({}, { name: 1, logs: { $elemMatch: { member: ObjectId("60a54ff1617882583771b983"), startAt: { $gte: ISODate("2021-05-21T07:57:00.046Z") }}}}) # 嵌套
$ db.sports.find({}, { nums: { $elemMatch: { $gte: 290 } }, colors: { $elemMatch: { color: "blue" }}}) # 多個字段使用
$ db.sports.find({ nums: { $in: [24, 28] }}, { "nums.$": 1, colors: { $elemMatch: { color: "blue" }}}) # 錯誤 (無法與 $ 同時使用)

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
$ db.users.updateMany({}, { $rename: { age: "totalAge" }}) # 根據頂級字段重命名名稱 (新名稱必須與現有名稱不同，否則報錯)(文檔數組不起作用)(不存在不做更動)
$ db.sports.updateOne({ name: "Chris" }, { $currentDate: { ts: { $type: "timestamp" }, a_iso: { $type: "date" }, b_iso: true }}) # 根據頂級字段更新其日期 (透過布爾或物件傳遞)(不存在即創建)

---

$ db.users.updateOne({ name: "Manuel" }, { $inc: { age: -3 }, $unset: { num: "", count: "" }}) # 運算符可結合使用 (同字段會產生錯誤)
$ db.users.updateOne({ name: "Manuel" }, { $rename: { "oauth.github": "oauth.google" }}) # 嵌套重命名須包含完整路徑 (不支持文檔數組)

--- array

$ db.sports.updateOne({ colors: { $elemMatch: { v: { $gte: 2 }}}}, { $set: { name: "Sharon", "colors.$.v": 1, "colors.$.color": "pink" }}) # 使用 $ 充當查詢匹配到的數組第一個項目 (查詢必須存在數組)(不可用於嵌套數組，意指數組內的數組)
$ db.sports.updateOne({ colors: { $elemMatch: { v: { $gte: 1 }}}}, { $set: { "colors.$": { color: "blue", v: 2, a: 1 } }}) # 同上 (覆蓋項目)
$ db.sports.updateOne({ colors: { $elemMatch: { v: { $gte: 1 }}}}, { $set: { "colors.$.a": 1 }}) # 同上 (新增字段)
$ db.sports.updateMany({ nums: { $elemMatch: { $gte: 6 }}}, { $set: { "nums.$": 100, isVerify: true }}) # 同上 (同樣適用於 updateMany)

$ db.sports.updateMany({}, { $inc: { "colors.$[].v": 1, "colors.$[].a": -3 }}) # 使用 $[] 充當查詢匹配到的所有數組項目 (可用於嵌套數組，意指數組內的數組)
$ db.sports.updateMany({ "colors.color": "black" }, { $inc: { "colors.$[].v": -2 }}) # 同上
$ db.sports.updateMany({ nums: { $elemMatch: { $gte: 255, $lte: 275 }}}, { $inc: { "nums.$[]": 5, limit: 12 }}) # 同上 (新增字段)
$ db.sports.updateMany({}, { $unset: { "colors.$[].a": "", count: "", limit: "" }}) # 同上 (消除數組文檔字段)

$ db.sports.updateMany({}, { $inc: { "nums.$[el]": 5 }}, { arrayFilters: [{ "el": { $gte: 200 }}] }) # 使用 $[<identifier>] 充當 arrayFilters 匹配到的所有數組項目 (必須準確指定一個 <identifier>)(可用於嵌套數組，意指數組內的數組)
$ db.sports.updateMany({ name: { $in: ["Sharon", "Ian"] }}, { $set: { "colors.$[item].verify": true }}, { arrayFilters: [{ "item.v": { $gte: 4 }}] }) # 同上 (操作文檔數組)
$ db.sports.updateMany({}, { $set: { "colors.$[el].verify": true }}, { arrayFilters: [{ "el.color": { $in: ["blue", "red"] }, "el.v": { $ne: 0 }}] }) # 指定複合條件 (<identifier> 只能存在於一個過濾器文檔)
$ db.students3.updateMany({}, { $inc: { "grades.$[el1].questions.$[]": 100 }}, { arrayFilters: [{ "el1.type": { $in: ["quiz", "exam"] }}] }) # 與 $[] 結合使用
$ db.students3.updateMany({}, { $inc: { "grades.$[el1].questions.$[el2]": 100 }}, { arrayFilters: [{ "el1.type": "quiz" }, { "el2": { $gte: 9 }}] }) # 指定多個 <identifier>

$ db.sports.updateMany({}, { $push: { nums: 2 }}) # 將值附加到數組 (字段非數組將報錯)(字段不存在將新增已套用修飾符並賦值的數組)
$ db.sports.updateMany({}, { $push: { nums: [2] } }) # 將數組附加到數組
$ db.sports.updateMany({}, { $push: { colors: { color: "green", v: 0 }}}) # 將文檔附加到數組
$ db.sports.updateMany({}, { $push: { colors: { $each: [{ color: "brown", v: 2 }] }}}) # 使用 $each 添加多個對象
$ db.sports.updateMany({}, { $push: { colors: { $each: [{ color: "gold", v: -10 }], $sort: { v: 1 }}}}) # 使用 $sort 升降排序 (必須與 $each 搭配使用，可設為 [])(1 表示升序、-1 表示降序)(存在排序順序之分)(陣列原始項目一併處理)
$ db.sports.updateMany({}, { $push: { colors: { $each: [{ color: "cyan", v: 4 }], $sort: { v: -1 }, $slice: 3 }}}) # 使用 $slice 切片 (必須與 $each 搭配使用，可設為 [])(0 表示清空數組、正數表示從開頭計算、負數表示從結尾計算)(陣列原始項目一併處理)
$ db.sports.updateMany({}, { $push: { colors: { $each: [{ color: "orange", v: 6 }], $position: 0 }}}) # 使用 $position 指定位置 (必須與 $each 搭配使用，可設為 [])(正數表示從開頭計算、負數表示從結尾計算，各自極限為結尾及開頭)(注意負數不含最後尾數計算)

$ db.sports.updateMany({}, { $pull: { nums: { $gte: 100, $lte: 200 }}}) # 從數組刪除所有匹配項目 (頂級文檔不需要使用 $elemMatch)
$ db.sports.updateMany({}, { $pull: { colors: { color: { $nin: ["red"] } ,v: { $gte: 10, $lte: 20 } }}}) # 從文檔數組刪除所有匹配項目 (頂級文檔不需要使用 $elemMatch)

$ db.sports.updateMany({}, { $pop: { nums: 1 }}) # 刪除數組最後一個項目 (空數組不會被修改)
$ db.sports.updateMany({}, { $pop: { colors: -1 }}) # 刪除數組第一個項目 (空數組不會被修改)

$ db.sports.updateMany({}, { $addToSet: { nums: 28 }}) # 將值附加到數組 (字段非數組將報錯)(字段不存在將新增已套用修飾符並賦值的數組)(只附加不完全匹配的數組對象)
$ db.sports.updateMany({}, { $addToSet: { nums: [28] }}) # 將數組附加到數組
$ db.sports.updateMany({}, { $addToSet: { colors: { color: "teal", v: 6 }}}) # 將文檔附加到數組
$ db.sports.updateMany({}, { $addToSet: { colors: { $each: [{ color: "yellow", v: 19 }] }}}) # 使用 $each 添加多個對象

--- upsert (Parameters)

$ db.users.updateOne({ name: "Maria", age: 29 }, { $set: { isSporty: true }}, { upsert: true }) # 在過濾器無匹配時選擇插入文檔 (包含更新字段與唯一索引字段)(更新運算符以不存在處理)(預設為 false)
$ db.sports.updateMany({}, { $set: { title: "Football", requireTeam: true }}, { upsert: true }) # 同上 (同樣適用於 updateMany，在無匹配時插入一個新文檔)
$ db.sports.updateMany({ v: { $gt: 4 }}, { $set: { title: "Basketball" }}, { upsert: true }) # 同上 (v 不為唯一索引值，故單純插入 title)
$ db.users.updateOne({ name: "Maria" }, { $set: { name: "Sharon" }}, { upsert: true }) # 同上 (相同字段時，更新字段將覆蓋唯一索引字段)
```

---

### 刪除操作

```shell
$ db.users.deleteOne({ name: "Chris" })
$ db.users.deleteMany({ totalAge: { $exists: false }, isSporty: true })

---

$ db.users.deleteMany({})
$ db.users.drop()

---

$ db.dropDatabase()
```

---

### 分析查詢

```shell
$ db.contacts.explain().help() # 查詢可用於分析的方法
$ db.contacts.explain().find().help() # 查詢可用於分析的查詢修飾符
$ db.contacts.explain().find({ "dob.age": { $gt: 60 }}) # 預設使用 queryPlanner 模式 (寫入操作實際不修改資料庫)
$ db.contacts.explain("executionStats").find({ "dob.age": { $gt: 60 }}) # 使用 executionStats 模式 (寫入操作實際不修改資料庫)
$ db.contacts.explain("allPlansExecution").find({ "dob.age": { $gt: 60 }}) # 使用 allPlansExecution 模式 (寫入操作實際不修改資料庫)
```

### 創建索引

```shell
$ db.contacts.getIndexes() # 取得集合索引
$ db.contacts.createIndex({ "dob.age": 1 }) # 建立索引 (1 表示升序、-1 表示降序)
$ db.contacts.dropIndex({ "dob.age": 1 }) # 刪除索引 (可透過索引名稱或文檔將其刪除)

---

$ db.contacts.createIndex({ "dob.age": 1 }) # 創建單字段索引 (索引鍵的升降序不重要，對於 sort 來說，MongoDB 可從任一方向遍歷索引)
$ db.contacts.createIndex({ "dob.age": 1, gender: 1 }) # 創建複合索引 (存在排序順序之分)(索引可由左至右組合使用，必須與第一個鍵搭配)(查詢鍵順序不影響排列順序)(索引鍵的升降序很重要，對於 sort 來說，必須與其匹配或逆匹配，且須按照它們在索引中出現的順序列出)

--- remark

db.contacts.createIndex({ x: 1, y: 1, z: 1 }) # 索引前綴為 { x: 1 }、{ x: 1, y: 1 } (可用於查詢及排序)

db.contacts.find({ x: 20 }) # 使用索引前綴 { x: 1 } 查詢
db.contacts.find({ x: 20, y: 40 }) # 使用索引前綴 { x: 1, y: 1 } 查詢
db.contacts.find({ x: 20, y: 40, z: 60 }) # 使用索引 { x: 1, y: 1, z: 1 } 查詢
db.contacts.find({ x: 20, z: 60 }) # 使用索引前綴 { x: 1 } 查詢

db.contacts.find().sort({ x: 1 }) # 使用索引前綴 { x: 1 } 排序
db.contacts.find().sort({ x: 1, y: 1 }) # 使用索引前綴 { x: 1, y: 1 } 排序
db.contacts.find().sort({ x: -1, y: -1, z: -1 }) # 使用索引 { x: 1, y: 1, z: 1 } 排序
db.contacts.find({ x: 20 }).sort({ y: 1, z: 1 }) # 使用索引 { x: 1, y: 1, z: 1 } 排序 (查詢須為相等條件)
db.contacts.find({ x: 20 }).sort({ y: 1 }) # 使用索引 { x: 1, y: 1 } 排序 (查詢須為相等條件)
db.contacts.find({ x: 20, y: 40 }).sort({ z: 1 }) # 使用索引 { x: 1, y: 1, z: 1 } 排序 (查詢須為相等條件)
db.contacts.find().sort({ x: 1, z: 1 }) # 未使用索引排序 (缺少 y，並非像查詢一樣可自動省略 z)
db.contacts.find({ y: 40 }).sort({ x: 1, z: 1 }) # 未使用索引排序 (x 應設為查詢，y 設為排序)
db.contacts.find({ x: 20 }).sort({ z: 1 }) # 未使用索引排序 (缺少 y 相等條件或 y 排序)
```
