在上一篇博文`《C/C++ Qt 数据库QSql增删改查组件应用》`介绍了Qt中如何使用SQL操作函数，并实现了对数据库的增删改查等基本功能，从本篇开始将实现数据库与View组件的绑定，通过数据库与组件关联可实现动态展示数据库中的表记录。

我们先以`TreeView`组件为例，简单介绍一下如何实现组件与数据的绑定，首先我们需要创建一个表并插入几条测试记录，运行如下代码实现建库建表.
```C
#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <iostream>
#include <QStringList>
#include <QString>
#include <QVariant>

#include <QDataWidgetMapper>
#include <QtSql>

void Init()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return;
     }

    // 执行SQL创建表
    db.exec("DROP TABLE LyShark");
    db.exec("CREATE TABLE LyShark ("
                    "id INTEGER PRIMARY KEY AUTOINCREMENT, "
                    "name VARCHAR(40) NOT NULL, "
                    "age INTEGER NOT NULL)"
         );

    // 逐条插入
    db.exec("INSERT INTO LyShark(name,age) VALUES('admin',22)");
    db.exec("INSERT INTO LyShark(name,age) VALUES('lyshark',25)");
    db.exec("INSERT INTO LyShark(name,age) VALUES('zhangsan',22)");
    db.exec("INSERT INTO LyShark(name,age) VALUES('wangwu',22)");

    db.commit();
}

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    Init();
}
```

执行建库建表后,数据库内记录如下:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211207104211264-685453126.png)

有了数据表以后，接着就需要将数据表中的记录与View组件进行绑定，绑定组件首先需要调用`QSqlQueryModel`查询数据表中的记录，当查询到记录以后，调用`QItemSelectionModel()`将该记录绑定到对应的模型中，最后调用`ui->treeView->setModel(qryModel);`以及`ui->treeView->setSelectionModel(theSelection);`将该模型显示在`TreeView`组件内，这段代码如下:
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <iostream>
#include <QStringList>
#include <QString>
#include <QVariant>

#include <QDataWidgetMapper>
#include <QtSql>

#include <QStandardItem>
#include <QStringList>
#include <QStringListModel>

// 定义数据模型指针
QSqlQueryModel *qryModel;          // 数据模型
QItemSelectionModel *theSelection; // 选择模型
QDataWidgetMapper *dataMapper;     // 数据界面映射

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return;
     }

     // 查询数据表中记录
     qryModel=new QSqlQueryModel(this);
     qryModel->setQuery("SELECT * FROM LyShark ORDER BY id");
     if (qryModel->lastError().isValid())
     {
         return;
     }

     // 设置TableView表头数据
     qryModel->setHeaderData(0,Qt::Horizontal,"ID");
     qryModel->setHeaderData(1,Qt::Horizontal,"Name");
     qryModel->setHeaderData(2,Qt::Horizontal,"Age");

     // 将数据绑定到模型上
     theSelection=new QItemSelectionModel(qryModel);
     ui->treeView->setModel(qryModel);
     ui->treeView->setSelectionModel(theSelection);
     ui->treeView->setSelectionBehavior(QAbstractItemView::SelectRows);
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

运行代码后,程序会从数据库内取出结果并输出到`TreeView`组件上:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211207104540340-1274053622.png)
