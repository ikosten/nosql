
### Базовые возможности MongoDB

1. Скачиваем файл из тренировочного датасета с размеченными новостями: правда или ложь https://www.kaggle.com/c/fake-news/data, и называем его fakenews.csv
2. Загужаем файл в MongoDB c помощью утилиты mongoimport
```
kosten@mongo:~$ mongoimport -u root -d assignments -c fakenews --headerline --authenticationDatabase admin --type csv fakenews.csv
Enter password for mongo user:

2022-11-14T17:50:56.712+0000	connected to: mongodb://localhost/
2022-11-14T17:50:59.714+0000	[#################.......] assignments.fakenews	68.6MB/94.1MB (72.9%)
2022-11-14T17:51:00.803+0000	[########################] assignments.fakenews	94.1MB/94.1MB (100.0%)
```
3. Подкючаемся к БД
```
kosten@mongo:~$ mongosh -u root assignments --authenticationDatabase admin
Enter password: ****
Current Mongosh Log ID:	6372863b5954a07a73158374
Connecting to:		mongodb://<credentials>@127.0.0.1:27017/assignments?directConnection=true&serverSelectionTimeoutMS=2000&authSource=admin&appName=mongosh+1.6.0
Using MongoDB:		6.0.2
Using Mongosh:		1.6.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2022-11-14T18:16:23.220+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2022-11-14T18:16:24.205+00:00: vm.max_map_count is too low
------
```
4. Проверяем количество загруженных записей в коллекции fakenews
```
assignments> db.fakenews.countDocuments()
20800
```
5. Запросим первую запись из коллекции, чтобы ознакомиться со структурой
```
assignments> db.fakenews.find().limit(1)
[
  {
    _id: ObjectId("63728000366a750f91e975c3"),
    id: 0,
    title: 'House Dem Aide: We Didn’t Even See Comey’s Letter Until Jason Chaffetz Tweeted It',
    author: 'Darrell Lucus',
    text: 'House Dem Aide: We Didn’t Even See Comey’s Letter Until Jason Chaffetz Tweeted It By Darrell Lucus on October 30, 2016 Subscribe Jason Chaffetz on the stump in American Fork, Utah ( image courtesy Michael Jolley, available under a Creative Commons-BY license) \n' +
      'With apologies to Keith Olbermann, there is no doubt who the Worst Person in The World is this week–FBI Director James Comey. But according to a House Democratic aide, it looks like we also know who the second-worst person is as well. It turns out that when Comey sent his now-infamous letter announcing that the FBI was looking into emails that may be related to Hillary Clinton’s email server, the ranking Democrats on the relevant committees didn’t hear about it from Comey. They found out via a tweet from one of the Republican committee chairmen. \n' +
      'As we now know, Comey notified the Republican chairmen and Democratic ranking members of the House Intelligence, Judiciary, and Oversight committees that his agency was reviewing emails it had recently discovered in order to see if they contained classified information. Not long after this letter went out, Oversight Committee Chairman Jason Chaffetz set the political world ablaze with this tweet. FBI Dir just informed me, "The FBI has learned of the existence of emails that appear to be pertinent to the investigation." Case reopened \n' +
      '— Jason Chaffetz (@jasoninthehouse) October 28, 2016 \n' +
      'Of course, we now know that this was not the case . Comey was actually saying that it was reviewing the emails in light of “an unrelated case”–which we now know to be Anthony Weiner’s sexting with a teenager. But apparently such little things as facts didn’t matter to Chaffetz. The Utah Republican had already vowed to initiate a raft of investigations if Hillary wins–at least two years’ worth, and possibly an entire term’s worth of them. Apparently Chaffetz thought the FBI was already doing his work for him–resulting in a tweet that briefly roiled the nation before cooler heads realized it was a dud. \n' +
      'But according to a senior House Democratic aide, misreading that letter may have been the least of Chaffetz’ sins. That aide told Shareblue that his boss and other Democrats didn’t even know about Comey’s letter at the time–and only found out when they checked Twitter. “Democratic Ranking Members on the relevant committees didn’t receive Comey’s letter until after the Republican Chairmen. In fact, the Democratic Ranking Members didn’ receive it until after the Chairman of the Oversight and Government Reform Committee, Jason Chaffetz, tweeted it out and made it public.” \n' +
      'So let’s see if we’ve got this right. The FBI director tells Chaffetz and other GOP committee chairmen about a major development in a potentially politically explosive investigation, and neither Chaffetz nor his other colleagues had the courtesy to let their Democratic counterparts know about it. Instead, according to this aide, he made them find out about it on Twitter. \n' +
      'There has already been talk on Daily Kos that Comey himself provided advance notice of this letter to Chaffetz and other Republicans, giving them time to turn on the spin machine. That may make for good theater, but there is nothing so far that even suggests this is the case. After all, there is nothing so far that suggests that Comey was anything other than grossly incompetent and tone-deaf. \n' +
      'What it does suggest, however, is that Chaffetz is acting in a way that makes Dan Burton and Darrell Issa look like models of responsibility and bipartisanship. He didn’t even have the decency to notify ranking member Elijah Cummings about something this explosive. If that doesn’t trample on basic standards of fairness, I don’t know what does. \n' +
      'Granted, it’s not likely that Chaffetz will have to answer for this. He sits in a ridiculously Republican district anchored in Provo and Orem; it has a Cook Partisan Voting Index of R+25, and gave Mitt Romney a punishing 78 percent of the vote in 2012. Moreover, the Republican House leadership has given its full support to Chaffetz’ planned fishing expedition. But that doesn’t mean we can’t turn the hot lights on him. After all, he is a textbook example of what the House has become under Republican control. And he is also the Second Worst Person in the World. About Darrell Lucus \n' +
      "Darrell is a 30-something graduate of the University of North Carolina who considers himself a journalist of the old school. An attempt to turn him into a member of the religious right in college only succeeded in turning him into the religious right's worst nightmare--a charismatic Christian who is an unapologetic liberal. His desire to stand up for those who have been scared into silence only increased when he survived an abusive three-year marriage. You may know him on Daily Kos as Christian Dem in NC . Follow him on Twitter @DarrellLucus or connect with him on Facebook . Click here to buy Darrell a Mello Yello. Connect",
    label: 1
  }
]
```
6. Выведем TOP-10 авторов с наибольшим количеством фейков
```
assignments> db.fakenews.aggregate( {$group : {_id : "$author", count : {$sum : 1}, count_fake : {$sum : "$label"} } },{$sort: {count_fake: -1}},{$limit:10})
[
  { _id: NaN, count: 1957, count_fake: 1931 },
  { _id: 'admin', count: 193, count_fake: 193 },
  { _id: 'Pakalert', count: 86, count_fake: 86 },
  { _id: 'Eddy Lavine', count: 85, count_fake: 85 },
  { _id: 'Starkman', count: 84, count_fake: 84 },
  { _id: 'Gillian', count: 82, count_fake: 82 },
  { _id: 'Alex Ansary', count: 82, count_fake: 82 },
  { _id: 'Editor', count: 81, count_fake: 81 },
  {
    _id: 'noreply@blogger.com (Alexander Light)',
    count: 80,
    count_fake: 80
  },
  { _id: 'Dave Hodges', count: 77, count_fake: 77 }
]
```
7. Выведем TOP-10 надежных авторов
```
assignments> db.fakenews.aggregate( {$group : {_id : "$author", count : {$sum : 1}, count_fake : {$sum : "$label"} } },{$sort: {count_fake: 1, count: -1}},{$limit:10})
[
  { _id: 'Jerome Hudson', count: 166, count_fake: 0 },
  { _id: 'Charlie Spiering', count: 141, count_fake: 0 },
  { _id: 'John Hayward', count: 140, count_fake: 0 },
  { _id: 'Katherine Rodriguez', count: 124, count_fake: 0 },
  { _id: 'Warner Todd Huston', count: 122, count_fake: 0 },
  { _id: 'Ian Hanchett', count: 119, count_fake: 0 },
  { _id: 'Breitbart News', count: 118, count_fake: 0 },
  { _id: 'Daniel Nussbaum', count: 112, count_fake: 0 },
  { _id: 'AWR Hawkins', count: 107, count_fake: 0 },
  { _id: 'Jeff Poor', count: 107, count_fake: 0 }
]
```
8. Заменим в неопределенных авторах NaN на 'undefined'
```
assignments> db.fakenews.updateMany({author: NaN},{ $set: {author: 'undefined'}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1957,
  modifiedCount: 1957,
  upsertedCount: 0
}
```
9. Проверим, что данные обновились
```
assignments> db.fakenews.aggregate( {$group : {_id : "$author", count : {$sum : 1}, count_fake : {$sum : "$label"} } },{$sort: {count_fake: -1}},{$limit:10})
[
  { _id: 'undefined', count: 1957, count_fake: 1931 },
  { _id: 'admin', count: 193, count_fake: 193 },
  { _id: 'Pakalert', count: 86, count_fake: 86 },
  { _id: 'Eddy Lavine', count: 85, count_fake: 85 },
  { _id: 'Starkman', count: 84, count_fake: 84 },
  { _id: 'Gillian', count: 82, count_fake: 82 },
  { _id: 'Alex Ansary', count: 82, count_fake: 82 },
  { _id: 'Editor', count: 81, count_fake: 81 },
  {
    _id: 'noreply@blogger.com (Alexander Light)',
    count: 80,
    count_fake: 80
  },
  { _id: 'Anonymous', count: 77, count_fake: 77 }
]
```
10. Удалим все статьи автора Anonymous
```
assignments> db.fakenews.deleteMany({author: 'Anonymous'})
{ acknowledged: true, deletedCount: 77 }
```
11. Проверим, что статьи удалились
```
assignments> db.fakenews.aggregate( {$group : {_id : "$author", count : {$sum : 1}, count_fake : {$sum : "$label"} } },{$sort: {count_fake: -1}},{$limit:10})
[
  { _id: 'undefined', count: 1957, count_fake: 1931 },
  { _id: 'admin', count: 193, count_fake: 193 },
  { _id: 'Pakalert', count: 86, count_fake: 86 },
  { _id: 'Eddy Lavine', count: 85, count_fake: 85 },
  { _id: 'Starkman', count: 84, count_fake: 84 },
  { _id: 'Alex Ansary', count: 82, count_fake: 82 },
  { _id: 'Gillian', count: 82, count_fake: 82 },
  { _id: 'Editor', count: 81, count_fake: 81 },
  {
    _id: 'noreply@blogger.com (Alexander Light)',
    count: 80,
    count_fake: 80
  },
  { _id: 'Dave Hodges', count: 77, count_fake: 77 }
]
```

### Сравнение производительности при использовании индексов

12. Запросим количество статей, в тексте которых есть "Trump"
```
assignments> let start = Date.now(); db.fakenews.countDocuments({text: {$regex: 'Trump'}}); print("Execution time:", Date.now() - start, " ms");
xecution time: 321  ms
```
13. Создадим индекс по полю text
```
assignments> db.fakenews.createIndex( {"text" : "text" })
text_text
```
14. Найдем статьи с упоминанием "Trump" с использованием текстового индекса
```
assignments>  let start = Date.now(); db.fakenews.countDocuments({$text: {$search: 'Trump'}}); print("Execution time:", Date.now() - start, " ms");
Execution time: 34  ms
```
Видим, что время выполнения уменьшилось практически в 10 раз.