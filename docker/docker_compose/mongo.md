```yaml
version: "3.8"
services:
  mongodb:
    image: mongo:6.0.2
    container_name: mongodb
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=root
  # mongo-express:
  #   image: mongo-express
  #   container_name: mongo-express
  #   ports:
  #     - 8081:8081
  #   environment:
  #     - ME_CONFIG_MONGODB_ADMINUSERNAME=root
  #     - ME_CONFIG_MONGODB_ADMINPASSWORD=root
  #     - ME_CONFIG_MONGODB_SERVER=mongodb
```

使用客户端的话就用 mongodb compass。mongo-express 只是个网页端，功能较少，没有用。

默认超级管理员是 root:root。可以给其他数据库添加用户。比如添加了 `mt` 数据库。然后给 mt 数据库添加 test1 用户。

```shell
> use mt
'switched to db mt'
> show users
[ { _id: 'mt.test',
    userId: UUID("28ee4b13-e590-4acb-bbcd-3d9b0dffabac"),
    user: 'test',
    db: 'mt',
    roles: [ { role: 'readWrite', db: 'mt' } ],
    mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ] } ]
> db.createUser( { user : "test1", pwd : "test", roles: [ { role : "readWrite", db : "mt" } ] } )
{ ok: 1 }
> show users
[ { _id: 'mt.test',
    userId: UUID("28ee4b13-e590-4acb-bbcd-3d9b0dffabac"),
    user: 'test',
    db: 'mt',
    roles: [ { role: 'readWrite', db: 'mt' } ],
    mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ] },
  { _id: 'mt.test1',
    userId: UUID("8ce9275f-3fe6-426e-8d46-b33700c524fb"),
    user: 'test1',
    db: 'mt',
    roles: [ { role: 'readWrite', db: 'mt' } ],
    mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ] } ]
> db.dropUser('test1')
{ ok: 1 }

```