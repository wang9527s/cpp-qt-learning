
#### QUrl的组成

#### Qurl和QurlQuery的搭配使用方法

```cpp
//以?为分隔，其前部分为不带QurlQuery的Url，其后部分为QurlQuery部分。
QUrl("https://www.foo.com?email=foo@bar.com&pass=secret")
```

```cpp
//向QUrl中添加QUrlQuery
QUrl url("https://www.foo.com");
QUrlQuery query;
query.addQueryItem("email", "foo@bar.com");
query.addQueryItem("pass", "secret");
url.setQuery(query);
qDebug() << url;    //QUrl("https://www.foo.com?email=foo@bar.com&pass=secret")

//提取QUrl中的query请求"参数-值"对
QUrlQuery query2(url.query());
QList<QPair<QString, QString> > list = query.queryItems();
for(int i=0;i<list.size();++i)
{
    QPair<QString,QString> pair=list[i];
    qDebug()<<pair.first<<pair.second;//注意：是成员变量而不是成员函数
}

//QUrl的query请求"参数-值"对分割符
QUrlQuery query3(url.query());
QChar pair = query3.queryPairDelimiter();  // '&'
QChar value = query3.queryValueDelimiter();  // '='
qDebug()<<pair<<"  "<<value;
//     query3.setQueryDelimiters('(', ')');
//     qDebug()<< query3.queryPairDelimiter()<<query3.queryValueDelimiter();
//     QUrl outUrl("https://www.foo.com");
//     outUrl.setQuery(query3);
//     qDebug()<<outUrl;    //QUrl("https://www.foo.com?email(foo@bar.com)pass(secret")
```
