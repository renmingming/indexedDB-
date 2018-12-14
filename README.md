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
