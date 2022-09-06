SqlTableModel 组件可以将数据库中的特定字段动态显示在`TableView`表格组件中，通常设置`QSqlTableModel`类的变量作为数据模型后就可以显示数据表内容，界面组件中则通过`QDataWidgetMapper`类实例设置为与某个数据库字段相关联，则可以实现自动显示字段的内容，不仅是显示，其还支持`动态增删改查`等各种复杂操作，期间不需要使用任何SQL语句。

首先绘制好UI界面,本次案例界面稍显复杂，左侧是一个`TableView`组件，其他地方均为`LineEdit`组件与`Button`组件。

![](/image/1379525-20211209103258170-1850862982.png)

先来生成数据库表记录，此处我们只需要增加一个`Student`学生表，并插入两条测试数据即可，运行以下代码完成数据创建。
```C
#include <iostream>
#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QSqlRecord>
#include <QtSql>
#include <QDataWidgetMapper>

QSqlDatabase DB;                     // 数据库连接
void MainWindow::InitSQL()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
     if (!db.open())
     {
            return;
     }

    // 执行SQL创建表
    // https://Www.cnbloGs.com/lyshark
    db.exec("DROP TABLE Student");
    db.exec("CREATE TABLE Student ("
                    "id INTEGER PRIMARY KEY AUTOINCREMENT, "
                    "name VARCHAR(40) NOT NULL, "
                    "sex VARCHAR(40) NOT NULL, "
                    "age INTEGER NOT NULL,"
                    "mobile VARCHAR(40) NOT NULL,"
                    "city VARCHAR(40) NOT NULL)"
         );

    // 逐条插入数据
    db.exec("INSERT INTO Student(name,sex,age,mobile,city) ""VALUES ('lyshark.cnblogs.com','m','25','1234567890','beijing')");
    db.exec("INSERT INTO Student(name,sex,age,mobile,city) ""VALUES ('www.lyshark.com','x','22','4567890987','shanghai')");

    db.commit();
    db.close();
}
```

数据库创建后表内记录如下:

![](/image/1379525-20211209104227298-853484863.png)

