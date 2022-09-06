QT是一个跨平台的图形化类库，常用数据结构就是对C++ STL的二次封装，使其更加易用，如下是经常会用到的一些数据结构和算法笔记。

### 字符串容器

**QString 追加/删除:**
```C
#include <QCoreApplication>
#include <iostream>
#include <QChar>
#include <QString>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 定义两个字符串并将其链接在一起
    QString Str1 = "hello", Str2 = "lyshark",temp;
    temp = Str1 + Str2;

    std::cout << temp.toStdString().data() << std::endl;
    std::cout << (Str1+Str2).toStdString().data() << std::endl;

    // 使用append/remove 追加与移除
    QString Str3 = "hello ";
    Str3.append("lyshark");
    Str3.push_back("test");
    Str3.remove("hello");
    Str3.prepend("-->");
    std::cout << Str3.toStdString().data() << std::endl;

    // 使用Sprintf/arg 将特定字符串连接
    QString Str4;
    Str4.sprintf("%s %s","Welcome","to you !");
    std::cout << Str4.toStdString().data() << std::endl;

    QString Str5;
    Str5 = QString("%1 is age =  %2 . ").arg("lyshark").arg("24");
    std::cout << Str5.toStdString().data() << std::endl;
    std::cout << (QString("1") + QChar('A')).toStdString().data() << std::endl;
    std::cout << (QString("2") + QString('B')).toStdString().data() << std::endl;

    // 实现统计字符串长度
    std::cout << Str5.count() << std::endl;
    std::cout << Str5.size() << std::endl;
    std::cout << Str5.length() << std::endl;

    // 去空格
    QString Str6 = " hello  lyshark   welcome !  ";
    Str6 = Str6.trimmed();    // 去掉首尾空格
    Str6 = Str6.simplified();   // 去掉所有空格,中间连续的只保留一个
    std::cout << Str6.toStdString().data() << std::endl;

    Str6 = Str6.mid(2,10);       // 从索引2开始向后取10
    std::cout << Str6.toStdString().data() << std::endl;

    //移除，1，3两个位置的字符
    std::cout << (QString("123456").remove(1,3)).toStdString().data() << std::endl;

    // 超过 11 个字符就保留 11 个字符，否则不足替换为 '.'
    std::cout << (QString("abcdefg").leftJustified(11,'.',true)).toStdString().data() << std::endl;

    std::cout << (QString::number(100,16)).toStdString().data() << std::endl;    // 100 转16进制

    // 转换为 16 进制，不足 8 位前面补 ‘0’
    std::cout << (QString("0%1").arg(123,8,16,QLatin1Char('0'))).toStdString().data() << std::endl;

    // 转为8进制
    std::cout << QString("0%1").arg(QString::number(100,8)).toStdString().data() << std::endl;
    std::cout << (QString("0%1").arg(QString::number(.777,'f',1))).toStdString().data() << std::endl;

    // 切割字符串
    std::cout << (QString("1,2,3,4,5,6").split(',')[2]).toStdString().data() << std::endl;

    // 类型转换  QByteArray 转换 QString
    QByteArray byte;

    byte.resize(2);
    byte[0]='1';
    byte[1]='2';
    QString strs = byte;

    return a.exec();
}
```

