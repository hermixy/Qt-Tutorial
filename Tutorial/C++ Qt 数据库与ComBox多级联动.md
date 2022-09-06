Qt中的SQL数据库组件可以与`ComBox`组件形成多级联动效果，在日常开发中多级联动效果应用非常广泛，例如当我们选择指定用户时，我们让其在另一个`ComBox`组件中列举出该用户所维护的主机列表，又或者当用户选择省份时，自动列举出该省份下面的城市列表等。

今天给大家分享二级`ComBox`菜单如何与数据库形成联动，在进行联动之前需要创建两张表，表结构内容介绍如下:

 - User表：存储指定用户的ID号与用户名
 - UserAddressList表：与User表中的用户名相关联，存储该用户所管理的主机列表信息

```C
void InitMultipleSQL()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return;
     }

    // 执行SQL创建User表并插入测试数据
    // https://www.cnblogs.com/lyshark
    db.exec("DROP TABLE User");
    db.exec("CREATE TABLE User ("
                    "id INTEGER PRIMARY KEY AUTOINCREMENT, "
                    "name VARCHAR(40) NOT NULL)"
         );

    db.exec("INSERT INTO User(name) VALUES('lyshark')");
    db.exec("INSERT INTO User(name) VALUES('root')");
    db.exec("INSERT INTO User(name) VALUES('admin')");

    // 创建第二张表,与第一张表通过姓名关联起来
    db.exec("DROP TABLE UserAddressList");
    db.exec("CREATE TABLE UserAddressList("
            "id INTEGER PRIMARY KEY AUTOINCREMENT, "
            "name VARCHAR(40) NOT NULL, "
            "address VARCHAR(128) NOT NULL"
            ")");

    db.exec("INSERT INTO UserAddressList(name,address) VALUES ('lyshark','192.168.1.1')");
    db.exec("INSERT INTO UserAddressList(name,address) VALUES ('lyshark','192.168.1.2')");
    db.exec("INSERT INTO UserAddressList(name,address) VALUES ('lyshark','192.168.1.3')");
    db.exec("INSERT INTO UserAddressList(name,address) VALUES ('root','192.168.10.10')");
    db.exec("INSERT INTO UserAddressList(name,address) VALUES ('root','192.168.10.11')");
    db.exec("INSERT INTO UserAddressList(name,address) VALUES ('admin','192.168.100.100')");

    db.commit();
    db.close();
}
```
初始化表结构以后就得到了两张表，当程序运行时默认在构造函数处填充第一个`ComBox`组件，也就是执行一次数据库查询，并将结果通过`ui->comboBox_1->addItem();`放入到第一个组件内。
```C
QSqlDatabase db;

// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    InitMultipleSQL();

    db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return;
     }

     QSqlQuery query;
     query.exec("select * from User;");
     QSqlRecord rec = query.record();

     while(query.next())
     {
         int index_name = rec.indexOf("name");
         QString data_name = query.value(index_name).toString();
         ui->comboBox->addItem(data_name);
     }
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

代码运行后第一个ComBox会显示所有用户名:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211207145813790-316401904.gif)

此时回到UI编辑界面，我们在第一个ComBox上转到槽函数`on_comboBox_activated(const QString &arg1)`上面。

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211207150016357-721319935.png)

当用户选择第一个`ComBox`选择框时，自动查询数据库中与该选择框对应的字段，并关联到第二个选择框内，代码如下:
```C
void MainWindow::on_comboBox_activated(const QString &arg1)
{
    std::cout << "www.lyshark.com Name = " << arg1.toStdString()<<std::endl;

    if(db.open())
    {
        QSqlQuery query;
        query.prepare("select * from UserAddressList where name = :x");
        query.bindValue(":x",arg1);
        query.exec();

        QSqlRecord rec = query.record();

        ui->comboBox_2->clear();
        while(query.next())
        {
            int index = rec.indexOf("address");
            QString data_ = query.value(index).toString();
            ui->comboBox_2->addItem(data_);
        }
    }
}
```

最终关联效果如下，当选择用户是自动关联到所维护的主机列表上面:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211207150316084-287479371.gif)
