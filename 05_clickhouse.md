
### Clickhouse

1. Создадим ВМ
```
gcloud compute --project=nosql-367017 instances create ch --zone=europe-west1-c --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1022317557791-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/cloud-platform --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud --boot-disk-size=30GB --boot-disk-type=pd-ssd --boot-disk-device-name=ch --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

2. Установим и запустим ClickHouse
```
kosten@ch:~$ sudo apt-get install -y apt-transport-https ca-certificates dirmngr && sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754 && echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee /etc/apt/sources.list.d/clickhouse.list && sudo apt-get update && sudo apt-get install -y clickhouse-server clickhouse-client && sudo service clickhouse-server start
```

3. Скачаем данные для тестовой БД
```
kosten@ch:~$ curl https://datasets.clickhouse.com/hits/tsv/hits_v1.tsv.xz | unxz --threads=`nproc` > hits_v1.tsv && curl https://datasets.clickhouse.com/visits/tsv/visits_v1.tsv.xz | unxz --threads=`nproc` > visits_v1.tsv
```

4. Создадим БД 
```
kosten@ch:~$ clickhouse-client
ch.europe-west1-c.c.nosql-367017.internal :) CREATE DATABASE IF NOT EXISTS tutorial

CREATE DATABASE IF NOT EXISTS tutorial

Query id: 9d540960-6d0f-40c6-b8f4-0fdbf3cd07d7

Ok.