**QString 查询/替换:** 字符串的查询，替换，扫描与切割。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 查询与替换
    QString str = "hello lyshark welcome admin";
    int index;
    bool ref;

    // 查询字符串中是否包含特定字符
    ref = str.contains("lyshark",Qt::CaseInsensitive);  // 不区分大小写
    std::cout << ref << std::endl;

    ref = str.contains("LYSHARK",Qt::CaseSensitive);    // 区分大小写
    std::cout << ref << std::endl;

    // 判断是否以某个字符串开头或结束
    ref = str.startsWith("hello",Qt::CaseInsensitive); // 判断是否hello开头
    std::cout << ref << std::endl;

    ref = str.endsWith("lyshark",Qt::CaseSensitive);        // 判断是否lyshark结尾
    std::cout << ref << std::endl;

    // 从字符串中取左边/右边多少个字符
    index = str.indexOf(" ");        // 第一个空格出现的位置
    std::cout << str.left(index).toStdString().data()<< std::endl;

    index = str.lastIndexOf(" ");    // 最后一个空格出现的位置
    std::cout << str.right(str.size() - index - 1).toStdString().data() << std::endl;

    // 替换字符串中所有的lyshark为admin
    str = str.replace("lyshark","admin");
    std::cout << str.toStdString().data() << std::endl;

    // 字符串的截取
    QString str2 = "uname,uage,usex";
    std::cout << str2.section(",",0,0).toStdString().data() << std::endl;
    std::cout << str2.section(",",1,1).toStdString().data() << std::endl;

    QString str3 ="192.168.1.10";
    std::cout << str3.left(str3.indexOf(".")).toStdString().data() << std::endl;
    std::cout << str3.mid(str3.indexOf(".")+1,3).toStdString().data() << std::endl;
    std::cout << str3.mid(str3.indexOf(".")+1,1).toStdString().data() << std::endl;
    std::cout << str3.right(str3.size() - (str3.lastIndexOf(".")+1)).toStdString().data() << std::endl;

    // 判断字符串是否为空
    QString str4,str5="";
    std::cout << str4.isNull() << std::endl;    // 为空则为True
    std::cout << str5.isNull() << std::endl;    // \0不为空
    std::cout << str5.isEmpty() << std::endl;   // 为空则为False

    return a.exec();
}
```

**QString 字符串转换:**
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QByteArray>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QString str = "uname,uage,usex";
    QString int_str = "100,200,300";

    // 大小写转换
    str = str.toUpper();            // 转为大写
    std::cout << str.toStdString().data() << std::endl;
    str = str.toLower();            // 转为小写
    std::cout << str.toStdString().data() << std::endl;

    // 将字符串转为整数
    bool flag = false;
    QString x = int_str.section(",",0,0);   // 提取出第一个字符串

    int dec = x.toInt(&flag,10);              // 转为十进制整数
    std::cout << dec << std::endl;

    int hex = x.toUInt(&flag,16);            // 转为十六进制数
    std::cout << hex << std::endl;

    // 将整数转为字符串
    int number = 100;
    QString number_str;

    number_str = number_str.setNum(number,16);  // 转为十六进制字符串
    std::cout << number_str.toStdString().data() << std::endl;

    // 编码之间的转换
    QString str_string = "welcome to you !";

    QByteArray ba = str_string.toUtf8();
    std::cout << ba.toStdString().data() << std::endl;

    // 格式化输出转换
    float total = 3.1415926;
    QString str_total;

    str_total = str_total.sprintf("%.4f",total);
    std::cout << str_total.toStdString().data() << std::endl;

    str_total = QString::asprintf("%2f",total);
    std::cout << str_total.toStdString().data() << std::endl;

    return a.exec();
}
```
<br>

### 顺序容器

顺序容器 QList,QLinkedList,QVector,QStack,QQueue