程序运行后我们将在`MainWindow::MainWindow(QWidget *parent)`构造函数内完成数据库表记录与`TableView`组件字段的对应关系绑定，将数据库绑定到`QDataWidgetMapper`对象上，绑定代码如下。
```C
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 打开数据库
    DB=QSqlDatabase::addDatabase("QSQLITE"); // 添加 SQL LITE数据库驱动
    DB.setDatabaseName("./lyshark.db");     // 设置数据库名称
    if (!DB.open())
    {
        return;
    }

    // 打开数据表
    tabModel=new QSqlTableModel(this,DB);                             // 数据表
    tabModel->setTable("Student");                                    // 设置数据表
    tabModel->setEditStrategy(QSqlTableModel::OnManualSubmit);        // 数据保存方式，OnManualSubmit , OnRowChange
    tabModel->setSort(tabModel->fieldIndex("id"),Qt::AscendingOrder); // 排序
    if (!(tabModel->select()))                                        // 查询数据
    {
       return;
    }

    // 设置字段名称
    tabModel->setHeaderData(tabModel->fieldIndex("id"),Qt::Horizontal,"Uid");
    tabModel->setHeaderData(tabModel->fieldIndex("name"),Qt::Horizontal,"Uname");
    tabModel->setHeaderData(tabModel->fieldIndex("sex"),Qt::Horizontal,"Usex");
    tabModel->setHeaderData(tabModel->fieldIndex("age"),Qt::Horizontal,"Uage");
    tabModel->setHeaderData(tabModel->fieldIndex("mobile"),Qt::Horizontal,"Umobile");
    tabModel->setHeaderData(tabModel->fieldIndex("city"),Qt::Horizontal,"Ucity");

    theSelection=new QItemSelectionModel(tabModel);                       // 关联选择模型
    ui->tableView->setModel(tabModel);                                    // 设置数据模型
    ui->tableView->setSelectionModel(theSelection);                       // 设置选择模型
    ui->tableView->setSelectionBehavior(QAbstractItemView::SelectRows);   // 行选择模式

    // 添加数据映射,将选中字段映射到指定编辑框中
    // https://www.cnblogs.com/lysharK
    dataMapper= new QDataWidgetMapper();
    dataMapper->setModel(tabModel);
    dataMapper->setSubmitPolicy(QDataWidgetMapper::AutoSubmit);

    dataMapper->addMapping(ui->lineEdit_name,tabModel->fieldIndex("name"));          // 设置映射字段
    dataMapper->addMapping(ui->lineEdit_mobile,tabModel->fieldIndex("mobile"));      // 第二个映射字段
    dataMapper->toFirst();                                                           // 默认选中首条映射记录

    // 绑定信号,当鼠标选择时,在底部编辑框中输出
    connect(theSelection,SIGNAL(currentRowChanged(QModelIndex,QModelIndex)),this,SLOT(on_currentRowChanged(QModelIndex,QModelIndex)));
    getFieldNames();
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

绑定成功后运行程序即可看到如下效果,数据库中的记录被映射到了组件内.

![](/image/1379525-20211209104508130-277623719.png)

当用户点击`TableView`组件内的某一行记录时，则触发`MainWindow::on_currentRowChanged`函数。

 - 执行获取name/mobile字段,并放入映射数据集中的 lineEdit编辑框中

```C
void MainWindow::on_currentRowChanged(const QModelIndex &current, const QModelIndex &previous)
{
    Q_UNUSED(previous);

    dataMapper->setCurrentIndex(current.row());      // 更细数据映射的行号
    int curRecNo=current.row();                      // 获取行号
    QSqlRecord  curRec=tabModel->record(curRecNo);   // 获取当前记录

    QString uname = curRec.value("name").toString();     // 取出数据
    QString mobile = curRec.value("mobile").toString();

    ui->lineEdit_name->setText(uname);                   // 设置到编辑框
    ui->lineEdit_mobile->setText(mobile);
}
```

运行效果如下:

![](/image/1379525-20211209110140286-641789757.gif)

增加插入与删除记录实现方法都是调用`TabModel`提供的默认函数，通过获取当前选中行号，并对该行号执行增删改查方法即可。
```C
// 新增一条记录
// https://www.cnblogS.com/lyshark
void MainWindow::on_pushButton_add_clicked()
{
    tabModel->insertRow(tabModel->rowCount(),QModelIndex());                 // 在末尾添加一个记录
    QModelIndex curIndex=tabModel->index(tabModel->rowCount()-1,1);          // 创建最后一行的ModelIndex
    theSelection->clearSelection();                                          // 清空选择项
    theSelection->setCurrentIndex(curIndex,QItemSelectionModel::Select);     // 设置刚插入的行为当前选择行

    int currow=curIndex.row();                                              // 获得当前行
    tabModel->setData(tabModel->index(currow,0),1000+tabModel->rowCount()); // 自动生成编号
    tabModel->setData(tabModel->index(currow,2),"M");                       // 默认为男
    tabModel->setData(tabModel->index(currow,3),"0");                       // 默认年龄0
}

// 插入一条记录
void MainWindow::on_pushButton_insert_clicked()
{
    QModelIndex curIndex=ui->tableView->currentIndex();
    int currow=curIndex.row();                                              // 获得当前行

    tabModel->insertRow(curIndex.row(),QModelIndex());
    tabModel->setData(tabModel->index(currow,0),1000+tabModel->rowCount()); // 自动生成编号

    theSelection->clearSelection();                                         // 清除已有选择
    theSelection->setCurrentIndex(curIndex,QItemSelectionModel::Select);
}

// 删除一条记录
void MainWindow::on_pushButton_delete_clicked()
{
    QModelIndex curIndex=theSelection->currentIndex();  // 获取当前选择单元格的模型索引
    tabModel->removeRow(curIndex.row());                // 删除最后一行
}

// 保存修改数据
void MainWindow::on_pushButton_save_clicked()
{
    bool res=tabModel->submitAll();
    if (!res)
    {
        std::cout << "save as ok" << std::endl;
    }
}

