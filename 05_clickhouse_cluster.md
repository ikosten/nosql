
### Кластер Clickhouse

1. Создадим 3 ВМ для кластера
```
for i in {1..3}; do gcloud compute --project=nosql-367017 instances create ch$i --zone=europe-west1-c --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1022317557791-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud --boot-disk-size=30GB --boot-disk-type=pd-ssd --boot-disk-device-name=ch$i --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any & done;
```
2. Установим и запустим ClickHouse
```
for i in {1..3}; do gcloud compute ssh ch$i --command='sudo apt-get install -y apt-transport-https ca-certificates dirmngr && sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754 && echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee /etc/apt/sources.list.d/clickhouse.list && sudo apt-get update && sudo apt-get install -y clickhouse-server clickhouse-client && sudo service clickhouse-server start' & done;
```
3. На каждом из серверов поправим файл /etc/clickhouse-server/config.xml, добавим
```
        <perftest_3shards_1replicas>
            <shard>
                <replica>
                    <host>ch1.europe-west1-c.c.nosql-367017.internal</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <replica>
                    <host>ch2.europe-west1-c.c.nosql-367017.internal</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <replica>
                    <host>ch3.europe-west1-c.c.nosql-367017.internal</host>
                    <port>9000</port>
                </replica>
            </shard>
        </perftest_3shards_1replicas>
```
и раскомментируем строку для сетевого доступа
```
<listen_host>::</listen_host>
```
3. Скачаем данные для тестовой БД на одну из машин
```
kosten@ch1:~$ curl https://datasets.clickhouse.com/hits/tsv/hits_v1.tsv.xz | unxz --threads=`nproc` > hits_v1.tsv
```
4. Создадим БД tutorial на каждой ноде
```
for i in {1..3}; do gcloud compute ssh ch$i --command='clickhouse-client --query "CREATE DATABASE IF NOT EXISTS tutorial"' & done;
```
5. Создадим таблицу hits_v1 на первой ноде ch1
```
ch1.europe-west1-c.c.nosql-367017.internal :) CREATE TABLE tutorial.hits_v1
                                                (
                                                    `WatchID` UInt64,
                                                    `JavaEnable` UInt8,
                                                    `Title` String,
                                                    `GoodEvent` Int16,
                                                    `EventTime` DateTime,
                                                    `EventDate` Date,
                                                    `CounterID` UInt32,
                                                    `ClientIP` UInt32,
                                                    `ClientIP6` FixedString(16),
                                                    `RegionID` UInt32,
                                                    `UserID` UInt64,
                                                    `CounterClass` Int8,
                                                    `OS` UInt8,
                                                    `UserAgent` UInt8,
                                                    `URL` String,
                                                    `Referer` String,
                                                    `URLDomain` String,
                                                    `RefererDomain` String,
                                                    `Refresh` UInt8,
                                                    `IsRobot` UInt8,
                                                    `RefererCategories` Array(UInt16),
                                                    `URLCategories` Array(UInt16),
                                                    `URLRegions` Array(UInt32),
                                                    `RefererRegions` Array(UInt32),
                                                    `ResolutionWidth` UInt16,
                                                    `ResolutionHeight` UInt16,
                                                    `ResolutionDepth` UInt8,
                                                    `FlashMajor` UInt8,
                                                    `FlashMinor` UInt8,
                                                    `FlashMinor2` String,
                                                    `NetMajor` UInt8,
                                                    `NetMinor` UInt8,
                                                    `UserAgentMajor` UInt16,
                                                    `UserAgentMinor` FixedString(2),
                                                    `CookieEnable` UInt8,
                                                    `JavascriptEnable` UInt8,
                                                    `IsMobile` UInt8,
                                                    `MobilePhone` UInt8,
                                                    `MobilePhoneModel` String,
                                                    `Params` String,
                                                    `IPNetworkID` UInt32,
                                                    `TraficSourceID` Int8,
                                                    `SearchEngineID` UInt16,
                                                    `SearchPhrase` String,
                                                    `AdvEngineID` UInt8,
                                                    `IsArtifical` UInt8,
                                                    `WindowClientWidth` UInt16,
                                                    `WindowClientHeight` UInt16,
                                                    `ClientTimeZone` Int16,
                                                    `ClientEventTime` DateTime,
                                                    `SilverlightVersion1` UInt8,
                                                    `SilverlightVersion2` UInt8,
                                                    `SilverlightVersion3` UInt32,
                                                    `SilverlightVersion4` UInt16,
                                                    `PageCharset` String,
                                                    `CodeVersion` UInt32,
                                                    `IsLink` UInt8,
                                                    `IsDownload` UInt8,
                                                    `IsNotBounce` UInt8,
                                                    `FUniqID` UInt64,
                                                    `HID` UInt32,
                                                    `IsOldCounter` UInt8,
                                                    `IsEvent` UInt8,
                                                    `IsParameter` UInt8,
                                                    `DontCountHits` UInt8,
                                                    `WithHash` UInt8,
                                                    `HitColor` FixedString(1),
                                                    `UTCEventTime` DateTime,
                                                    `Age` UInt8,
                                                    `Sex` UInt8,
                                                    `Income` UInt8,
                                                    `Interests` UInt16,
                                                    `Robotness` UInt8,
                                                    `GeneralInterests` Array(UInt16),
                                                    `RemoteIP` UInt32,
                                                    `RemoteIP6` FixedString(16),
                                                    `WindowName` Int32,
                                                    `OpenerName` Int32,
                                                    `HistoryLength` Int16,
                                                    `BrowserLanguage` FixedString(2),
                                                    `BrowserCountry` FixedString(2),
                                                    `SocialNetwork` String,
                                                    `SocialAction` String,
                                                    `HTTPError` UInt16,
                                                    `SendTiming` Int32,
                                                    `DNSTiming` Int32,
                                                    `ConnectTiming` Int32,
                                                    `ResponseStartTiming` Int32,
                                                    `ResponseEndTiming` Int32,
                                                    `FetchTiming` Int32,
                                                    `RedirectTiming` Int32,
                                                    `DOMInteractiveTiming` Int32,
                                                    `DOMContentLoadedTiming` Int32,
                                                    `DOMCompleteTiming` Int32,
                                                    `LoadEventStartTiming` Int32,
                                                    `LoadEventEndTiming` Int32,
                                                    `NSToDOMContentLoadedTiming` Int32,
                                                    `FirstPaintTiming` Int32,
                                                    `RedirectCount` Int8,
                                                    `SocialSourceNetworkID` UInt8,
                                                    `SocialSourcePage` String,
                                                    `ParamPrice` Int64,
                                                    `ParamOrderID` String,
                                                    `ParamCurrency` FixedString(3),
                                                    `ParamCurrencyID` UInt16,
                                                    `GoalsReached` Array(UInt32),
                                                    `OpenstatServiceName` String,
                                                    `OpenstatCampaignID` String,
                                                    `OpenstatAdID` String,
                                                    `OpenstatSourceID` String,
                                                    `UTMSource` String,
                                                    `UTMMedium` String,
                                                    `UTMCampaign` String,
                                                    `UTMContent` String,
                                                    `UTMTerm` String,
                                                    `FromTag` String,
                                                    `HasGCLID` UInt8,
                                                    `RefererHash` UInt64,
                                                    `URLHash` UInt64,
                                                    `CLID` UInt32,
                                                    `YCLID` UInt64,
                                                    `ShareService` String,
                                                    `ShareURL` String,
                                                    `ShareTitle` String,
                                                    `ParsedParams` Nested(
                                                        Key1 String,
                                                        Key2 String,
                                                        Key3 String,
                                                        Key4 String,
                                                        Key5 String,
                                                        ValueDouble Float64),
                                                    `IslandID` FixedString(16),
                                                    `RequestNum` UInt32,
                                                    `RequestTry` UInt8
                                                )
                                                ENGINE = MergeTree()
                                                PARTITION BY toYYYYMM(EventDate)
                                                ORDER BY (CounterID, EventDate, intHash32(UserID))
                                                SAMPLE BY intHash32(UserID);
```
и загрузим в нее данные из файла
```
kosten@ch1:~$ clickhouse-client --query "INSERT INTO tutorial.hits_v1 FORMAT TSV" --max_insert_block_size=100000 < hits_v1.tsv
```
6. На каждой ноде создадим таблицу hits_local с той же структурой
```
ch1.europe-west1-c.c.nosql-367017.internal :) CREATE TABLE tutorial.hits_local
                                                (
                                                    `WatchID` UInt64,
                                                    `JavaEnable` UInt8,
                                                    `Title` String,
                                                    `GoodEvent` Int16,
                                                    `EventTime` DateTime,
                                                    `EventDate` Date,
                                                    `CounterID` UInt32,
                                                    `ClientIP` UInt32,
                                                    `ClientIP6` FixedString(16),
                                                    `RegionID` UInt32,
                                                    `UserID` UInt64,
                                                    `CounterClass` Int8,
                                                    `OS` UInt8,
                                                    `UserAgent` UInt8,
                                                    `URL` String,
                                                    `Referer` String,
                                                    `URLDomain` String,
                                                    `RefererDomain` String,
                                                    `Refresh` UInt8,
                                                    `IsRobot` UInt8,
                                                    `RefererCategories` Array(UInt16),
                                                    `URLCategories` Array(UInt16),
                                                    `URLRegions` Array(UInt32),
                                                    `RefererRegions` Array(UInt32),
                                                    `ResolutionWidth` UInt16,
                                                    `ResolutionHeight` UInt16,
                                                    `ResolutionDepth` UInt8,
                                                    `FlashMajor` UInt8,
                                                    `FlashMinor` UInt8,
                                                    `FlashMinor2` String,
                                                    `NetMajor` UInt8,
                                                    `NetMinor` UInt8,
                                                    `UserAgentMajor` UInt16,
                                                    `UserAgentMinor` FixedString(2),
                                                    `CookieEnable` UInt8,
                                                    `JavascriptEnable` UInt8,
                                                    `IsMobile` UInt8,
                                                    `MobilePhone` UInt8,
                                                    `MobilePhoneModel` String,
                                                    `Params` String,
                                                    `IPNetworkID` UInt32,
                                                    `TraficSourceID` Int8,
                                                    `SearchEngineID` UInt16,
                                                    `SearchPhrase` String,
                                                    `AdvEngineID` UInt8,
                                                    `IsArtifical` UInt8,
                                                    `WindowClientWidth` UInt16,
                                                    `WindowClientHeight` UInt16,
                                                    `ClientTimeZone` Int16,
                                                    `ClientEventTime` DateTime,
                                                    `SilverlightVersion1` UInt8,
                                                    `SilverlightVersion2` UInt8,
                                                    `SilverlightVersion3` UInt32,
                                                    `SilverlightVersion4` UInt16,
                                                    `PageCharset` String,
                                                    `CodeVersion` UInt32,
                                                    `IsLink` UInt8,
                                                    `IsDownload` UInt8,
                                                    `IsNotBounce` UInt8,
                                                    `FUniqID` UInt64,
                                                    `HID` UInt32,
                                                    `IsOldCounter` UInt8,
                                                    `IsEvent` UInt8,
                                                    `IsParameter` UInt8,
                                                    `DontCountHits` UInt8,
                                                    `WithHash` UInt8,
                                                    `HitColor` FixedString(1),
                                                    `UTCEventTime` DateTime,
                                                    `Age` UInt8,
                                                    `Sex` UInt8,
                                                    `Income` UInt8,
                                                    `Interests` UInt16,
                                                    `Robotness` UInt8,
                                                    `GeneralInterests` Array(UInt16),
                                                    `RemoteIP` UInt32,
                                                    `RemoteIP6` FixedString(16),
                                                    `WindowName` Int32,
                                                    `OpenerName` Int32,
                                                    `HistoryLength` Int16,
                                                    `BrowserLanguage` FixedString(2),
                                                    `BrowserCountry` FixedString(2),
                                                    `SocialNetwork` String,
                                                    `SocialAction` String,
                                                    `HTTPError` UInt16,
                                                    `SendTiming` Int32,
                                                    `DNSTiming` Int32,
                                                    `ConnectTiming` Int32,
                                                    `ResponseStartTiming` Int32,
                                                    `ResponseEndTiming` Int32,
                                                    `FetchTiming` Int32,
                                                    `RedirectTiming` Int32,
                                                    `DOMInteractiveTiming` Int32,
                                                    `DOMContentLoadedTiming` Int32,
                                                    `DOMCompleteTiming` Int32,
                                                    `LoadEventStartTiming` Int32,
                                                    `LoadEventEndTiming` Int32,
                                                    `NSToDOMContentLoadedTiming` Int32,
                                                    `FirstPaintTiming` Int32,
                                                    `RedirectCount` Int8,
                                                    `SocialSourceNetworkID` UInt8,
                                                    `SocialSourcePage` String,
                                                    `ParamPrice` Int64,
                                                    `ParamOrderID` String,
                                                    `ParamCurrency` FixedString(3),
                                                    `ParamCurrencyID` UInt16,
                                                    `GoalsReached` Array(UInt32),
                                                    `OpenstatServiceName` String,
                                                    `OpenstatCampaignID` String,
                                                    `OpenstatAdID` String,
                                                    `OpenstatSourceID` String,
                                                    `UTMSource` String,
                                                    `UTMMedium` String,
                                                    `UTMCampaign` String,
                                                    `UTMContent` String,
                                                    `UTMTerm` String,
                                                    `FromTag` String,
                                                    `HasGCLID` UInt8,
                                                    `RefererHash` UInt64,
                                                    `URLHash` UInt64,
                                                    `CLID` UInt32,
                                                    `YCLID` UInt64,
                                                    `ShareService` String,
                                                    `ShareURL` String,
                                                    `ShareTitle` String,
                                                    `ParsedParams` Nested(
                                                        Key1 String,
                                                        Key2 String,
                                                        Key3 String,
                                                        Key4 String,
                                                        Key5 String,
                                                        ValueDouble Float64),
                                                    `IslandID` FixedString(16),
                                                    `RequestNum` UInt32,
                                                    `RequestTry` UInt8
                                                )
                                                ENGINE = MergeTree()
                                                PARTITION BY toYYYYMM(EventDate)
                                                ORDER BY (CounterID, EventDate, intHash32(UserID))
                                                SAMPLE BY intHash32(UserID);
```
7. Создадим распределенную таблицу на каждой ноде
```
for i in {1..3}; do gcloud compute ssh ch$i --command='clickhouse-client --query "CREATE TABLE tutorial.hits_all AS tutorial.hits_local ENGINE = Distributed(perftest_3shards_1replicas, tutorial, hits_local, rand());"' & done;
```
8. Наполним распределенную таблицу данными из таблицы hits_v1
```
kosten@ch1:~$ clickhouse-client 
ClickHouse client version 22.11.2.30 (official build).
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 22.11.2 revision 54460.

Warnings:
 * Maximum number of threads is lower than 30000. There could be problems with handling a lot of simultaneous queries.

ch1.europe-west1-c.c.nosql-367017.internal :) INSERT INTO tutorial.hits_all SELECT * FROM tutorial.hits_v1;

INSERT INTO tutorial.hits_all SELECT *
FROM tutorial.hits_v1

Query id: 5a17803d-4b81-4912-b2ff-023aa389b2df

Ok.

0 rows in set. Elapsed: 50.895 sec. Processed 8.87 million rows, 8.46 GB (174.36 thousand rows/s., 166.23 MB/s.)
```
9. Выполним аналитический запрос к локальной таблице с количеством роботов с группировкой по домену
```
ch1.europe-west1-c.c.nosql-367017.internal :)                                             SELECT
                                                                                               URLDomain AS URL,
                                                                                               sum(IsRobot) AS CntRobots,
                                                                                               bar(CntRobots, 0, 100000, 80)
                                                                                            FROM tutorial.hits_v1
                                                                                           WHERE EventDate BETWEEN '2014-03-23' AND '2014-03-30'
                                                                                           GROUP BY URL
                                                                                           ORDER BY CntRobots DESC
                                                                                           LIMIT 10;

SELECT
    URLDomain AS URL,
    sum(IsRobot) AS CntRobots,
    bar(CntRobots, 0, 100000, 80)
FROM tutorial.hits_v1
WHERE (EventDate >= '2014-03-23') AND (EventDate <= '2014-03-30')
GROUP BY URL
ORDER BY CntRobots DESC
LIMIT 10

Query id: c5759bf9-6244-4ebc-ae38-9aee4673221e

┌─URL──────────────────────────────┬─CntRobots─┬─bar(sum(IsRobot), 0, 100000, 80)─┐
│ cars2.auto.ru                    │     25760 │ ████████████████████▌            │
│ rulsma.deti.mail                 │      2498 │ █▊                               │
│ kiev.ko.slands.ruvr.russic-bazar │      2380 │ █▊                               │
│ piloral.auto                     │      1312 │ █                                │
│ yandex.ru.livemaster             │      1110 │ ▊                                │
│ maxserialu.net                   │       982 │ ▋                                │
│ ivi.ru.com                       │       956 │ ▋                                │
│ aeroflot.ru.msn                  │       792 │ ▋                                │
│ kino-rf.ru                       │       776 │ ▌                                │
│ yandex.ru.msn.com                │       733 │ ▌                                │
└──────────────────────────────────┴───────────┴──────────────────────────────────┘

10 rows in set. Elapsed: 0.131 sec. Processed 6.51 million rows, 172.16 MB (49.77 million rows/s., 1.32 GB/s.)
```
10. Повторим тот же запрос, но к распределенной таблице
```
cch1.europe-west1-c.c.nosql-367017.internal :) SELECT
                                                 URLDomain AS URL,
                                                 sum(IsRobot) AS CntRobots,
                                                 bar(CntRobots, 0, 100000, 80)
                                              FROM tutorial.hits_all
                                              WHERE EventDate BETWEEN '2014-03-23' AND '2014-03-30'
                                              GROUP BY URL
                                              ORDER BY CntRobots DESC
                                              LIMIT 10;

SELECT
    URLDomain AS URL,
    sum(IsRobot) AS CntRobots,
    bar(CntRobots, 0, 100000, 80)
FROM tutorial.hits_all
WHERE (EventDate >= '2014-03-23') AND (EventDate <= '2014-03-30')
GROUP BY URL
ORDER BY CntRobots DESC
LIMIT 10

Query id: ec4ac463-f17a-4b52-8cc4-033c81d4f807

┌─URL──────────────────────────────┬─CntRobots─┬─bar(sum(IsRobot), 0, 100000, 80)─┐
│ cars2.auto.ru                    │     25760 │ ████████████████████▌            │
│ rulsma.deti.mail                 │      2498 │ █▊                               │
│ kiev.ko.slands.ruvr.russic-bazar │      2380 │ █▊                               │
│ piloral.auto                     │      1312 │ █                                │
│ yandex.ru.livemaster             │      1110 │ ▊                                │
│ maxserialu.net                   │       982 │ ▋                                │
│ ivi.ru.com                       │       956 │ ▋                                │
│ aeroflot.ru.msn                  │       792 │ ▋                                │
│ kino-rf.ru                       │       776 │ ▌                                │
│ yandex.ru.msn.com                │       733 │ ▌                                │
└──────────────────────────────────┴───────────┴──────────────────────────────────┘

10 rows in set. Elapsed: 0.103 sec. Processed 7.26 million rows, 201.65 MB (70.65 million rows/s., 1.96 GB/s.)
```
Распределенная таблица читается быстрее, но разница в скорости не такая большая - порядка 20-30%.