**qlist:** 顺序容器,qlist是以下表的方式对数据进行访问的，可以使用下表索引的方式访问特定数据。
```C
#include <QCoreApplication>
#include <iostream>
#include <QList>

void Display(QList<QString> &ptr)
{
    std::cout << "-----------------------------" << std::endl;
    for(qint32 x=0;x<ptr.count();x++)
    {
        // std::cout << ptr[x].toStdString().data() << std::endl;
        std::cout << (ptr.at(x)).toStdString().data() << std::endl;
    }
    std::cout << std::endl;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QList<QString> StringPtrA;
    QList<QString> StringPtrB;

    // 添加三个成员
    StringPtrA.append("admin");
    StringPtrA.append("guest");
    StringPtrA.append("lyshark");
    Display(StringPtrA);

    // 在首部插入hanter
    StringPtrA.prepend("hanter");
    Display(StringPtrA);

    // 在第0的位置插入lucy
    StringPtrA.insert(0,QString("lucy"));
    Display(StringPtrA);

    // 替换原来的admin为全拼
    StringPtrA.replace(1,"Administrator");
    Display(StringPtrA);

    // 删除第0个元素
    StringPtrA.removeAt(0);
    Display(StringPtrA);

    // 删除首部和尾部
    StringPtrA.removeFirst();
    StringPtrA.removeLast();

    // 移动两个变量
    StringPtrA.move(0,1);
    Display(StringPtrA);

    // 将两个list容器对调交换
    StringPtrB = {"youtube","facebook"};
    StringPtrA.swap(StringPtrB);
    Display(StringPtrA);
    return a.exec();
}
```
qlist可以指定一个struct结构进行数据操作.
```C
#include <QCoreApplication>
#include <iostream>
#include <QList>
#include <QListIterator>
#include <QMutableListIterator>

struct MyStruct
{
    qint32 uid;
    QString uname;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QList<MyStruct> ptr;
    MyStruct str_ptr;

    str_ptr.uid = 1001;
    str_ptr.uname = "admin";
    ptr.append(str_ptr);

    str_ptr.uid = 1002;
    str_ptr.uname = "guest";
    ptr.append(str_ptr);

    // 使用传统方式遍历数据
    for(qint32 x=0;x<ptr.count();x++)
    {
        std::cout << ptr.at(x).uid << std::endl;
        std::cout << ptr[x].uname.toStdString().data() << std::endl;
    }

    // 使用只读迭代器遍历
    QListIterator<MyStruct> x(ptr);
    while(x.hasNext())
    {
        // peeknext读取下一个节点,但不影响指针变化
        std::cout << x.peekNext().uid << std::endl;
        std::cout << (x.peekNext().uname).toStdString().data() << std::endl;
        // 最后将x指针指向下一个数据
        x.next();
    }

    // 使用读写迭代器:如果uid=1002则将guest改为lyshark
    QMutableListIterator<MyStruct> y(ptr);
    while(y.hasNext())
    {
        // y.peekNext().uid = 9999;
        if(y.peekNext().uid == 1002)
        {
            y.peekNext().uname = "lyshark";
        }
        y.next();
    }
    return a.exec();
}
```

**qlinklist:** qlinklist就是动态链表结构，数据的存储非连续，访问时无法直接使用下标定位，只能通过迭代器迭代寻找，参数定义与qlist基本一致。
```C
#include <QCoreApplication>
#include <iostream>
#include <QLinkedList>
#include <QLinkedListIterator>
#include <QMutableLinkedListIterator>

struct MyStruct
{
    qint32 uid;
    QString uname;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QLinkedList<MyStruct> ptr;
    MyStruct str_ptr;

    str_ptr.uid = 1001;
    str_ptr.uname = "admin";
    ptr.append(str_ptr);

    str_ptr.uid = 1002;
    str_ptr.uname = "guest";
    ptr.append(str_ptr);


    // 使用只读迭代器遍历: 从前向后遍历
    QLinkedListIterator<MyStruct> x(ptr);
    while(x.hasNext())
    {
        std::cout << x.peekNext().uid << std::endl;
        x.next();
    }

    // 使用只读迭代器遍历: 从后向前遍历
    for(x.toBack();x.hasPrevious();x.previous())
    {
        std::cout << x.peekPrevious().uid << std::endl;

    }

    // 使用STL风格的迭代器遍历
    QLinkedList<MyStruct>::iterator y;
    for(y=ptr.begin();y!=ptr.end();++y)
    {
        std::cout << (*y).uid << std::endl;
    }

    // STL风格的只读迭代器
    QLinkedList<MyStruct>::const_iterator z;
    for(z=ptr.constBegin();z!=ptr.constEnd();++z)
    {
        std::cout <<((*z).uname).toStdString().data()<< std::endl;
    }


    // 使用读写迭代器: 动态生成列表,每次对二取余
    QLinkedList<int> Number = {1,2,3,4,5,6,7,8,9,10};
    QMutableLinkedListIterator<int> item(Number);

    // --> 从前向后输出一次
    for(item.toFront();item.hasNext();item.next())
        std::cout << item.peekNext() << std::endl;

    // --> 将指针移动到最后然后判断
    for(item.toBack();item.hasPrevious();)
    {
        if(item.previous() % 2==0)
            item.remove();
        else
            item.setValue(item.peekNext() * 10);
    }
    // --> 最后输出出相加后的结果
    for(item.toFront();item.hasNext();)
    {
        std::cout << item.peekNext() << std::endl;
        item.next();
    }
    return a.exec();
}
```

