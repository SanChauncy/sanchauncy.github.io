### 创建数据库

```javascript
this.db = new Database("foobar.db");
    try {
        this.db.exec('CREATE TABLE  Photo (id integer primary key AUTOINCREMENT,photo_path text,is_detect bool,detect_status int,desc text,time timestamp)');
        } catch (error) {
            console.log(error);
            if (error.message == "table Photo already exists") {
            console.log("Photo表已经存在");
        }
}
```
### 写入数据

```javascript
const insert = this.db.prepare("INSERT INTO Photo (photo_path, is_detect, detect_status, time) " + "VALUES (@photo_path, @is_detect, @detect_status, @time)");
const insertMany = this.db.transaction((cats) => {
        for (const cat of cats) {
          insert.run(cat);
        }
      });
insertMany(arr_photo_path);
```
### 查询数据
```javascript
const stmt = this.db.prepare("select * from Photo where is_detect=0 limit 0,1;");
let obj_photo = stmt.get();
```
### 修改数据
```javascript
const update = this.db.prepare("update Photo set is_detect=? where id=?;");
update.run(1, 1);
```
### 删除数据
```javascript
 this.db.exec("delete from Photo;update sqlite_sequence SET seq = 0 where name ='Photo';");
```
