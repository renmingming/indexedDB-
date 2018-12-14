# indexedDB-本地存储
本地indexedDB以及dexie.js库

## indexedDB

1、创建本地数据库
  ```
    function openDB (myDB) {
        let version=version || 1; // 版本
        let request=window.indexedDB.open(myDB.name,myDB.version);  // 打开数据库（DB命，DB版本）
        request.onerror=function(e){ // 打开失败
            console.log(e.currentTarget.error.message);
        };
        request.onsuccess=function(e){ // 打开成功
            myDB.db=e.target.result; // 拿到数据库实例
        };
        request.onupgradeneeded=function(e){ // 新建数据库或者升级数据都会触发
            let db=e.target.result;
            if(!db.objectStoreNames.contains('students')){ // 是否有这个对象仓库/表
                let store = db.createObjectStore('students',{keyPath:"id", autoIncrement: true}); //创建对象仓库/表，主键为id，自增
                store.createIndex('groupIndex','group_id',{unique:false});  // 创建索引groupIndex, 对象属性为group_id, unique是否唯一
                store.createIndex('idIndex','id',{unique:true});
            }
            console.log('DB version changed to '+myDB.version);
        };
    }
  ```
  
  2、关闭数据库
  
  ```
    function closeDB(db){
        db.close();
    }
  ```
  
  3、删除数据库
  
  ```
    function deleteDB(name){
        indexedDB.deleteDatabase(name);
    }
  ```
  
  4、添加数据
  
  ```
    function addData(db,storeName, addData){ // 数据库实例，仓库/表名，添加的数据
        let transaction=db.transaction(storeName,'readwrite');  //新建事务：仓库/表名， 只读/读写
        let store=transaction.objectStore(storeName);  // 拿到IDBObjectStore对象

        for(let i=0;i<addData.length;i++){
            store.add(addData[i]); // 添加方法
        }
    }
  ```
  
  5、使用索引读取数据（唯一）
  
  ```
    function getDataByIndex(db,storeName, nameIndex, keyName){ // 数据库实例，仓库/表名，索引键，键值
        let transaction=db.transaction(storeName);
        let store=transaction.objectStore(storeName);
        let index = store.index(nameIndex);
        index.get(keyName).onsuccess=function(e){
            let student=e.target.result;
            console.log(student);
        }
    }
  ```
  
  6、游标与索引结合（多个）
  
  ```
    // 游标于index结合
    function getMultipleData(db,storeName, keyName, cb){ // 数据库实例，仓库/表名，索引键，键值, 返回结果集回调
        let transaction=db.transaction(storeName);
        let store=transaction.objectStore(storeName);
        let index = store.index("groupIndex");
        // 打开游标
        // IDBKeyRange.only获取对应索引值的结果集
        let request=index.openCursor(IDBKeyRange.only(keyName));
        let rowData = [];
        let temp = 0;
        request.onsuccess=function(e){
            let cursor=e.target.result;
            if(!cursor && cb) {
                cb({
                    error: 0,
                    data: rowData
                })
                return;
            }
            if(cursor){
                let student=cursor.value;
                rowData.push(student);
                cursor.continue(); // 会使游标下移，没有结果时返回undefault
            }
        }
    }
  ```

## dexie.js库

1、npm安装

```
  npm install dexie --save
```

2、引入依赖

```
  //es6 import 
  import Dexie from 'dexie'
```

3、使用

```
  const db = new Dexie('myDb'); // 数据库名为myDb
  
  db.version(1).stores({
    groupChats: `++id, group_id, chat_id`, // 新建打开groupChats表，并创建group_id,chat_id索引，主键id自增
    userChats: `++id, friend_id, chat_id`,
  })
  
```

4、查询数组展示

```
  // groupChats为表名，chat_id索引查询条件，equals等于，msgChat.id值，toArray结果数组集
  db.groupChats.where('chat_id').equals(msgChat.id).toArray() // 为promise，返回结果为异步
  以下所有返回都可以使用以下操作
  ---- 结果获取
    toArray(function(item) {
      console.log(item) // item 为数组
    })
    或
    toArray().then(item => {
      console.log(item) // 数组
    })
  --- 使用async和await获取到结果后在执行后续代码
    方法外使用async
    let groupChat = await db.groupChats.where('chat_id').equals(msgChat.id).toArray();
    grouopChat 为结果数组
    
    // 根据条件获取表中的最后一个
    let lastChat = await mydb.groupChats.where('group_id').equals(groupId).last();
    
    // 根据条件修改对应数据
    db.groupChats.where('chat_id').equals(msgChat.recall_chat_id).modify(msgchat);
    
    // 根据条件获取结果集，reverse先对结果集反序，limit()取结果集多少条数据，sortBy()根据字段属性排序
    db.groupChats.where('group_id').equals(Number(group_id)).reverse().limit(this.page*15).sortBy('created_at').then(() => {})
    
    // 获取表中的总条数
    db.groupChats.where('group_id').equals(Number(group_id)).count()；
    
```