**QVector:** 该容器在相邻内存中存储连续的数据，该方式的使用与Qlist完全一致，但性能要比Qlist更高，但在插入时速度最慢。
```C
#include <QCoreApplication>
#include <iostream>
#include <QVector>
#include <QVectorIterator>
#include <QMutableVectorIterator>

struct MyStruct
{
    qint32 uid;
    QString uname;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QVector<MyStruct> ptr;
    MyStruct str_ptr;

    str_ptr.uid = 1001;
    str_ptr.uname = "admin";
    ptr.append(str_ptr);

    str_ptr.uid = 1002;
    str_ptr.uname = "guest";
    ptr.append(str_ptr);

    // 使用传统方式遍历
    for(qint32 x=0;x<ptr.count();x++)
    {
        std::cout << ptr.at(x).uid << std::endl;
        std::cout << ptr[x].uname.toStdString().data() << std::endl;
    }

    // 使用只读迭代器遍历: C++ STL写法
    QVector<MyStruct>::const_iterator item;
    for(item = ptr.begin();item != ptr.end(); ++item)
    {
        std::cout << (*item).uid << std::endl;
        std::cout << (*item).uname.toStdString().data() << std::endl;
    }

    // 使用读写迭代器修改: C++ STL写法
    QVector<MyStruct>::iterator write_item;
    for(write_item = ptr.begin();write_item !=ptr.end();++write_item)
    {
        if((*write_item).uid == 1001)
        {
            (*write_item).uname = "xxxx";
        }
        std::cout << (*write_item).uid << std::endl;
        std::cout << (*write_item).uname.toStdString().data() << std::endl;
    }

    return a.exec();
}
```

**QStack:** qstack时堆栈结构先进后出。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QStack>
#include <QQueue>

struct MyStruct
{
    qint32 uid;
    QString uname;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 定义并弹出QString类型数据
    QStack<QString> stack;

    stack.push("admin");
    stack.push("guest");

    std::cout << (stack.top()).toStdString().data()<<std::endl;
    while(!stack.isEmpty())
    {
        std::cout << (stack.pop()).toStdString().data() << std::endl;
    }

    // 定义并弹出一个结构类型数据
    QStack<MyStruct> struct_stack;
    MyStruct ptr;

    ptr.uid = 1001;
    ptr.uname = "admin";
    struct_stack.push(ptr);

    ptr.uid = 1002;
    ptr.uname = "guest";
    struct_stack.push(ptr);

    // 分别弹出数据并输出
    while(!struct_stack.isEmpty())
    {
        MyStruct ref;

        ref = struct_stack.pop();
        std::cout << "uid = " << ref.uid << std::endl;
        std::cout << "uname = " << ref.uname.toStdString().data() << std::endl;
    }

    return a.exec();
}
```

**QQueue:** qqueue 队列，先进先出。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QQueue>

struct MyStruct
{
    qint32 uid;
    QString uname;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QQueue<MyStruct> ptr;
    MyStruct queue_ptr;

    // 实现对结构体的入队
    queue_ptr.uid = 1001;
    queue_ptr.uname = "admin";
    ptr.enqueue(queue_ptr);

    queue_ptr.uid = 1002;
    queue_ptr.uname = "guest";
    ptr.enqueue(queue_ptr);

    // 实现对结构体的出队
    while(!ptr.isEmpty())
    {
        MyStruct ref;

        ref = ptr.dequeue();
        std::cout << "uid = " << ref.uid << std::endl;
        std::cout << "uname = " << ref.uname.toStdString().data() << std::endl;
    }

    return a.exec();
}
```
<br>