// 恢复原始状态
void MainWindow::on_pushButton_reset_clicked()
{
    tabModel->revertAll();
}
```

增删改查实现如下:

![](/image/1379525-20211209110445100-978356720.gif)

针对与排序与过滤的实现方式如下，同样是调用了标准函数。
```C
// 以Combox中的字段对目标 升序排列
void MainWindow::on_pushButton_ascending_clicked()
{
    tabModel->setSort(ui->comboBox->currentIndex(),Qt::AscendingOrder);
    tabModel->select();
}

// 以Combox中的字段对目标 降序排列
// https://www.Cnblogs.com/LyShark
void MainWindow::on_pushButton_descending_clicked()
{
    tabModel->setSort(ui->comboBox->currentIndex(),Qt::DescendingOrder);
    tabModel->select();
}

// 过滤出所有男记录
void MainWindow::on_pushButton_filter_man_clicked()
{
    tabModel->setFilter(" sex = 'M' ");
}
// 恢复默认过滤器
void MainWindow::on_pushButton_default_clicked()
{
    tabModel->setFilter("");
}
```

过滤效果如下所示:

![](/image/1379525-20211209111111901-192090476.gif)

批量修改某个字段，其实现原理是首先通过`i<tabModel->rowCount()`获取记录总行数，然后通过`aRec.setValue`设置指定字段数值，并最终`tabModel->submitAll()`提交到表格中。
```C
void MainWindow::on_pushButton_clicked()
{
    if (tabModel->rowCount()==0)
        return;

    for (int i=0;i<tabModel->rowCount();i++)
    {
        QSqlRecord aRec=tabModel->record(i);            // 获取当前记录
        aRec.setValue("age",ui->lineEdit->text());      // 设置数据
        tabModel->setRecord(i,aRec);
    }
    tabModel->submitAll();                              // 提交修改
}
```

循环修改实现效果如下:

![](/image/1379525-20211209111422876-1283825423.gif)

上方代码中，如果需要修改或增加特定行或记录我们只需要点击相应的按钮，并在选中行直接编辑即可实现向数据库中插入数据，而有时我们不希望通过在原表上操作，而是通过新建窗体并在窗体中完成增删改，此时就需要使用Dialog窗体并配合原生SQL语句来实现对记录的操作了。

以增加为例，主窗体中直接弹出增加选项卡，并填写相关参数，直接提交即可。
```C
// https://www.cnblogs.com/LyShark
void MainWindow::on_pushButton_insert_clicked()
{
    QSqlQuery query;
    query.exec("select * from Student where id =-1");    // 查询字段信息,是否存在
    QSqlRecord curRec=query.record();                    // 获取当前记录,实际为空记录
    curRec.setValue("id",qryModel->rowCount()+1001);

    Dialog *WindowPtr=new Dialog(this);
    Qt::WindowFlags flags=WindowPtr->windowFlags();
    WindowPtr->setWindowFlags(flags | Qt::MSWindowsFixedSizeDialogHint); // 设置对话框固定大小
    WindowPtr->setInsertRecord(curRec);                                  // 插入记录
    int ret=WindowPtr->exec();                                           // 以模态方式显示对话框
    if (ret==QDialog::Accepted)                                          // OK键被按下
    {
        QSqlRecord  recData=WindowPtr->getRecordData();

        query.prepare("INSERT INTO Student(id,name,sex,age,mobile,city)"
                      " VALUES(:Id, :Name, :Sex, :Age, :Mobile, :City)");

        query.bindValue(":Id",recData.value("id"));
        query.bindValue(":Name",recData.value("name"));
        query.bindValue(":Sex",recData.value("sex"));
        query.bindValue(":Age",recData.value("age"));
        query.bindValue(":Mobile",recData.value("mobile"));
        query.bindValue(":City",recData.value("city"));

        if (query.exec())
        {
            QString sqlStr=qryModel->query().executedQuery(); // 执行过的SELECT语句
            qryModel->setQuery(sqlStr);                       // 重新查询数据
        }
    }
    delete WindowPtr;
}
```

Dialog增加效果如下:

![](/image/1379525-20211209125411334-60479430.gif)
