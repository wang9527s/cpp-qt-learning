
## 客户端

```cpp
#include<QtNetwork>
QTcpSocket *socket = new QTcpSocket();
socket->connectToHost("127.0.0.1", 6666);
QObject::connect(socket, &QTcpSocket::readyRead,
        [=]()
    {
    #if 1
        QByteArray buf;
        buf=socket->readAll();//接数据
        qDebug()<<buf;
    #endif
    #if 0
        QDataStream in(socket);
        QString str;
        int a;
        in>>str>>a;
        qDebug()<<"str:"<<str<<"a:"<<a;
    #endif
    #if 1
        QByteArray byte = socket->readAll();
        qDebug() << byte;    //输出："\x00\x00\x01\x00"
#endif
    });
```

## 服务器

当有客户端连接时，发送hello NetWork给客户端

```cpp
#include <QtNetwork>
QTcpServer *tcpServer = new QTcpServer();
tcpServer->listen(QHostAddress::LocalHost, 6666);
connect(tcpServer, &QTcpServer::newConnection,
        [=]()
    {
        QTcpSocket *socket=tcpServer->nextPendingConnection();
#if 1
        QByteArray buf="hello Network";
        socket->write(buf);
#endif
#if 0
        /*不要同时使用，会出错！  可能是黏包，*/
        QByteArray buf;
        QDataStream out(&buf,QIODevice::WriteOnly);
        /*只发送数字123出错。可能是数据太少，或者是需要以字符串开头*/
        out<<QString("QDataStream")<<123;
        socket->write(buf);
#endif
#if 0
        QByteArray buf;
        /*只发送数字123出错。可能是数据太少，或者是需要以字符串开头*/
        buf[0]=0x00;
        buf[1]=0x00;
        buf[2]=0x01;
        buf[3]=0x00;
        socket->write(buf);
    });
#endif
```


