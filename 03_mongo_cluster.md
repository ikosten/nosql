
### Кластер MongoDB

1. Для построения кластера выбираем следующую схему с 4-мя ВМ:
```
mongo1=config1 + primary1 + slave2 + slave3
mongo2=config2 + slave1 + primary2 + slave3
mongo3=config3 + slave1 + slave2 + primary3 + mongos2
mongo4=mongos1
```
2. Создадим ВМ
```
gcloud compute --project=nosql-367017 instances create mongo1 --zone=us-central1-a --machine-type=e2-small --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1022317557791-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-ssd --boot-disk-device-name=mongoc --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
gcloud compute --project=nosql-367017 instances create mongo2 --zone=us-central1-a --machine-type=e2-small --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1022317557791-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-ssd --boot-disk-device-name=mongoc --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
gcloud compute --project=nosql-367017 instances create mongo3 --zone=us-central1-a --machine-type=e2-small --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1022317557791-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-ssd --boot-disk-device-name=mongoc --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
gcloud compute --project=nosql-367017 instances create mongo4 --zone=us-central1-a --machine-type=e2-small --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1022317557791-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-ssd --boot-disk-device-name=mongoc --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```
Установим MongoDB на каждой ВМ
```
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt update
sudo apt-get install -y mongodb-org
```
3. Создадим репликасет с конфигурацией шарда
```
kosten@mongo1:~$ sudo mkdir /home/mongo && sudo mkdir /home/mongo/dbc1 && sudo chmod 777 /home/mongo/dbc1
kosten@mongo1:~$ mongod --configsvr --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid --bind_ip_all
kosten@mongo2:~$ sudo mkdir /home/mongo && sudo mkdir /home/mongo/dbc2 && sudo chmod 777 /home/mongo/dbc2
kosten@mongo2:~$ mongod --configsvr --dbpath /home/mongo/dbc2 --port 27002 --replSet RScfg --fork --logpath /home/mongo/dbc2/dbc2.log --pidfilepath /home/mongo/dbc2/dbc2.pid --bind_ip_all
kosten@mongo3:~$ sudo mkdir /home/mongo && sudo mkdir /home/mongo/dbc3 && sudo chmod 777 /home/mongo/dbc3
kosten@mongo3:~$ mongod --configsvr --dbpath /home/mongo/dbc3 --port 27003 --replSet RScfg --fork --logpath /home/mongo/dbc3/dbc3.log --pidfilepath /home/mongo/dbc3/dbc3.pid --bind_ip_all
```
Инициализируем репликасет
```
kosten@mongo1:~$ mongosh --port 27001
test> rs.initiate({"_id" : "RScfg", configsvr: true, members : [{"_id" : 0, host : "mongo1:27001"},{"_id" : 1, host : "mongo2:27002"},{"_id" : 2, host : "mongo3:27003"}]});
{ ok: 1, lastCommittedOpTime: Timestamp({ t: 1669138369, i: 1 }) }
```
4. Создадим 3 репликасета для данных
```
kosten@mongo1:~$ sudo mkdir /home/mongo/{db11,db21,db31} && sudo chmod 777 /home/mongo/{db11,db21,db31}
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db11 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db11/db11.log --pidfilepath /home/mongo/db11/db11.pid --bind_ip_all
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db21 --port 27021 --replSet RS2 --fork --logpath /home/mongo/db21/db21.log --pidfilepath /home/mongo/db21/db21.pid --bind_ip_all
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db31 --port 27031 --replSet RS3 --fork --logpath /home/mongo/db31/db31.log --pidfilepath /home/mongo/db31/db31.pid --bind_ip_all
kosten@mongo2:~$ sudo mkdir /home/mongo/{db12,db22,db32} && sudo chmod 777 /home/mongo/{db12,db22,db32}
kosten@mongo2:~$ mongod --shardsvr --dbpath /home/mongo/db12 --port 27012 --replSet RS1 --fork --logpath /home/mongo/db12/db12.log --pidfilepath /home/mongo/db12/db12.pid --bind_ip_all
kosten@mongo2:~$ mongod --shardsvr --dbpath /home/mongo/db22 --port 27022 --replSet RS2 --fork --logpath /home/mongo/db22/db22.log --pidfilepath /home/mongo/db22/db22.pid --bind_ip_all
kosten@mongo2:~$ mongod --shardsvr --dbpath /home/mongo/db32 --port 27032 --replSet RS3 --fork --logpath /home/mongo/db32/db32.log --pidfilepath /home/mongo/db32/db32.pid --bind_ip_all
kosten@mongo3:~$ sudo mkdir /home/mongo/{db13,db23,db33} && sudo chmod 777 /home/mongo/{db13,db23,db33}
kosten@mongo3:~$ mongod --shardsvr --dbpath /home/mongo/db13 --port 27013 --replSet RS1 --fork --logpath /home/mongo/db13/db13.log --pidfilepath /home/mongo/db13/db13.pid --bind_ip_all
kosten@mongo3:~$ mongod --shardsvr --dbpath /home/mongo/db23 --port 27023 --replSet RS2 --fork --logpath /home/mongo/db23/db23.log --pidfilepath /home/mongo/db23/db23.pid --bind_ip_all
kosten@mongo3:~$ mongod --shardsvr --dbpath /home/mongo/db33 --port 27033 --replSet RS3 --fork --logpath /home/mongo/db33/db33.log --pidfilepath /home/mongo/db33/db33.pid --bind_ip_all
```
Инициализируем репликасеты
```
kosten@mongo1:~$ mongosh --port 27011
test> rs.initiate({"_id" : "RS1", members : [{"_id" : 0, priority : 3, host : "mongo1:27011"},{"_id" : 1, host : "mongo2:27012"},{"_id" : 2, host : "mongo3:27013"}]});
{
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1669139968, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1669139968, i: 1 })
}
kosten@mongo1:~$ mongosh --port 27021
test> rs.initiate({"_id" : "RS2", members : [{"_id" : 0, host : "mongo1:27021"},{"_id" : 1, priority : 3, host : "mongo2:27022"},{"_id" : 2, host : "mongo3:27023"}]});
{ ok: 1 }
kosten@mongo1:~$ mongosh --port 27031
test> rs.initiate({"_id" : "RS3", members : [{"_id" : 0, host : "mongo1:27031"},{"_id" : 1, host : "mongo2:27032"},{"_id" : 2, priority : 3, host : "mongo3:27033"}]});
{ ok: 1 }
```
5. Создадим шардированный кластер
```
kosten@mongo3:~$ sudo mkdir /home/mongo/dbs2 && sudo chmod 777 /home/mongo/dbs2
kosten@mongo3:~$ mongos --configdb RScfg/mongo1:27001,mongo2:27002,mongo3:27003 --port 27000 --fork --logpath /home/mongo/dbs2/dbs2.log --pidfilepath /home/mongo/dbs2/dbs2.pid
kosten@mongo4:~$ sudo mkdir /home/mongo && sudo mkdir /home/mongo/dbs1 && sudo chmod 777 /home/mongo/dbs1
kosten@mongo4:~$ mongos --configdb RScfg/mongo1:27001,mongo2:27002,mongo3:27003 --port 27000 --fork --logpath /home/mongo/dbs1/dbs1.log --pidfilepath /home/mongo/dbs1/dbs1.pid
```
Добавим шарды
```
kosten@mongo4:~$ mongosh --port 27000
[direct: mongos] test> sh.addShard("RS1/mongo1:27011,mongo2:27012,mongo3:27013")
{
  shardAdded: 'RS1',
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1669141187, i: 8 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1669141187, i: 8 })
}
[direct: mongos] test> sh.addShard("RS2/mongo1:27021,mongo2:27022,mongo3:27023")
{
  shardAdded: 'RS2',
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1669141205, i: 7 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1669141205, i: 7 })
}
[direct: mongos] test> sh.addShard("RS3/mongo1:27031,mongo2:27032,mongo3:27033")
{
  shardAdded: 'RS3',
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1669141221, i: 7 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1669141221, i: 7 })
}
```
Проверим статус
```
[direct: mongos] test> sh.status()
shardingVersion
{
  _id: 1,
  minCompatibleVersion: 5,
  currentVersion: 6,
  clusterId: ObjectId("637d07cc14c5113d6bf73c4c")
}
---
shards
[
  {
    _id: 'RS1',
    host: 'RS1/mongo1:27011,mongo2:27012,mongo3:27013',
    state: 1,
    topologyTime: Timestamp({ t: 1669141187, i: 5 })
  },
  {
    _id: 'RS2',
    host: 'RS2/mongo1:27021,mongo2:27022,mongo3:27023',
    state: 1,
    topologyTime: Timestamp({ t: 1669141205, i: 5 })
  },
  {
    _id: 'RS3',
    host: 'RS3/mongo1:27031,mongo2:27032,mongo3:27033',
    state: 1,
    topologyTime: Timestamp({ t: 1669141221, i: 5 })
  }
]
---
active mongoses
[ { '6.0.3': 2 } ]
---
autosplit
{ 'Currently enabled': 'yes' }
---
balancer
{
  'Currently enabled': 'yes',
  'Currently running': 'no',
  'Failed balancer rounds in last 5 attempts': 0,
  'Migration Results for the last 24 hours': 'No recent migrations'
}
---
databases
[
  {
    database: { _id: 'config', primary: 'config', partitioned: true },
    collections: {
      'config.system.sessions': {
        shardKey: { _id: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'RS1', nChunks: 1024 } ],
        chunks: [
          'too many chunks to print, use verbose if you want to force print'
        ],
        tags: []
      }
    }
  }
]
```
Устанавливаем размер чанка 1Мб
```
[direct: mongos] test> use config
switched to db config
[direct: mongos] config> db.settings.insert({ _id:"chunksize", value: 1})
DeprecationWarning: Collection.insert() is deprecated. Use insertOne, insertMany, or bulkWrite.
{ acknowledged: true, insertedIds: { '0': 'chunksize' } }
```

