Qt文件操作有两种方式，第一种使用QFile类的IODevice读写功能直接读写，第二种是利用 QFile和QTextStream结合起来，用流的方式进行文件读写。

第一种，利用QFile中的相关函数，实现对文件的读写操作，QFile会调用IODevice设备，从而实现文件读写。

**QT基本文件读写:** 通过QFile实现文本文件读写操作.
```C
#include <QCoreApplication>
#include <iostream>
#include <QFile>
#include <QString>
#include <QTextStream>

// 一次读入所有文本
bool ReadFileOnly(const QString &file_path)
{
    QFile ptr(file_path);

    // 文件是否存在
    if(!ptr.exists())
    {
        return false;
    }

    // 文件是否打开
    /*
    ReadOnly   以只读方式打开
    WriteOnly  以只写方式打开
    ReadWrite  读写方式打开
    Append     以追加方式打开
    Truncate   以截取方式打开(原有内容被清空)
    Text       以文件方式打开
    */

    if(!ptr.open(QIODevice::ReadWrite | QIODevice::Text))
    {
        return false;
    }

    QString text = ptr.readAll();
    std::cout << text.toStdString() << std::endl;
    ptr.close();
}

// 追加写入文本
bool WriteFileOnly(const QString &file_path, QString save)
{
    // 如果参数为空则返回假
    if(file_path.isEmpty() && save.isEmpty())
    {
        return false;
    }

    QFile ptr(file_path);
    if(!ptr.open(QIODevice::Append | QIODevice::Text))
    {
        return false;
    }

    QByteArray str_bytes = save.toUtf8();

    ptr.write(str_bytes,str_bytes.length());
    ptr.close();
    return true;
}
```

**QTextStream 实现流读写：** 直接使用流写入，可以使用<< 运算符，方便的写入文本。
```C
#include <QCoreApplication>
#include <iostream>
#include <QFile>
#include <QString>
#include <QTextStream>
#include <QTextCodec>

// 计算文件行数
qint32 get_file_count(const QString &file_path)
{
    QFile ptr(file_path);
    qint32 count = 0;

    if(ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        QTextStream in(&ptr);

        // 自动检测unicode编码,显示中文
        in.setAutoDetectUnicode(true);

        while(!in.atEnd())
        {
            QString line = in.readLine();
            std::cout << line.toStdString() << std::endl;
            count = count +1;
        }

        return count;
    }
    return 0;
}

// 追加写入数据
bool write_file_stream(const QString &file_path, QString save)
{
    QFile ptr(file_path);

    if(ptr.open(QIODevice::Append | QIODevice::Text))
    {
        QTextStream in(&ptr);
        in << save;
    }
    ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 设置编码
    QTextCodec *codec = QTextCodec::codecForName("utf-8");
    QTextCodec::setCodecForLocale(codec);

    // 流写入
    write_file_stream("d://test.txt","hello lyshark");
    write_file_stream("d://test.txt","你好,世界");

    // 取文本长度
    qint32 count = get_file_count("d://test.txt");
    std::cout << "line = > " << count << std::endl;
    return a.exec();
}
```
