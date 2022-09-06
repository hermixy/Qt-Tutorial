**实现简单的结构体传输:** 两端传输结构体。

服务端
```C
#include <QCoreApplication>
#include <QTcpServer>
#include <QTcpSocket>
#include <iostream>

struct MyStruct
{
    char uname[7];
    qint32 id;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    MyStruct ptr;
    QTcpServer server;

    server.listen(QHostAddress::Any,9000);
    server.waitForNewConnection(100000);

    QTcpSocket *socket;

    socket = server.nextPendingConnection();
    while(socket->state() && QAbstractSocket::ConnectedState)
    {
        socket->waitForReadyRead(1000);
        socket->read((char*)&ptr,sizeof(MyStruct));
        std::cout << ptr.uname << std::endl;

        socket->write((char *)"sz",sizeof("sz"));
    }

    socket->close();
    server.close();
    return a.exec();
}
```
客户端
```C
#include <QCoreApplication>
#include <QTcpServer>
#include <QTcpSocket>
#include <iostream>

struct MyStruct
{
    char uname[7];
    qint32 id;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QTcpSocket socket;

    MyStruct ptr = {"abc",1001};

    socket.connectToHost(QHostAddress::LocalHost,9000);

    while(socket.state() && QAbstractSocket::ConnectedState)
    {
        socket.waitForConnected();
        socket.write((char *)&ptr,sizeof(MyStruct));

        socket.waitForBytesWritten();
        socket.waitForReadyRead(10000);

        char sz_ref[1024]={0};

        QByteArray qb = socket.readAll();

        QString str;
        str.prepend(qb);
        std::cout << str.toStdString() << std::endl;
    }

    socket.close();
    return a.exec();
}
```

**实现简单传递字符串:**

client
```C
#include <QCoreApplication>
#include <QTcpServer>
#include <QTcpSocket>
#include <iostream>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QTcpSocket socket;
    socket.connectToHost(QHostAddress::LocalHost,9000);

    if(socket.state() && QAbstractSocket::ConnectedState)
    {
        socket.waitForReadyRead(10000);

        QByteArray ref = socket.readAll();

        QString ref_string;

        ref_string.prepend(ref);

        std::cout << ref_string.toStdString() << std::endl;
    }

    socket.close();
    return a.exec();
}
```
server
```C
#include <QCoreApplication>
#include <QTcpServer>
#include <QTcpSocket>
#include <iostream>

struct MyStruct
{
    char uname[7];
    qint32 id;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    MyStruct ptr;
    QTcpServer server;

    server.listen(QHostAddress::Any,9000);
    server.waitForNewConnection(100000);

    QTcpSocket *socket;

    socket = server.nextPendingConnection();
    if(socket->state() && QAbstractSocket::ConnectedState)
    {
        QString str = "abcde";
        QByteArray bytes = str.toUtf8();
        socket->write(bytes.data(),bytes.length());
    }

    socket->close();
    server.close();
    return a.exec();
}
```

**简单通信框架**

server
```C
#include <QCoreApplication>
#include <QTcpServer>
#include <QTcpSocket>
#include <iostream>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QTcpServer server;

    server.listen(QHostAddress::Any,9000);
    server.waitForNewConnection(100000);

    QTcpSocket *socket;

    socket = server.nextPendingConnection();
    if(socket->state() && QAbstractSocket::ConnectedState)
    {
        // 获取CPU数据
        QString str = "GetCPU";
        QByteArray bytes = str.toUtf8();
        socket->write(bytes.data(),bytes.length());

        // 接收返回结果
        socket->waitForReadyRead(10000);
        QByteArray ref = socket->readAll();
        QString ref_buffer(ref);
        std::cout << "返回结果: " << ref_buffer.toStdString() << std::endl;
    }

    socket->close();
    server.close();
    return a.exec();
}
```
client
```C
#include <QCoreApplication>
#include <QTcpServer>
#include <QTcpSocket>
#include <iostream>
#include <QString>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    QTcpSocket socket;

    while(1)
    {
        // 循环链接服务器
        socket.connectToHost(QHostAddress::LocalHost,9000);

        // 循环接收数据包,并处理请求
        while(socket.state() && QAbstractSocket::ConnectedState)
        {
            socket.waitForReadyRead(10000);

            // 将字节序转为字符串
            QByteArray ref = socket.readAll();
            QString ref_string(ref);

            if( (QString::compare(ref_string,"GetCPU")) == 0)
            {
                std::cout << "获取CPU数据" << std::endl;

                QString str = "94%";
                QByteArray bytes = str.toUtf8();
                socket.write(bytes.data(),bytes.length());
            }
            else if( QString::compare(ref_string,"GetMemory") == 0)
            {
                std::cout << "ref memory" << std::endl;
            }
            else if( QString::compare(ref_string,"GetLoadAvg") == 0)
            {
                std::cout << "ref load avg" << std::endl;
            }
        }
    }

    socket.close();
    return a.exec();
}
```
