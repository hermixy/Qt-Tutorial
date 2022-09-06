Qt 数据库组件与TableView组件实现联动，以下案例中实现了，当用户点击并选中TableView组件内的某一行时，我们通过该行中的name字段查询并将查询结果关联到`ListView`组件内，同时将TableView中选中行的字段分别显示在窗体底部的`LineEdit`编辑内，该案例具体实现细节如下。

首先在UI界面中绘制好需要的控件，左侧放一个`TableView`组件，右侧是一个`ListView`组件，底部放三个`LineEdit`组件，界面如下:

![](/image/1379525-20211207161548479-880279341.png)

我们还是需要创建两张表结构，表`Student`用于存储学生的基本信息，表`StudentTimetable`存储的是每个学生所需要学习的课程列表，执行后创建数据表。
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

    // 执行SQL创建表
    db.exec("DROP TABLE Student");
    db.exec("CREATE TABLE Student ("
                    "id INTEGER PRIMARY KEY AUTOINCREMENT, "
                    "name VARCHAR(40) NOT NULL, "
                    "age INTEGER NOT NULL)"
         );

    // 批量创建数据
    // https://www.cnblogs.com/lyshark
    QStringList name_list; name_list << "lyshark" << "lisi" << "wangwu";
    QStringList age_list; age_list << "25" << "34" << "45";

    // 绑定并插入数据
    QSqlQuery query;
    query.prepare("INSERT INTO Student(name,age) ""VALUES (:name, :age)");

    if(name_list.size() == age_list.size())
    {
        for(int x=0;x< name_list.size();x++)
        {
            query.bindValue(":name",name_list[x]);
            query.bindValue(":age",age_list[x]);
            query.exec();
        }
    }

    // ------------------------------------------------
    // 创建第二张表,与第一张表通过姓名关联起来
    db.exec("DROP TABLE StudentTimetable");
    db.exec("CREATE TABLE StudentTimetable("
            "id INTEGER PRIMARY KEY AUTOINCREMENT, "
            "name VARCHAR(40) NOT NULL, "
            "timetable VARCHAR(128) NOT NULL"
            ")");

    db.exec("INSERT INTO StudentTimetable(name,timetable) VALUES ('lyshark','AAA')");
    db.exec("INSERT INTO StudentTimetable(name,timetable) VALUES ('lyshark','BBB')");
    db.exec("INSERT INTO StudentTimetable(name,timetable) VALUES ('lyshark','CCC')");

    db.exec("INSERT INTO StudentTimetable(name,timetable) VALUES ('lisi','QQQ')");
    db.exec("INSERT INTO StudentTimetable(name,timetable) VALUES ('lisi','WWW')");

    db.exec("INSERT INTO StudentTimetable(name,timetable) VALUES ('wangwu','EEE')");

    db.commit();
    db.close();
}
```

程序运行后，构造函数`MainWindow::MainWindow(QWidget *parent)`内初始化表格，查询`Student`表内记录，将查询到的指针绑定到`theSelection`模型上，绑定后再将绑定指针加入到`dataMapper`组件映射中，即可实现初始化，其初始化代码如下:
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
     qryModel->setQuery("SELECT * FROM Student ORDER BY id");
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
     ui->tableView->setModel(qryModel);
     ui->tableView->setSelectionModel(theSelection);
     ui->tableView->setSelectionBehavior(QAbstractItemView::SelectRows);

     // 创建数据映射
     dataMapper= new QDataWidgetMapper();
     dataMapper->setSubmitPolicy(QDataWidgetMapper::AutoSubmit);
     dataMapper->setModel(qryModel);
     dataMapper->addMapping(ui->lineEdit_id,0);
     dataMapper->addMapping(ui->lineEdit_name,1);
     dataMapper->addMapping(ui->lineEdit_age,2);
     dataMapper->toFirst();

     // 绑定信号,当鼠标选择时,在底部编辑框中输出
     // https://www.cnblogs.com/lyshark
     connect(theSelection,SIGNAL(currentRowChanged(QModelIndex,QModelIndex)),this,SLOT(on_currentRowChanged(QModelIndex,QModelIndex)));
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

此时这个程序运行后会得到表内数据:

![](/image/1379525-20211207163030063-452460389.png)

接着我们需要绑定`TableView`表格的`on_currentRowChanged()`事件，当用户点击`TableView`表格中的某个属性是则自动触发该函数，在此函数内我们完成对其他组件的填充.
 - 1.通过`currentIndex`方法获取到当前表所在行
 - 2.通过当前行号查询表中姓名，并带入`StudentTimetable`表查该表中记录
 - 3.循环获取该用户的数据,并将`timetable`字段提取出来放入`QStringList`容器
 - 4.将数据直接关联到`ListView`数据表中

```C
// 鼠标点击后的处理槽函数
void MainWindow::on_currentRowChanged(const QModelIndex &current, const QModelIndex &previous)
{
    Q_UNUSED(previous);
    if (!current.isValid())
    {
        return;
    }

    dataMapper->setCurrentModelIndex(current);

    // 获取到记录开头结尾
    bool first=(current.row()==0);                    // 是否首记录
    bool last=(current.row()==qryModel->rowCount()-1);// 是否尾记录
    std::cout << "IsFirst: " << first << "IsLast: " << last << std::endl;


    // 获取name字段数据
    int curRecNo=theSelection->currentIndex().row();  // 获取当前行号
    QSqlRecord curRec=qryModel->record(curRecNo);     // 获取当前记录
    QString uname = curRec.value("name").toString();
    std::cout << "Student Name = " << uname.toStdString() << std::endl;

    // 查StudentTimetable表中所有数据
    // 根据姓名过滤出该用户的所有数据
    QSqlQuery query;
    query.prepare("select * from StudentTimetable where name = :x");
    query.bindValue(":x",uname);
    query.exec();

    // 循环获取该用户的数据,并将timetable字段提取出来放入QStringList容器
    // https://www.cnblogs.com/lyshark
    QSqlRecord rec = query.record();
    QStringList the_data;

    while(query.next())
    {
        int index = rec.indexOf("timetable");
        QString data = query.value(index).toString();

        std::cout << "User timetable = " << data.toStdString() << std::endl;
        the_data.append(data);
    }

    // 关联到ListView数据表中
    QStringListModel *model;
    model = new QStringListModel(the_data);
    ui->listView->setModel(model);
    ui->listView->setEditTriggers(QAbstractItemView::NoEditTriggers);
}
```

当绑定选中事件时,程序运行效果如下:

![](/image/1379525-20211207163617597-850645388.gif)

针对底部按钮处理事件相对来说较为简单，其实现原理就是调用了`TableView`默认提供的一些函数而已，代码如下:
```C
// 刷新tableView的当前选择行
// https://www.cnblogs.com/lyshark
void MainWindow::refreshTableView()
{
    int index=dataMapper->currentIndex();
    QModelIndex curIndex=qryModel->index(index,0);      // 定位到低0行0列
    theSelection->clearSelection();                     // 清空选择项
    theSelection->setCurrentIndex(curIndex,QItemSelectionModel::Select);//设置刚插入的行为当前选择行
}

// 第一条记录
void MainWindow::on_pushButton_clicked()
{
    dataMapper->toFirst();
    refreshTableView();
}
// 最后一条记录
void MainWindow::on_pushButton_2_clicked()
{
    dataMapper->toLast();
    refreshTableView();
}

// 前一条记录
void MainWindow::on_pushButton_3_clicked()
{
    dataMapper->toPrevious();
    refreshTableView();
}

// 下一条记录
void MainWindow::on_pushButton_4_clicked()
{
    dataMapper->toNext();
    refreshTableView();
}
```

最终运行效果如下所示:

![](/image/1379525-20211207164007889-1493554657.gif)
