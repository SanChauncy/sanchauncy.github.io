<!--
 * @Descripttion: 
 * @version: 
 * @Author: liushibo
 * @Date: 2021-08-20 15:14:18
 * @LastEditors: liushibo
 * @LastEditTime: 2021-08-20 17:14:50
 * @FilePath: \blog\docs\guide.md
-->
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
### electron-vue使用sqlite3
安装splite3依赖:`npm install sqlite3 --save`
>新建index.js
```js
import {app,remote} from 'electron'
import fs from 'fs-extra'
import path from 'path'
import sqlite3 from 'sqlite3'
// 创建类
class DB {
  /**
   * Creates an instance of DB.
   * @memberof DB
   */
  constructor () {
    this.connect()
      .then(() => this.createTables())
      .catch((err) => {
        console.log('Constructor err: ', err)
      })
  }

  /**
   * Get shared instance of DB.
   *
   * @static
   * @returns The shared instance of DB.
   * @memberof DB
   */
  static sharedInstance () {
    this.instance = this.instance ? this.instance : new DB()
    return this.instance
  }

  /**
   * Connect to the database.
   *
   * @returns
   * @memberof DB
   */
  connect () {
    return new Promise((resolve, reject) => {
      this.db = new sqlite.Database(DB_PATH, (err) => {
        if (err) {
          console.log('Could not connect to db: ' + DB_PATH + ' err: ' + err)
          reject(err)
        } else {
          console.log('Connected to db: ', DB_PATH)
          resolve()
        }
      })
    })
  }

  /**
   * Create tables and insert default data.
   *
   * @returns
   * @memberof DB
   */
  createTables () {
    return new Promise((resolve, reject) => {
      const filePath = path.join(__dirname, '/sqlite.sql')
      fs.readFile(filePath, 'utf-8').then((sql, err) => {
        if (err) {
          console.log('Readfile err: ', err)
          reject(err)
        } else {
          this.exec(sql)
            .then(() => resolve())
            .catch((err) => reject(err))
        }
      })
    })
  }

  /**
   * Close the database.
   *
   * @memberof DB
   */
  close () {
    console.log('Close db')
    this.db.close()
  }

  /**
   * Runs the SQL query with the specified parameters and calls the callback afterwards.
   * It does not retrieve any result data.
   *
   * @param {*} sql
   * @param {*} [params=[]]
   * @returns
   * @memberof DB
   */
  run (sql, params = []) {
    console.log('Running sql: ' + sql + ' params: ' + params)
    return new Promise((resolve, reject) => {
      this.db.run(sql, params, (err) => {
        if (err) {
          console.log('Error running sql: ' + sql + ' params: ' + params)
          reject(err)
        } else {
          resolve()
        }
      })
    })
  }

  /**
   * Runs all SQL queries in the supplied string.
   * No result rows are retrieved.
   *
   * @param {*} sql
   * @returns
   * @memberof DB
   */
  exec (sql) {
    // console.log('Running sql: ' + sql)
    return new Promise((resolve, reject) => {
      this.db.exec(sql, (err) => {
        if (err) {
          console.log('Error running sql: ', sql)
          reject(err)
        } else {
          resolve()
        }
      })
    })
  }

  /**
   * Executes the statement and retrieves the first result row.
   *
   * @param {*} sql
   * @param {*} [params=[]]
   * @returns
   * @memberof DB
   */
  get (sql, params = []) {
    console.log('Running sql: ' + sql + ' params: ' + params)
    return new Promise((resolve, reject) => {
      this.db.get(sql, params, (err, data) => {
        if (err) {
          console.log('Error running sql: ' + sql + ' params: ' + params)
          reject(err)
        } else {
          resolve(data)
        }
      })
    })
  }

  /**
   * Executes the statement and calls the callback with all result rows.
   *
   * @param {*} sql
   * @param {*} [params=[]]
   * @returns
   * @memberof DB
   */
  all (sql, params = []) {
    console.log('Running sql: ' + sql + ' params: ' + params)
    return new Promise((resolve, reject) => {
      this.db.all(sql, params, (err, data) => {
        if (err) {
          console.log('Error running sql: ' + sql + ' params: ' + params)
          reject(err)
        } else {
          resolve(data)
        }
      })
    })
  }
}

export default DB
```