6. Скачиваем файл из с вопросами игры Jeopardy https://www.reddit.com/r/datasets/comments/1uyd0t/200000_jeopardy_questions_in_a_json_file/, и называем его jq.json
Загужаем файл в MongoDB c помощью утилиты mongoimport
```
kosten@mongo4:~$ mongoimport --port 27000 -d assignments -c jq --headerline --type csv jq.csv
2022-11-22T18:43:03.269+0000  connected to: mongodb://localhost:27000/
2022-11-22T18:43:06.270+0000  [######..................] assignments.jq 9.39MB/33.2MB (28.3%)
2022-11-22T18:43:09.269+0000  [#############...........] assignments.jq 18.9MB/33.2MB (56.8%)
2022-11-22T18:43:12.269+0000  [####################....] assignments.jq 28.8MB/33.2MB (86.6%)
2022-11-22T18:43:13.612+0000  [########################] assignments.jq 33.2MB/33.2MB (100.0%)
2022-11-22T18:43:13.613+0000  216930 document(s) imported successfully. 0 document(s) failed to import.
```
Выбираем шардировать по _id, т.к. остальные поля распределены неравномерно
```
[direct: mongos] assignments> sh.shardCollection("assignments.jq",{_id: 1})
{
  collectionsharded: 'assignments.jq',
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1669143332, i: 29 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1669143332, i: 25 })
}
```
Проверяем, что чанки разъехались по шардам.
Видим, что на RS1 17 чанков, на RS2 - 1, на RS3 - 17.

