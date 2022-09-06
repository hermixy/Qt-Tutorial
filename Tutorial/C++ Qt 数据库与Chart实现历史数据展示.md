在前面的博文中具体介绍了QChart组件是如何绘制各种通用的二维图形的，本章内容将继续延申一个新的知识点，通过数据库存储某一段时间节点数据的走向，当用户通过编辑框提交查询记录时，程序自动过滤出该时间节点下所有的数据，并将该数据动态绘制到图形组件内，实现动态查询图形的功能。

首先通过如下代码，创建`Times`表，表内记录有某个主机某个时间节点下的数值:
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
#include <QDebug>
#include <QDateTime>
#include <QTime>

// 初始化数据库
// https://www.cnblogs.com/lyshark
void InitSql()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");

    db.setDatabaseName("lyshark.db");
    if (!db.open())
    {
           std::cout << db.lastError().text().toStdString()<< std::endl;
           return;
    }

   // 执行SQL创建表
   db.exec("DROP TABLE Times");
   db.exec("CREATE TABLE Times ("
                   "id INTEGER PRIMARY KEY AUTOINCREMENT, "
                   "address VARCHAR(64) NOT NULL, "
                   "datetime VARCHAR(128) NOT NULL, "
                   "value INTEGER NOT NULL"
           ")"
        );

   db.commit();
   db.close();
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    InitSql();
    return a.exec();
}
```

数据库结构如下:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211211153357049-1422271476.png)

接着编写一个模拟插入数据的案例，该案例每一秒向数据库内插入一条记录，我们运行一段时间。
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
#include <QDebug>
#include <QDateTime>
#include <QTime>

// 延时函数
void Sleep(int msec)
{
    QTime dieTime = QTime::currentTime().addMSecs(msec);
    while(QTime::currentTime() < dieTime)
        QCoreApplication::processEvents(QEventLoop::AllEvents,100);
}
// 生成随机数
int GetRandom()
{
    int num = qrand() % 100;
    return num;
}

// 插入数据
void InsertSQL()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return;
     }

     for(int index=0;index <99999;index++)
     {
        QString address = QString("192.168.1.100");
        QDateTime curDateTime = QDateTime::currentDateTime();
        QString date_time = curDateTime.toString("yyyy-MM-dd hh:mm:ss");
        int value = GetRandom();

        QString run_sql = QString("INSERT INTO Times(id,address,datetime,value) VALUES (%1,'%2','%3',%4);")
                                  .arg(index).arg(address).arg(date_time).arg(value);
        std::cout << "执行插入语句: " << run_sql.toStdString() << std::endl;

        db.exec(run_sql);
        db.commit();
        Sleep(1000);
     }
     db.close();
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    qsrand(QTime(0,0,0).secsTo(QTime::currentTime()));
    InsertSQL();
    return a.exec();
}
```

运行插入程序，统计一段时间 从 `2021-12-11 15:34:16` 到 `2021-12-11 15:40:04` 停止，表内记录如下:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211211154113377-1732323193.png)

如果我们需要查询某一个时间节点下的数据，例如查询`2021-12-11 15:35:00 - 2021-12-11 15:37:00`的数据可以这样写SQL:
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
#include <QDebug>
#include <QDateTime>
#include <QTime>

// 输出数据
// https://www.cnblogs.com/lyshark
void SelectSQL()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("lyshark.db");
     if (!db.open())
     {
            std::cout << db.lastError().text().toStdString()<< std::endl;
            return;
     }

    // 查询数据
    QSqlQuery query("SELECT * FROM Times;",db);
    QSqlRecord rec = query.record();

    // 循环所有记录
    while(query.next())
    {
        // 判断当前记录是否有效
        if(query.isValid())
        {
            int id_value = query.value(rec.indexOf("id")).toInt();
            QString address_value = query.value(rec.indexOf("address")).toString();
            QString date_time = query.value(rec.indexOf("datetime")).toString();
            int this_value = query.value(rec.indexOf("value")).toInt();

            if(date_time.toStdString() >= "2021-12-11 15:35:00" && date_time.toStdString() <="2021-12-11 15:37:00")
            {
                std::cout << "value: " << this_value << std::endl;
            }
        }
    }
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
    SelectSQL();
    return a.exec();
}
```

这样就可以将该区间内所有的数据全部过滤出来了:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211211154358303-1588780205.png)

将过滤参数与`QChart`组件结合即可实现动态绘图效果，绘制UI界面如下:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211211155306964-1273840321.png)

当用户点击查询时，直接从数据库内取出数据，并将其动态更新到Chart组件内即可，实现代码如下:
```C
#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <iostream>
#include <QStringList>
#include <QString>
#include <QVariant>
#include <QDebug>
#include <QDateTime>
#include <QTime>