### 关联容器

关联容器: qmap，qmultimap，qhash，qmultihash，qmultihash，qset

**qmap/qmultimap:** 提供了一个字典类型的关联数组，一个键映射一个值，qmap是按照顺序存储的，如果不在意顺序可以使用qhash，使用qhash效率更高。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QtGlobal>
#include <QMap>
#include <QMapIterator>

struct MyStruct
{
    qint32 uid;
    QString uname;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<QString,QString> map;

    map["1001"] = "admin";
    map["1002"] = "guest";
    map.insert("1003","lyshark");
    map.insert("1004","lucy");
    // map.remove("1002");

    // 根据键值对查询属性
    std::cout << map["1002"].toStdString().data() << std::endl;
    std::cout << map.value("1003").toStdString().data() << std::endl;
    std::cout << map.key("admin").toStdString().data() << std::endl;

    // 使用STL语法迭代枚举Map键值对
    QMap<QString,QString>::const_iterator x;
    for(x=map.constBegin();x != map.constEnd(); ++x)
    {
        std::cout << x.key().toStdString().data() << " : ";
        std::cout << x.value().toStdString().data() << std::endl;
    }

    // 使用STL语法实现修改键值对
    QMap<QString,QString>::iterator write_x;
    write_x = map.find("1003");
    if(write_x !=map.end())
        write_x.value()= "you ary in";

    // 使用QTglobal中自带的foreach遍历键值对
    QString each;

    // --> 单循环遍历
    foreach(const QString &each,map.keys())
    {
        std::cout << map.value(each).toStdString().data() << std::endl;
    }

    // --> 多循环遍历
    foreach(const QString &each,map.uniqueKeys())
    {
        foreach(QString x,map.value(each))
        {
            std::cout << each.toStdString().data() << " : ";
            std::cout << x.toStdString().data() << std::endl;
        }
    }

    return a.exec();
}
``` 
qmultimap是qmap的子集，用于处理多值映射的类。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QList>
#include <QMultiMap>


struct MyStruct
{
    qint32 uid;
    QString uname;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMultiMap<QString,QString> mapA,mapB,mapC,mapD;

    mapA.insert("lyshark","1000");
    mapA.insert("lyshark","2000");
    mapB.insert("admin","3000");
    mapB.insert("admin","4000");
    mapC.insert("admin","5000");

    // 获取到里面的所有key=lyshark的值
    QList<QString> ref;

    ref = mapA.values("lyshark");
    for(int x=0;x<ref.size();++x)
    {
        std::cout << ref.at(x).toStdString().data() << std::endl;
    }

    // 两个key相同可相加后输出
    mapD = mapB + mapC;

    ref = mapD.values("admin");
    for(int x=0;x<ref.size();x++)
    {
        std::cout << ref.at(x).toStdString().data() << std::endl;
    }

    return a.exec();
}
```
qhash使用上与qmap相同，但qhash效率更高，唯一的不同时qhash不排序，qmap自动排序.

**qset:** qset 集合容器，是基于散列表的集合模板，存储顺序不定，查找速度最快，内部使用qhash实现。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QSet>

