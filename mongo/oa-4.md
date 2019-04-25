# 【运维】创建用户


```
db.createUser(
  {
    user: "test",
    pwd: "123456",
    roles: [ { role: "read", db: "data_center" },{ role: "readWrite", db: "data_center" },
             { role: "dbAdmin", db: "data_center" },{ role: "userAdmin", db: "data_center" } ],
    mechanisms : ["SCRAM-SHA-1"] 
  }
)
```