```
[direct: mongos] assignments> sh.status()
shardingVersion
{
  _id: 1,
  minCompatibleVersion: 5,
  currentVersion: 6,
  clusterId: ObjectId("637d07cc14c5113d6bf73c4c")
}
---
shards
[
  {
    _id: 'RS1',
    host: 'RS1/mongo1:27011,mongo2:27012,mongo3:27013',
    state: 1,
    topologyTime: Timestamp({ t: 1669141187, i: 5 })
  },
  {
    _id: 'RS2',
    host: 'RS2/mongo1:27021,mongo2:27022,mongo3:27023',
    state: 1,
    topologyTime: Timestamp({ t: 1669141205, i: 5 })
  },
  {
    _id: 'RS3',
    host: 'RS3/mongo1:27031,mongo2:27032,mongo3:27033',
    state: 1,
    topologyTime: Timestamp({ t: 1669141221, i: 5 })
  }
]
---
active mongoses
[ { '6.0.3': 2 } ]
---
autosplit
{ 'Currently enabled': 'yes' }
---
balancer
{
  'Currently enabled': 'yes',
  'Currently running': 'no',
  'Failed balancer rounds in last 5 attempts': 0,
  'Migration Results for the last 24 hours': { '34': 'Success' }
}
---
databases
[
  {
    database: {
      _id: 'assignments',
      primary: 'RS2',
      partitioned: false,
      version: {
        uuid: new UUID("5a47cebb-b855-40f8-9efb-cdcf4f3d8713"),
        timestamp: Timestamp({ t: 1669142582, i: 1 }),
        lastMod: 1
      }
    },
    collections: {
      'assignments.jq': {
        shardKey: { _id: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [
          { shard: 'RS1', nChunks: 17 },
          { shard: 'RS2', nChunks: 1 },
          { shard: 'RS3', nChunks: 17 }
        ],
        chunks: [
          'too many chunks to print, use verbose if you want to force print'
        ],
        tags: []
      }
    }
  },
  {
    database: { _id: 'config', primary: 'config', partitioned: true },
    collections: {
      'config.system.sessions': {
        shardKey: { _id: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'RS1', nChunks: 1024 } ],
        chunks: [
          'too many chunks to print, use verbose if you want to force print'
        ],
        tags: []
      }
    }
  }
]

```
7.  А теперь уроним машину mongo1
```
kosten@mongo1:~$ sudo shutdown now
```
Все круто - кластер продложает работать.
Загрузим еще раз данные
```
kosten@mongo4:~$ mongoimport --port 27000 -d assignments -c jq --headerline --type csv jq.csv
2022-11-22T19:08:34.084+0000  connected to: mongodb://localhost:27000/
2022-11-22T19:08:37.085+0000  [#####...................] assignments.jq 8.14MB/33.2MB (24.5%)
2022-11-22T19:08:40.084+0000  [###########.............] assignments.jq 15.8MB/33.2MB (47.6%)
2022-11-22T19:08:43.084+0000  [################........] assignments.jq 22.5MB/33.2MB (67.8%)
2022-11-22T19:08:46.085+0000  [####################....] assignments.jq 29.0MB/33.2MB (87.1%)
2022-11-22T19:08:47.919+0000  [########################] assignments.jq 33.2MB/33.2MB (100.0%)
2022-11-22T19:08:47.919+0000  216930 document(s) imported successfully. 0 document(s) failed to import.
```
Проверим статус шарда. Странно, но на RS2 по-прежнему 1 чанк. Все новые разъехались на RS1 и RS2.
```
[direct: mongos] test> sh.status()
shardingVersion
{
  _id: 1,
  minCompatibleVersion: 5,
  currentVersion: 6,
  clusterId: ObjectId("637d07cc14c5113d6bf73c4c")
}
---
shards
[
  {
    _id: 'RS1',
    host: 'RS1/mongo1:27011,mongo2:27012,mongo3:27013',
    state: 1,
    topologyTime: Timestamp({ t: 1669141187, i: 5 })
  },
  {
    _id: 'RS2',
    host: 'RS2/mongo1:27021,mongo2:27022,mongo3:27023',
    state: 1,
    topologyTime: Timestamp({ t: 1669141205, i: 5 })
  },
  {
    _id: 'RS3',
    host: 'RS3/mongo1:27031,mongo2:27032,mongo3:27033',
    state: 1,
    topologyTime: Timestamp({ t: 1669141221, i: 5 })
  }
]
---
active mongoses
[ { '6.0.3': 2 } ]
---
autosplit
{ 'Currently enabled': 'yes' }
---
balancer
{
  'Currently running': 'no',
  'Currently enabled': 'yes',
  'Failed balancer rounds in last 5 attempts': 0,
  'Migration Results for the last 24 hours': { '70': 'Success' }
}
---
databases
[
  {
    database: {
      _id: 'assignments',
      primary: 'RS2',
      partitioned: false,
      version: {
        uuid: new UUID("5a47cebb-b855-40f8-9efb-cdcf4f3d8713"),
        timestamp: Timestamp({ t: 1669142582, i: 1 }),
        lastMod: 1
      }
    },
    collections: {
      'assignments.jq': {
        shardKey: { _id: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [
          { shard: 'RS1', nChunks: 35 },
          { shard: 'RS2', nChunks: 1 },
          { shard: 'RS3', nChunks: 35 }
        ],
        chunks: [
          'too many chunks to print, use verbose if you want to force print'
        ],
        tags: []
      }
    }
  },
  {
    database: { _id: 'config', primary: 'config', partitioned: true },
    collections: {
      'config.system.sessions': {
        shardKey: { _id: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'RS1', nChunks: 1024 } ],
        chunks: [
          'too many chunks to print, use verbose if you want to force print'
        ],
        tags: []
      }
    }
  }
]
```
8.  Теперь делаем очень опасно - роняем машину mongo3
```
kosten@mongo3:~$ sudo shutdown now
```
К сожалению, наш кластер развалился...
```
[direct: mongos] test> sh.status()
MongoServerError: Encountered non-retryable error during query :: caused by :: Could not find host matching read preference { mode: "primary" } for set RScfg
```
Впрочем, этого и следовало ожидать. Впрочем, этого и следовало ожидать: для схемы PSS (у нас она используется для каждого отдельного репликасета) Fault Tolerance = 1
9. Пробуем поднять развалившийся кластер, запускаем mongo1
```
gcloud compute instances start mongo1
gcloud compute ssh mongo1
kosten@mongo1:~$ mongod --configsvr --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid --bind_ip_all
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db11 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db11/db11.log --pidfilepath /home/mongo/db11/db11.pid --bind_ip_all
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db21 --port 27021 --replSet RS2 --fork --logpath /home/mongo/db21/db21.log --pidfilepath /home/mongo/db21/db21.pid --bind_ip_all
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db31 --port 27031 --replSet RS3 --fork --logpath /home/mongo/db31/db31.log --pidfilepath /home/mongo/db31/db31.pid --bind_ip_all
``` 
Работоспособность кластера восстановлена
10. Добавим пользователей в БД: администратор и пользователь с правами только на чтение
```
[direct: mongos] test> use admin
switched to db admin
[direct: mongos] admin> db.createUser( { user: "root", pwd: "root", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
{
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1669146439, i: 2 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1669146439, i: 2 })
}
[direct: mongos] admin> db.createUser({      
...      user: "user1",      
...      pwd: "user1",      
...      roles: [{role: "read",db: "assignments"}] 
... })
{
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1669145695, i: 4 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1669145695, i: 4 })
}

```
11. Настроим аутентификацию в кластере. Сгенерим keyfile. При попытке стартовать инстанс с опцией keyFile получаем ошибку...
```
Unable to acquire security key
```
К сожалению, пока не удалось настроить аутентификацию для кластера.