struct MyStruct
{
    qint32 uid;
    QString uname;
};

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QSet<QString> set;

    set << "dog" << "cat" << "tiger";

    // 测试某值是否包含于集合
    if(set.contains("cat"))
    {
        std::cout << "include" << std::endl;
    }

    return a.exec();
}
```
将qlist与qmap结合使用，实现嵌套 , 在qmap中存储一个qlist数据。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QtGlobal>
#include <QList>
#include <QMap>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<QString,QList<float>> map;
    QList<float> ptr;

    // 指定第一组数据
    ptr.append(10.1);
    ptr.append(12.5);
    ptr.append(22.3);
    map["10:10"] = ptr;

    // 指定第二组数据
    ptr.clear();
    ptr.append(102.2);
    ptr.append(203.2);
    ptr.append(102.1);
    map["11:20"] = ptr;

    // 输出所有的数据
    QList<float> tmp;
    foreach(QString each,map.uniqueKeys())
    {
        tmp = map.value(each);
        std::cout << "Time: " << each.toStdString().data() << std::endl;
        for(qint32 x=0;x<tmp.count();x++)
        {
            std::cout << tmp[x]<< std::endl;
        }
    }

    return a.exec();
}
```
将两个qlist合并为一个qmap，将列表合并为一个字典。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QtGlobal>
#include <QList>
#include <QMap>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QList<QString> Header = {"MemTotal","MemFree","Cached","SwapTotal","SwapFree"};
    QList<float> Values = {12.5,46.8,68,100.3,55.9,86.1};
    QMap<QString,float> map;

    // 将列表合并为一个字典
    for(int x=0;x<Header.count();x++)
    {
        QString head = Header[x].toStdString().data();
        float val = Values[x];
        map[head] = val;
    }

    // 输出特定字典中的数据
    std::cout << map.key(100.3).toStdString().data() << std::endl;
    std::cout << map.value("SwapTotal") << std::endl;

    return a.exec();
}
```
反之，将字典拆分为一个列表。
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QtGlobal>
#include <QList>
#include <QMap>

void Display(QMap<QString,float> map)
{
    foreach(const QString &each,map.uniqueKeys())
    {
        std::cout << each.toStdString().data() << std::endl;
        std::cout << map.value(each) << std::endl;
    }
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    QMap<QString,float> map;

    map["MemTotal"] = 12.5;
    map["MemFree"] = 32.1;
    map["Cached"] = 19.2;

    Display(map);

    QList<QString> map_key;
    QList<float> map_value;

    // 分别存储起来
    map_key = map.keys();
    map_value = map.values();

    // 输出所有的key值
    for(int x=0;x<map_key.count();x++)
    {
        std::cout << map_key[x].toStdString().data() << std::endl;
    }

    // 输出所有的value值
    for(int x=0;x<map_value.count();x++)
    {
        std::cout << map_value[x] << std::endl;
    }

    return a.exec();
}
```

**排序结构体：**
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QtGlobal>
#include <QList>

struct MyStruct
{
    int uuid;
    QString uname;
};

void Display(QList<int> ptr)
{
    foreach(const int &each,ptr)
        std::cout << each << " ";
    std::cout << std::endl;
}

// 由大到小排列
int compare(const int &infoA,const int &infoB)
{
    return infoA > infoB;
}

// 针对结构体的排序方法
void devListSort(QList<MyStruct> *list)
{
    std::sort(list->begin(),list->end(),[](const MyStruct &infoA,const MyStruct &infoB)
    {
        return infoA.uuid < infoB.uuid;
    });
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 定义并对单一数组排序
    QList<int> list = {56,88,34,61,79,82,34,67,88,1};
    std::sort(list.begin(),list.end(),compare);
    Display(list);

    // 定义并对结构体排序
    QList<MyStruct> list_struct;
    MyStruct ptr;

    ptr.uuid=1005;
    ptr.uname="admin";
    list_struct.append(ptr);

    ptr.uuid=1002;
    ptr.uname = "guest";
    list_struct.append(ptr);

    ptr.uuid = 1000;
    ptr.uname = "lyshark";
    list_struct.append(ptr);

    devListSort(&list_struct);

    for(int x=0;x< list_struct.count();x++)
    {
        std::cout << list_struct[x].uuid << " ---> ";
        std::cout << list_struct[x].uname.toStdString().data() << std::endl;
    }

    return a.exec();
}
```

**正则表达式模块：**
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QtGlobal>

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 返回绝对值
    std::cout << qAbs(10.5) << std::endl;

    // 返回较大者
    std::cout << qMax(12,56) << std::endl;

    // 返回较小者
    std::cout << qMin(22,56) << std::endl;

    // 返回随机数
    double x=6.7,y=3.5;
    int ref = qRound(x);
    std::cout << ref << std::endl;

    // 交换位置
    qSwap(x,y);
    std::cout << x << std::endl;

    return a.exec();
}
```
