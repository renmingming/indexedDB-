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

## dexie.js库