Перенесем виртуалки в Европу europe-west1-c;, т.к. в США закончились ресурсы
for i in {1..4}; do gcloud compute instances set-disk-auto-delete mongo$i --zone us-central1-a --disk mongo$i --no-auto-delete; done;
for i in {1..4}; do gcloud compute disks snapshot mongo$i --snapshot-names backup-mongo$i --zone us-central1-a; done;
for i in {1..4}; do gcloud compute instances delete mongo$i --zone us-central1-a; done;
for i in {1..4}; do gcloud compute disks snapshot mongo$i --snapshot-names mongo$i-snapshot --zone us-central1-a; done;
for i in {1..4}; do gcloud compute disks delete mongo$i --zone us-central1-a; done;
for i in {1..4}; do gcloud compute disks create mongo$i --source-snapshot mongo$i-snapshot --zone europe-west1-c; done;
for i in {1..4}; do gcloud compute instances create mongo$i --zone=europe-west1-c --machine-type=e2-small --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1022317557791-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --disk name=mongo$i,boot=yes,mode=rw --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any; done;
for i in {1..4}; do gcloud compute snapshots delete backup-mongo$i mongo$i-snapshot; done;
gcloud compute project-info add-metadata --metadata google-compute-default-region=europe-west1,google-compute-default-zone=europe-west1-c

