### UDP发送
```cpp
QByteArray array;
QUdpSocket * sender=new QUdpSocket;
sender->bind(QHostAddress::LocalHost, 8080);
while(1)
{
    sender->writeDatagram(
            array,
            QHostAddress("239.0.0.1"),
            7000
            );
}

```