// 初始化Chart图表
void MainWindow::InitChart()
{
    // 创建图表的各个部件
    QChart *chart = new QChart();

    // 将Chart添加到ChartView
    ui->graphicsView->setChart(chart);
    ui->graphicsView->setRenderHint(QPainter::Antialiasing);

    // 隐藏图例
    chart->legend()->hide();

    // 设置图表主题色
    ui->graphicsView->chart()->setTheme(QChart::ChartTheme(1));

    // 创建曲线序列
    QLineSeries *series0 = new QLineSeries();

    // 序列添加到图表
    chart->addSeries(series0);

    // 创建坐标轴
    QValueAxis *axisX = new QValueAxis;    // X轴
    axisX->setRange(1, 100);               // 设置坐标轴范围
    axisX->setLabelFormat("%d %");         // 设置X轴格式
    axisX->setMinorTickCount(5);           // 设置X轴刻度

    QValueAxis *axisY = new QValueAxis;    // Y轴
    axisY->setRange(0, 100);               // Y轴范围
    axisY->setMinorTickCount(4);           // s设置Y轴刻度

    // 设置X于Y轴数据集
    chart->setAxisX(axisX, series0);   // 为序列设置坐标轴
    chart->setAxisY(axisY, series0);
}

// 为序列生成数据
void MainWindow::SetData()
{
    // 获取指针
    QLineSeries *series0=(QLineSeries *)ui->graphicsView->chart()->series().at(0);

    // 清空图例
    series0->clear();

    // 链接数据库
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("lyshark.db");
    if (!db.open())
    {
        return;
    }

    // 查询数据
    QSqlQuery query("SELECT * FROM Times;",db);
    QSqlRecord rec = query.record();

    // 赋予数据
    qreal t=0,intv=1;

    // 循环所有记录
    while(query.next())
    {
        // 判断当前记录是否有效
        // https://www.cnblogs.com/lyshark
        if(query.isValid())
        {
            QString address_value = query.value(rec.indexOf("address")).toString();
            QString date_time = query.value(rec.indexOf("datetime")).toString();
            int this_value = query.value(rec.indexOf("value")).toInt();

            // 获取组件字符串
            QString start_user_time = ui->dateTimeEdit_Start->text();
            QString end_user_time = ui->dateTimeEdit_End->text();

            // 将时间字符串转为秒,并计算差值 (秒为单位)
            QDateTime start_timet = QDateTime::fromString(start_user_time, "yyyy-MM-dd hh:mm:ss");
            QDateTime end_timet = QDateTime::fromString(end_user_time, "yyyy-MM-dd hh:mm:ss");

            uint stime = start_timet.toTime_t();
            uint etime = end_timet.toTime_t();

            // 只允许查询小于180秒的记录
            uint sub_time = etime - stime;
            if(sub_time <= 180)
            {
                // 查询指定区间内的数据
                if(date_time.toStdString() >= start_user_time.toStdString() && date_time.toStdString() <= end_user_time.toStdString())
                {
                    // std::cout << "区间内的数据: " << this_value << std::endl;
                    series0->append(t,this_value);
                    t+=intv;
                }
            }
            else
            {
                std::cout << "查询范围超出定义." << std::endl;
                return;
            }
        }
    }
}

// 将添加的widget控件件提升为QChartView类
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    InitChart();

    // 初始化时间组件
    QDateTime curDateTime = QDateTime::currentDateTime();

    // 设置当前时间
    ui->dateTimeEdit_Start->setDateTime(curDateTime);
    ui->dateTimeEdit_End->setDateTime(curDateTime);

    // 设置时间格式
    ui->dateTimeEdit_Start->setDisplayFormat("yyyy-MM-dd hh:mm:ss");
    ui->dateTimeEdit_End->setDisplayFormat("yyyy-MM-dd hh:mm:ss");
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_pushButton_clicked()
{
    SetData();
}
```

查询效果如下所示:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211211160318489-415725413.gif)