0 rows in set. Elapsed: 0.005 sec. 
```
Создадим таблицу hits_v1
```
ch.europe-west1-c.c.nosql-367017.internal :) CREATE TABLE tutorial.hits_v1
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
Создадим таблицу visits_v1
```
ch.europe-west1-c.c.nosql-367017.internal :) CREATE TABLE tutorial.visits_v1
                                                (
                                                    `CounterID` UInt32,
                                                    `StartDate` Date,
                                                    `Sign` Int8,
                                                    `IsNew` UInt8,
                                                    `VisitID` UInt64,
                                                    `UserID` UInt64,
                                                    `StartTime` DateTime,
                                                    `Duration` UInt32,
                                                    `UTCStartTime` DateTime,
                                                    `PageViews` Int32,
                                                    `Hits` Int32,
                                                    `IsBounce` UInt8,
                                                    `Referer` String,
                                                    `StartURL` String,
                                                    `RefererDomain` String,
                                                    `StartURLDomain` String,
                                                    `EndURL` String,
                                                    `LinkURL` String,
                                                    `IsDownload` UInt8,
                                                    `TraficSourceID` Int8,
                                                    `SearchEngineID` UInt16,
                                                    `SearchPhrase` String,
                                                    `AdvEngineID` UInt8,
                                                    `PlaceID` Int32,
                                                    `RefererCategories` Array(UInt16),
                                                    `URLCategories` Array(UInt16),
                                                    `URLRegions` Array(UInt32),
                                                    `RefererRegions` Array(UInt32),
                                                    `IsYandex` UInt8,
                                                    `GoalReachesDepth` Int32,
                                                    `GoalReachesURL` Int32,
                                                    `GoalReachesAny` Int32,
                                                    `SocialSourceNetworkID` UInt8,
                                                    `SocialSourcePage` String,
                                                    `MobilePhoneModel` String,
                                                    `ClientEventTime` DateTime,
                                                    `RegionID` UInt32,
                                                    `ClientIP` UInt32,
                                                    `ClientIP6` FixedString(16),
                                                    `RemoteIP` UInt32,
                                                    `RemoteIP6` FixedString(16),
                                                    `IPNetworkID` UInt32,
                                                    `SilverlightVersion3` UInt32,
                                                    `CodeVersion` UInt32,
                                                    `ResolutionWidth` UInt16,
                                                    `ResolutionHeight` UInt16,
                                                    `UserAgentMajor` UInt16,
                                                    `UserAgentMinor` UInt16,
                                                    `WindowClientWidth` UInt16,
                                                    `WindowClientHeight` UInt16,
                                                    `SilverlightVersion2` UInt8,
                                                    `SilverlightVersion4` UInt16,
                                                    `FlashVersion3` UInt16,
                                                    `FlashVersion4` UInt16,
                                                    `ClientTimeZone` Int16,
                                                    `OS` UInt8,
                                                    `UserAgent` UInt8,
                                                    `ResolutionDepth` UInt8,
                                                    `FlashMajor` UInt8,
                                                    `FlashMinor` UInt8,
                                                    `NetMajor` UInt8,
                                                    `NetMinor` UInt8,
                                                    `MobilePhone` UInt8,
                                                    `SilverlightVersion1` UInt8,
                                                    `Age` UInt8,
                                                    `Sex` UInt8,
                                                    `Income` UInt8,
                                                    `JavaEnable` UInt8,
                                                    `CookieEnable` UInt8,
                                                    `JavascriptEnable` UInt8,
                                                    `IsMobile` UInt8,
                                                    `BrowserLanguage` UInt16,
                                                    `BrowserCountry` UInt16,
                                                    `Interests` UInt16,
                                                    `Robotness` UInt8,
                                                    `GeneralInterests` Array(UInt16),
                                                    `Params` Array(String),
                                                    `Goals` Nested(
                                                        ID UInt32,
                                                        Serial UInt32,
                                                        EventTime DateTime,
                                                        Price Int64,
                                                        OrderID String,
                                                        CurrencyID UInt32),
                                                    `WatchIDs` Array(UInt64),
                                                    `ParamSumPrice` Int64,
                                                    `ParamCurrency` FixedString(3),
                                                    `ParamCurrencyID` UInt16,
                                                    `ClickLogID` UInt64,
                                                    `ClickEventID` Int32,
                                                    `ClickGoodEvent` Int32,
                                                    `ClickEventTime` DateTime,
                                                    `ClickPriorityID` Int32,
                                                    `ClickPhraseID` Int32,
                                                    `ClickPageID` Int32,
                                                    `ClickPlaceID` Int32,
                                                    `ClickTypeID` Int32,
                                                    `ClickResourceID` Int32,
                                                    `ClickCost` UInt32,
                                                    `ClickClientIP` UInt32,
                                                    `ClickDomainID` UInt32,
                                                    `ClickURL` String,
                                                    `ClickAttempt` UInt8,
                                                    `ClickOrderID` UInt32,
                                                    `ClickBannerID` UInt32,
                                                    `ClickMarketCategoryID` UInt32,
                                                    `ClickMarketPP` UInt32,
                                                    `ClickMarketCategoryName` String,
                                                    `ClickMarketPPName` String,
                                                    `ClickAWAPSCampaignName` String,
                                                    `ClickPageName` String,
                                                    `ClickTargetType` UInt16,
                                                    `ClickTargetPhraseID` UInt64,
                                                    `ClickContextType` UInt8,
                                                    `ClickSelectType` Int8,
                                                    `ClickOptions` String,
                                                    `ClickGroupBannerID` Int32,
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
                                                    `FirstVisit` DateTime,
                                                    `PredLastVisit` Date,
                                                    `LastVisit` Date,
                                                    `TotalVisits` UInt32,
                                                    `TraficSource` Nested(
                                                        ID Int8,
                                                        SearchEngineID UInt16,
                                                        AdvEngineID UInt8,
                                                        PlaceID UInt16,
                                                        SocialSourceNetworkID UInt8,
                                                        Domain String,
                                                        SearchPhrase String,
                                                        SocialSourcePage String),
                                                    `Attendance` FixedString(16),
                                                    `CLID` UInt32,
                                                    `YCLID` UInt64,
                                                    `NormalizedRefererHash` UInt64,
                                                    `SearchPhraseHash` UInt64,
                                                    `RefererDomainHash` UInt64,
                                                    `NormalizedStartURLHash` UInt64,
                                                    `StartURLDomainHash` UInt64,
                                                    `NormalizedEndURLHash` UInt64,
                                                    `TopLevelDomain` UInt64,
                                                    `URLScheme` UInt64,
                                                    `OpenstatServiceNameHash` UInt64,
                                                    `OpenstatCampaignIDHash` UInt64,
                                                    `OpenstatAdIDHash` UInt64,
                                                    `OpenstatSourceIDHash` UInt64,
                                                    `UTMSourceHash` UInt64,
                                                    `UTMMediumHash` UInt64,
                                                    `UTMCampaignHash` UInt64,
                                                    `UTMContentHash` UInt64,
                                                    `UTMTermHash` UInt64,
                                                    `FromHash` UInt64,
                                                    `WebVisorEnabled` UInt8,
                                                    `WebVisorActivity` UInt32,
                                                    `ParsedParams` Nested(
                                                        Key1 String,
                                                        Key2 String,
                                                        Key3 String,
                                                        Key4 String,
                                                        Key5 String,
                                                        ValueDouble Float64),
                                                    `Market` Nested(
                                                        Type UInt8,
                                                        GoalID UInt32,
                                                        OrderID String,
                                                        OrderPrice Int64,
                                                        PP UInt32,
                                                        DirectPlaceID UInt32,
                                                        DirectOrderID UInt32,
                                                        DirectBannerID UInt32,
                                                        GoodID String,
                                                        GoodName String,
                                                        GoodQuantity Int32,
                                                        GoodPrice Int64),
                                                    `IslandID` FixedString(16)
                                                )
                                                ENGINE = CollapsingMergeTree(Sign)
                                                PARTITION BY toYYYYMM(StartDate)
                                                ORDER BY (CounterID, StartDate, intHash32(UserID), VisitID)
                                                SAMPLE BY intHash32(UserID);
```
5. Загружаем данные из файлов
```
kosten@ch:~$ clickhouse-client --query "INSERT INTO tutorial.hits_v1 FORMAT TSV" --max_insert_block_size=100000 < hits_v1.tsv && clickhouse-client --query "INSERT INTO tutorial.visits_v1 FORMAT TSV" --max_insert_block_size=100000 < visits_v1.tsv
```
6. Оптимизируем загруженные таблицы
```
kosten@ch:~$ clickhouse-client --query "OPTIMIZE TABLE tutorial.hits_v1 FINAL" && clickhouse-client --query "OPTIMIZE TABLE tutorial.visits_v1 FINAL"
```
7. Запросим количество записей из таблиц
```
kosten@ch:~$ clickhouse-client --query "SELECT COUNT(*) FROM tutorial.hits_v1" && clickhouse-client --query "SELECT COUNT(*) FROM tutorial.visits_v1"
17747796
1676861
```
8. Выполним тестовый запрос с агрегацией. 1,46 млн строк обработано за 277 мс, что гораздо быстрее реляционных баз с построчным хранением данных.
```
ch.europe-west1-c.c.nosql-367017.internal :) SELECT
                                                 StartURL AS URL,
                                                 AVG(Duration) AS AvgDuration
                                             FROM tutorial.visits_v1
                                             WHERE StartDate BETWEEN '2014-03-23' AND '2014-03-30'
                                             GROUP BY URL
                                             ORDER BY AvgDuration DESC
                                             LIMIT 10;

SELECT
    StartURL AS URL,
    AVG(Duration) AS AvgDuration
FROM tutorial.visits_v1
WHERE (StartDate >= '2014-03-23') AND (StartDate <= '2014-03-30')
GROUP BY URL
ORDER BY AvgDuration DESC
LIMIT 10

Query id: bcd11454-2b7c-48ac-b514-cb1bb7f5bc5b

┌─URL─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┬─AvgDuration─┐
│ http://itpalanija-pri-patrivative=0&ads_app_user                                                                                                                                                │       60127 │
│ http://renaul-myd-ukraine                                                                                                                                                                       │       58938 │
│ http://karta/Futbol/dynamo.kiev.ua/kawaica.su/648                                                                                                                                               │       56538 │
│ https://moda/vyikroforum1/top.ru/moscow/delo-product/trend_sms/multitryaset/news/2014/03/201000                                                                                                 │       55218 │
│ http://e.mail=on&default?abid=2061&scd=yes&option?r=city_inter.com/menu&site-zaferio.ru/c/m.ensor.net/ru/login=false&orderStage.php?Brandidatamalystyle/20Mar2014%2F007%2F94dc8d2e06e56ed56bbdd │       51378 │
│ http://karta/Futbol/dynas.com/haberler.ru/messages.yandsearchives/494503_lte_13800200319                                                                                                        │       49078 │
│ http://xmusic/vstreatings of speeds                                                                                                                                                             │       36925 │
│ http://news.ru/yandex.ru/api.php&api=http://toberria.ru/aphorizana                                                                                                                              │       36902 │
│ http://bashmelnykh-metode.net/video/#!/video/emberkas.ru/detskij-yazi.com/iframe/default.aspx?id=760928&noreask=1&source                                                                        │       34323 │
│ http://censonhaber/547-popalientLog=0&strizhki-petro%3D&comeback=search?lr=213&text                                                                                                             │       31773 │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┴─────────────┘

10 rows in set. Elapsed: 0.277 sec. Processed 1.46 million rows, 115.63 MB (5.29 million rows/s., 418.05 MB/s.)
```
9. Выполним еще один запрос с условием. Из таблицы visits_v1 выбраны 13,1 тысяч записей и по ним произведено суммирование за 21 мс.
```
ch.europe-west1-c.c.nosql-367017.internal :) SELECT
                                                 sum(Sign) AS visits,
                                                 sumIf(Sign, has(Goals.ID, 1105530)) AS goal_visits,
                                                 (100. * goal_visits) / visits AS goal_percent
                                             FROM tutorial.visits_v1
                                             WHERE (CounterID = 912887) AND (toYYYYMM(StartDate) = 201403) AND (domain(StartURL) = 'yandex.ru');

SELECT
    sum(Sign) AS visits,
    sumIf(Sign, has(Goals.ID, 1105530)) AS goal_visits,
    (100. * goal_visits) / visits AS goal_percent
FROM tutorial.visits_v1
WHERE (CounterID = 912887) AND (toYYYYMM(StartDate) = 201403) AND (domain(StartURL) = 'yandex.ru')

Query id: d7a381ae-9d20-4848-836c-04970136a702

┌─visits─┬─goal_visits─┬──────goal_percent─┐
│  10543 │        8553 │ 81.12491700654462 │
└────────┴─────────────┴───────────────────┘

1 row in set. Elapsed: 0.021 sec. Processed 13.11 thousand rows, 2.86 MB (610.45 thousand rows/s., 133.32 MB/s.)
```
10. Создадим таблицу для дополнительного датасета UK Property Price Paid,
```
CREATE TABLE uk_price_paid
(
    price UInt32,
    date Date,
    postcode1 LowCardinality(String),
    postcode2 LowCardinality(String),
    type Enum8('terraced' = 1, 'semi-detached' = 2, 'detached' = 3, 'flat' = 4, 'other' = 0),
    is_new UInt8,
    duration Enum8('freehold' = 1, 'leasehold' = 2, 'unknown' = 0),
    addr1 String,
    addr2 String,
    street LowCardinality(String),
    locality LowCardinality(String),
    town LowCardinality(String),
    district LowCardinality(String),
    county LowCardinality(String)
)
ENGINE = MergeTree
ORDER BY (postcode1, postcode2, addr1, addr2);
```
11. Загрузим данные напрямую через url
```
INSERT INTO uk_price_paid
WITH
   splitByChar(' ', postcode) AS p
SELECT
    toUInt32(price_string) AS price,
    parseDateTimeBestEffortUS(time) AS date,
    p[1] AS postcode1,
    p[2] AS postcode2,
    transform(a, ['T', 'S', 'D', 'F', 'O'], ['terraced', 'semi-detached', 'detached', 'flat', 'other']) AS type,
    b = 'Y' AS is_new,
    transform(c, ['F', 'L', 'U'], ['freehold', 'leasehold', 'unknown']) AS duration,
    addr1,
    addr2,
    street,
    locality,
    town,
    district,
    county
FROM url(
    'http://prod.publicdata.landregistry.gov.uk.s3-website-eu-west-1.amazonaws.com/pp-complete.csv',
    'CSV',
    'uuid_string String,
    price_string String,
    time String,
    postcode String,
    a String,
    b String,
    c String,
    addr1 String,
    addr2 String,
    street String,
    locality String,
    town String,
    district String,
    county String,
    d String,
    e String'
) SETTINGS max_http_get_redirects=10;
Query id: e26aa078-1b9c-4715-afd3-03542cd2ab7b

Ok.

0 rows in set. Elapsed: 131.511 sec. Processed 27.73 million rows, 7.51 GB (210.90 thousand rows/s., 57.12 MB/s.)
```
11. Проверим объем места, занимаемого данными в БД, и удивимся - загруженные 7.5 Гб превратились в 298 Мб
```
ch.europe-west1-c.c.nosql-367017.internal :) SELECT formatReadableSize(total_bytes)
                                             FROM system.tables
                                             WHERE name = 'uk_price_paid';

SELECT formatReadableSize(total_bytes)
FROM system.tables
WHERE name = 'uk_price_paid'

Query id: c5908cc7-e0f4-4e03-ada6-fb11fb248e70

┌─formatReadableSize(total_bytes)─┐
│ 298.19 MiB                      │
└─────────────────────────────────┘

1 row in set. Elapsed: 0.002 sec. 
```
12. Запросимс среднюю цену по году. Все очень быстро и красиво - всего 306 мс понадобилось для обработки 27,7 млн строк. Функция bar делает реузльтат очень наглядным
```
ch.europe-west1-c.c.nosql-367017.internal :) SELECT
                                                toYear(date) AS year,
                                                round(avg(price)) AS price,
                                                bar(price, 0, 1000000, 80
                                             )
                                             FROM uk_price_paid
                                             GROUP BY year;

SELECT
    toYear(date) AS year,
    round(avg(price)) AS price,
    bar(price, 0, 1000000, 80)
FROM uk_price_paid
GROUP BY year

Query id: 27f6c43e-17bb-4c54-89d4-64f10a6c60dc

┌─year─┬──price─┬─bar(round(avg(price)), 0, 1000000, 80)─┐
│ 1995 │  67935 │ █████▍                                 │
│ 1996 │  71509 │ █████▋                                 │
│ 1997 │  78537 │ ██████▎                                │
│ 1998 │  85442 │ ██████▋                                │
│ 1999 │  96038 │ ███████▋                               │
│ 2000 │ 107487 │ ████████▌                              │
│ 2001 │ 118890 │ █████████▌                             │
│ 2002 │ 137955 │ ███████████                            │
│ 2003 │ 155895 │ ████████████▍                          │
│ 2004 │ 178889 │ ██████████████▎                        │
│ 2005 │ 189361 │ ███████████████▏                       │
│ 2006 │ 203532 │ ████████████████▎                      │
│ 2007 │ 219375 │ █████████████████▌                     │
│ 2008 │ 217042 │ █████████████████▎                     │
│ 2009 │ 213418 │ █████████████████                      │
│ 2010 │ 236111 │ ██████████████████▊                    │
│ 2011 │ 232807 │ ██████████████████▌                    │
│ 2012 │ 238383 │ ███████████████████                    │
│ 2013 │ 256927 │ ████████████████████▌                  │
│ 2014 │ 280012 │ ██████████████████████▍                │
│ 2015 │ 297280 │ ███████████████████████▋               │
│ 2016 │ 313533 │ █████████████████████████              │
│ 2017 │ 346430 │ ███████████████████████████▋           │
│ 2018 │ 350609 │ ████████████████████████████           │
│ 2019 │ 352401 │ ████████████████████████████▏          │
│ 2020 │ 376482 │ ██████████████████████████████         │
│ 2021 │ 382080 │ ██████████████████████████████▌        │
│ 2022 │ 378166 │ ██████████████████████████████▎        │
└──────┴────────┴────────────────────────────────────────┘

28 rows in set. Elapsed: 0.306 sec. Processed 27.73 million rows, 166.41 MB (90.65 million rows/s., 543.87 MB/s.)
```
13. Добавим проекцию к нашей таблице
```
ALTER TABLE uk_price_paid
    ADD PROJECTION projection_by_year_district_town
    (
        SELECT
            toYear(date),
            district,
            town,
            avg(price),
            sum(price),
            count()
        GROUP BY
            toYear(date),
            district,
            town
    );
```
и материализуем ее
```
ch.europe-west1-c.c.nosql-367017.internal :) ALTER TABLE uk_price_paid
                                                 MATERIALIZE PROJECTION projection_by_year_district_town
                                             SETTINGS mutations_sync = 1;

ALTER TABLE uk_price_paid
    MATERIALIZE PROJECTION projection_by_year_district_town
SETTINGS mutations_sync = 1

Query id: 4466b8de-b9e4-44fb-9f3f-c1fff323b450

Ok.

0 rows in set. Elapsed: 2.167 sec. 
```
14. Теперь снова запустим запрос со средней ценой
```
SELECT
    toYear(date) AS year,
    round(avg(price)) AS price,
    bar(price, 0, 1000000, 80)
FROM uk_price_paid
GROUP BY year
ORDER BY year ASC;
```
Результат получаем тот же, но скорость увеличилась почти в 10 раз!
```
NO PROJECTION: 28 rows in set. Elapsed: 0.306 sec. Processed 27.73 million rows, 166.41 MB (90.65 million rows/s., 543.87 MB/s.)
WITH PROJECTION: 28 rows in set. Elapsed: 0.036 sec. Processed 87.96 thousand rows, 3.28 MB (2.47 million rows/s., 92.09 MB/s.)
```