Стартуем виртуалки
for i in {1..4}; do gcloud compute instances start mongo$i --zone europe-west1-c; done;

-- создадим каталоги mongo-security
for i in {1..4}; do gcloud compute ssh mongo$i --zone europe-west1-c --command='sudo mkdir /home/mongo/mongo-security && sudo chmod 777 /home/mongo/mongo-security' & done;

-- генерируем кей файл на 1 инстансе
openssl rand -base64 756 > /home/mongo/mongo-security/keyfile
chmod 400 /home/mongo/mongo-security/keyfile

-- Поделимся ключом
scp kosten@34.78.18.194:/home/mongo/mongo-security/keyfile ~/Downloads/keyfile
scp ~/Downloads/keyfile kosten@35.187.63.18:/home/mongo/mongo-security/keyfile
scp ~/Downloads/keyfile kosten@35.240.108.161:/home/mongo/mongo-security/keyfile
scp ~/Downloads/keyfile kosten@34.77.185.93:/home/mongo/mongo-security/keyfile
for i in {2..4}; do gcloud compute ssh mongo$i --zone europe-west1-c --command='chmod 400 /home/mongo/mongo-security/keyfile' & done;

-- запускаем репликасеты с аутентификацией и ключом
kosten@mongo1:~$ mongod --configsvr --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db11 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db11/db11.log --pidfilepath /home/mongo/db11/db11.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db21 --port 27021 --replSet RS2 --fork --logpath /home/mongo/db21/db21.log --pidfilepath /home/mongo/db21/db21.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo1:~$ mongod --shardsvr --dbpath /home/mongo/db31 --port 27031 --replSet RS3 --fork --logpath /home/mongo/db31/db31.log --pidfilepath /home/mongo/db31/db31.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo2:~$ mongod --configsvr --dbpath /home/mongo/dbc2 --port 27002 --replSet RScfg --fork --logpath /home/mongo/dbc2/dbc2.log --pidfilepath /home/mongo/dbc2/dbc2.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo2:~$ mongod --shardsvr --dbpath /home/mongo/db12 --port 27012 --replSet RS1 --fork --logpath /home/mongo/db12/db12.log --pidfilepath /home/mongo/db12/db12.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo2:~$ mongod --shardsvr --dbpath /home/mongo/db22 --port 27022 --replSet RS2 --fork --logpath /home/mongo/db22/db22.log --pidfilepath /home/mongo/db22/db22.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo2:~$ mongod --shardsvr --dbpath /home/mongo/db32 --port 27032 --replSet RS3 --fork --logpath /home/mongo/db32/db32.log --pidfilepath /home/mongo/db32/db32.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo3:~$ mongod --configsvr --dbpath /home/mongo/dbc3 --port 27003 --replSet RScfg --fork --logpath /home/mongo/dbc3/dbc3.log --pidfilepath /home/mongo/dbc3/dbc3.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo3:~$ mongod --shardsvr --dbpath /home/mongo/db13 --port 27013 --replSet RS1 --fork --logpath /home/mongo/db13/db13.log --pidfilepath /home/mongo/db13/db13.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo3:~$ mongod --shardsvr --dbpath /home/mongo/db23 --port 27023 --replSet RS2 --fork --logpath /home/mongo/db23/db23.log --pidfilepath /home/mongo/db23/db23.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo3:~$ mongod --shardsvr --dbpath /home/mongo/db33 --port 27033 --replSet RS3 --fork --logpath /home/mongo/db33/db33.log --pidfilepath /home/mongo/db33/db33.pid --bind_ip_all --auth --keyFile /home/mongo/mongo-security/keyfile

-- запускаем шарды с ключом
kosten@mongo3:~$ mongos --configdb RScfg/mongo1:27001,mongo2:27002,mongo3:27003 --port 27000 --fork --logpath /home/mongo/dbs2/dbs2.log --pidfilepath /home/mongo/dbs2/dbs2.pid --keyFile /home/mongo/mongo-security/keyfile
kosten@mongo4:~$ mongos --configdb RScfg/mongo1:27001,mongo2:27002,mongo3:27003 --port 27000 --fork --logpath /home/mongo/dbs1/dbs1.log --pidfilepath /home/mongo/dbs1/dbs1.pid --keyFile /home/mongo/mongo-security/keyfile
