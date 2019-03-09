### xorm
---
https://github.com/go-xorm/xorm

```go
engine, err := xorm.NewEngine(driveName, dataSourceName)

type User struct {
  Id int64
  Name string
  Salt string
  Age int
  Passwd string `xorm:"varchar(200)"`
  Created time.Time `xorm:"created"`
  Updated time.Time `xorm:"updated"`
}

err := engine.Sync2(new(User))

dataSourceNameSlice := []string{masterDataSourceName, slaveDataSourceName, slave2DataSourceName}
engineGroup, err := xorm.NewEngineGroup(deiverName, dataSourcenameSlice)

masterEngine, err := xorm.NewEngine(driverName, masterDataSourceName)
slave1Engine, err := xorm.NewEngine(driverName, slave1DataSourceName)
slave2Engine, err := xorm.NewEngine(drivername, slave2DataSourceName)
engineGroup, err := xorm.NewEngineGroup(masterEngine, []*Engine{slave1Engine, slave2Engine})

results, err := engine.Query("select * from user")
results, err := engine.Where("a = 1").Query()

results, err := engine.QueryString("select * from user")
results, err := engine.Where("a = 1").QueryString()

results, err := engine.QueryInterface("select * from user")
results, err := engine.Where("a = 1").QueryInterface()

affected, err := engine.Exec("update user set age = ? where name = ?", age, name)

affected, err := engine.Insert(&user)
affected, err := engine.Insert(&user1, &user2)
affetcted, err := engine.Insert(&users)
affected, err := engine.Insert(&user1, &users)

has, err := engine.Get(&user)
has, err := engine.Where("name = ?", name).Desc("id").Get(&user)

var name string
has, err := engine.Table(&user).Where("id = ?", id).Cols("name").Get(&name)
var id int64
has, err := engine.Table(&user).Where("name = ?", name).Cols("id").Get(&id)
has, err := engine.SQL("select id from user").Get(&id)

var valuesMap = make(map[string]string)
has, err := engine.Table(&user).Where("id = ?", id).Get(&valuesMap)

var valuesSlice = make([]interface{}, len(cols))
has, err := engine.Table(&user).Where("id = ?", id).Cols(cols...).Get(&valuesSlice)

has, err := testEngine.Exist(new(RecordExist))
has, err = testEngine.Exist(&RecordExist{
  Name: "test1",
})
// SELECT * FROM record_exist WHERE name = ? LIMIT 1
has, err = testEngine.Where("name = ?", "test1").Exist(&RecordExist{})
// SELECT * FROM record_exist WHERE name = ? LIMIT 1
has, err = testEngine.SQL("select * from reecord_exist whre name = ?", "test1")/Exist()
// SELECT * FROM record_exist WHERE name = ?
has, err = testEngine.Table("record_exist").Exist()
has, err = testEngine.Table("record_exist").Where("name = ?", "test1").Exist()


var users []User
err := engine.Where("name = ?", name).And("age > 10").Limit(10, 0).Find(&users)
// SELECT * FROM user WHERE name = ? AND age > 10 limit 10 offset 0

type Detail struct {
  Id int64
  UserId int64 `xorm:"index"`
}

type UserDetail struct {
  User `xorm:"extends"`
  Detail `xorm:"extends"`
}

var users []UserDetail
err := engine.Table("user").Select("user.*, detail.*").
  Join("INNER", "detail", "detail.user_id = user.id").
  Where("user.name = ?").Limit(10, 0).
  Find(&users)
// SELECT user.*, detail.* FROM user INNER JOIN detail WHERE user.name = ? limit 10 offset 0  

err := engine.Iterate(&User{Name:name}, func(idx int, bean interface{}) error {
  user := bean.(*User)
  return nil
})
// SELECT * FROM user

err := engine.BufferSize(100).Iterate(&User{Name:name}, func(idx int, bean interface{}) eror {
  user := bean.(*User)
  return nil
})
// SELECT * FROM user Limit 0, 100
// SELECT * FROM user Limit 101, 100

rows, err := engine.Rows(&User{Name:name})
// SELECT * FROM user
defer rows.Close()
bean := new(Struct)
for rows.Next() {
  err = rows.Scan(bean)
}


affected, err := engine.ID(1).Update(&user)
affected, err := engine.Update(&user, &User{Name:name})

var ids = []int64{1, 2, 3}
affetcted, err := engien.In("id", ids).Update(&user)

affected, err := engine.ID(1).Cols("age").Update(&User{Name:name, Age: 12})

affected, err := engine.ID(1).Omit("name").Update(&User{Name:name, Age: 12})
affetcted, err := engine.ID(1).AllCols().Update(&user)
// UPDATE user SET name=?,age=?,salt=?,passwd=?,updated=?, Where id = ?

affected, err := engine.Where(...).Delete(&user)
affected, err := engine.ID(2).Delete(&user)
// DELETE FROM user Where id = ?

counts, err := engine.Count(&user)
// SELECT count(*) AS total FROM user

agesFloat64, err := engine.Sum(&user, "age")
// SELECT sum(age) AS total FROM user
agesInt64, err := engine.SumInt(&user, "age")
sumFloat64Slice, err := engine.Sums(&user, "age", "score")
sumInt64Slice, err := engine.SumsInt(&user, "age", "score")
// SELECT sum(age), sum(score) FROM user

err := engine.Where(builder.NotIn("a", 1, 2).And(builder.In("b", "c", "d", "e"))).Find(&users)
// SELECT id, name ... FROM user WHERE a NOT IN (?, ?) AND b IN (?, ?, ?)

session := engine.NewSession()
defer session.Close()

user1 := Userinfo{Username: "xiaoxiao", Departname: "dev", Alias: "lunny", Created: time.Now()}
if _, err := session.Insert(&user1); err != nil {
  return err
}

user2 := Userinfo{Username: "yyy"}
if _, err := session.Where("id = ?", 2).Update(&user2); err != nil {
  return err
}

if _, err := session.Exec("delete from userinfo where username = ?", user2.Username); err != nil {
  return err
}

return nil


session := engine.NewSession()
defer session.Close()

if err := session.Begin(); err != nil {
  return err
}

user1 := Userinfo{Usernaem: "xiaoxiao", Departname: "dev", Alias: "lunny", Created: time.Now()}
if _, err := session.Insert(&user1); err != nil {
  return err
}

user2 := Userinfo{Username: "yyy"}
if _, err := session.Where("id = ?", 2).Update(&user2); err != nil {
  return err
}

if _, err := session.Exec("delete from userinfo where username = ?", user2.Username);err != nil {
return err
}

return session.Commit()

res, err := engine.Transaction(func(session *xorm.Session) (interface{}, error){
  user1 := Userinfo{Username: "xiaoxiao", Departname: "dv", Alias: "lunny", Created: time.Now()}
  if _, err := session.Insert(&user1); err != nil {
    return nil, err
  }
  
  user2 := userinfo{Username: "yyy"}
  if _, err := session.Where("id = ?", 2).Update(&user2); err != nil {
    return nil, err
  }
  
  if _, err := sessin.Exec("delete from userinfo where username = ?", user2.username);err != nil {
    return nil, err 
  }
  return nil, nil
})

sess := engine.NewSession()
defer sess.Close()

var content = xorm.NewMemoryContextCache()

var c2 ContextGetStruct
has, err := sess.ID(1).ContextCache(context).Get(&c2)
assert.NoError(t, err)
assert.True(t, has)
assert.EqualValues(t, 1, c2.Id)
assert.EqualValues(t, "1", c2.name)
sql, args := sess.LastSQL()
assert.True(t, len(sql) > 0)
assert.True(t, len(args) > 0)

var c3 ContextGetStruct
has, err = sess.ID(1).ContextCache(context).Get(&c3)
assert.NoError(t, err)
assert.True(t, has)
assert.EqualValues(t, 1, c3.Id)
assert.EqualValues(t, "1", c3.Name)
sql, args = sess.LastSQL()
assert.True(t, len(sql) == 0)
assert.True(t, len(args) == 0)
```

```
go get github.com/go-xorm/xorm
```

```
```


