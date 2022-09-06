Qt SQL模块是Qt中用来操作数据库的类，该类封装了各种SQL数据库接口，可以很方便的链接并使用，数据的获取也使用了典型的Model/View结构，通过MV结构映射我们可以实现数据与通用组件的灵活绑定，一般SQL组件常用的操作，包括，读取数据，插入数据，更新数据，删除数据，这四个功能我将分别介绍它是如何使用的。

SQL模块在使用时必须引入模块，需要在pro文件内增加`QT += sql`并在头文件内增加`#include <QSqlDatabase>`导入模块才可以正常使用。

![](/image/1379525-20211206142322949-324340331.png)

**初始化数据库:** 初始化调用`QSqlDatabase::addDatabase`指定数据库类型,通过`db.setDatabaseName()`指定数据库文件名.
```C
#include <QCoreApplication>
#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <iostream>
#include <QStringList>
#include <QString>
#include <QVariant>

// 初始化表结构
// https://www.cnblogs.com/lyshark
bool InitSQL()
{
    // 指定数据库驱动类型
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return false;
     }

    // 执行SQL创建表
    db.exec("DROP TABLE LyShark");
    db.exec("CREATE TABLE LyShark ("
                    "id INTEGER PRIMARY KEY AUTOINCREMENT, "
                    "name VARCHAR(40) NOT NULL, "
                    "age INTEGER NOT NULL)"
         );

    // 提交事务请求
    bool ref = db.commit();
    db.close();
    return ref;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    bool ref = InitSQL();
    std::cout << "init: " << ref << std::endl;

    return a.exec();
}
```

初始化表结构如下:

![](/image/1379525-20211206145131537-1841889280.png)

<br>

**逐条插入数据:** 逐条插入记录在Qt中可直接调用SQL模块提供的`db.exec()`函数,插入后最后需要调用`db.commit()`一次性提交事务.
```C
#include <QCoreApplication>
#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <iostream>
#include <QStringList>
#include <QString>
#include <QVariant>

// 逐条插入数据
// https://www.cnblogs.com/lyshark
bool InsertSQL()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return false;
     }

     // 逐条插入数据
     db.exec("INSERT INTO LyShark(name,age) ""VALUES ('admin',23)");
     db.exec("INSERT INTO LyShark(name,age) ""VALUES ('zhangsan',25)");
     db.exec("INSERT INTO LyShark(name,age) ""VALUES ('lisi',34)");

     // 提交数据
     bool ref = db.commit();
     return ref;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    bool ref = InsertSQL();
    std::cout << "insert = > " << ref << std::endl;

    return a.exec();
}
```

插入记录如下:

![](/image/1379525-20211206145525146-1240412551.png)

<br>

**多条记录插入:** 一次性插入多条数据记录,可调用`query.prepare()`绑定字段与SQL记录,绑定后即可通过循环批量插入记录.
```C
#include <QCoreApplication>
#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <iostream>
#include <QStringList>
#include <QString>
#include <QVariant>

// 批量插入数据
// https://www.cnblogs.com/lyshark
bool InsertMultipleSQL()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
    if (!db.open())
    {
           std::cout << db.lastError().text().toStdString()<< std::endl;
           return false;
    }

    // 定义字符串链表
    QStringList user_name;
    QStringList user_age;

    // 批量插入数据到链表中
    user_name << "www.lyshark.com" << "https://www.cnblogs.com/lyshark" << "lyshark";
    user_age << "22" << "33" << "44";

    // 绑定数据记录
    QSqlQuery query;
    query.prepare("INSERT INTO LyShark(name,age) ""VALUES (:name, :age)");

    // 判断两张表中字段数据量是否一致
    if(user_name.size() == user_age.size())
    {
        // 循环插入数据
        for(int x=0;x< user_name.size(); x++)
        {
            query.bindValue(":name",user_name[x]);
            query.bindValue(":age",user_age[x]);
            query.exec();
        }
    }
    // 提交数据
    db.commit();
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    bool ref = InsertMultipleSQL();
    std::cout << "insert = > " << ref << std::endl;

    return a.exec();
}
```

批量插入数据如下:

![](/image/1379525-20211206150400597-2121821925.png)

<br>

**查询表中记录:** 查询记录可调用`QSqlQuery query()`得到记录条数,然后不断循环,每次循环调用一次`query.next()`获取一条,直到循环结束.
```C
#include <QCoreApplication>
#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <iostream>
#include <QStringList>
#include <QString>
#include <QVariant>

// 查询数据记录
// https://www.cnblogs.com/lyshark
bool SelectSQL()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return false;
     }

     // 查询数据
     QSqlQuery query("SELECT * FROM LyShark;",db);
     QSqlRecord rec = query.record();

     // 循环所有记录
     while(query.next())
     {
         // 判断当前记录是否有效
         if(query.isValid())
         {
             int id_ptr = rec.indexOf("id");
             int id_value = query.value(id_ptr).toInt();

             int name_ptr = rec.indexOf("name");
             QString name_value = query.value(name_ptr).toString();

             int age_ptr = rec.indexOf("age");
             int age_value = query.value(age_ptr).toInt();

             std::cout << "ID: " << id_value << "Name: " << name_value.toStdString() << "Age: " << age_value << std::endl;
         }
     }
     return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    bool ref = SelectSQL();
    std::cout << "select = > " << ref << std::endl;

    return a.exec();
}
```

循环输出的数据如下:

![](/image/1379525-20211206150400597-2121821925.png)

<br>

**更新表中记录:** 更新表中记录直接调用`update`语句即可,在调用之前通过`QString sql`拼接待修改语句并提交`db.commit()`事务即可完成更新.
```C
#include <QCoreApplication>
#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <iostream>
#include <QStringList>
#include <QString>
#include <QVariant>

// 修改表中数据
// https://www.cnblogs.com/lyshark
bool UpdateSQL(QString uid, QString new_name)
{
    // 指定数据库驱动类型
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return false;
     }

    // 执行SQL更新记录
    QString sql = QString("update LyShark set name='%1' where id=%2;").arg(new_name).arg(uid);
    db.exec(sql);
    std::cout << "update => " << sql.toStdString() << std::endl;

    // 提交事务请求
    bool ref = db.commit();
    db.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 更新ID=6的记录更新为 www.cnblogs.com/lyshark
    bool ref = UpdateSQL("6","www.cnblogs.com/lyshark");
    std::cout << "update flag: " << ref << std::endl;

    return a.exec();
}
```

更新后表中记录如下:

![](/image/1379525-20211206152828271-98954289.png)
