在上一篇博文中详细介绍了`SqlTableModle`组件是如何使用的，本篇博文将介绍`SqlRelationalTable`关联表组件，该组件其实是`SqlTableModle`组件的扩展类，`SqlRelationalTable`组件可以关联某个主表中的外键，例如将主表中的某个字段与附加表中的特定字段相关联起来，`QSqlRelation(关联表名,关联ID,名称)`就是用来实现多表之间快速关联的。

首先我们创建两张表，一张`Student`表存储学生名字以及学生课程号，另一张`Departments`存储每个编号所对应的系所名称，运行代码完成创建。
```C
void MainWindow::InitSQL()
{
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
    if (!db.open())
           return;

    // 执行SQL创建表
    db.exec("DROP TABLE Student");
    db.exec("CREATE TABLE Student ("
                   "id INTEGER PRIMARY KEY AUTOINCREMENT, "
                   "name VARCHAR(40) NOT NULL, "
                   "departID INTEGER NOT NULL)"
            );

    // 逐条插入数据
    db.exec("INSERT INTO Student(name,departID) VALUES('zhangsan',10)");
    db.exec("INSERT INTO Student(name,departID) VALUES('lisi',20)");
    db.exec("INSERT INTO Student(name,departID) VALUES('wangwu',30)");
    db.exec("INSERT INTO Student(name,departID) VALUES('wangmazi',40)");

    db.exec("DROP TABLE Departments");
    db.exec("CREATE TABLE Departments("
            "departID INTEGER NOT NULL,"
            "department VARCHAR(40) NOT NULL)"
            );

    db.exec("INSERT INTO Departments(departID,department) VALUES (10,'数学学院')");
    db.exec("INSERT INTO Departments(departID,department) VALUES (20,'物理学院')");
    db.exec("INSERT INTO Departments(departID,department) VALUES (30,'计算机学院')");
    
    db.commit();
    db.close();
}
```

初始化后将得到两张数据表，这两张表通过`departID`相关联，如下:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211209173024588-904016058.gif)

创建完成后，我们在程序的构造函数直接实现绑定即可，这段代码很简单如下:
```C
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    InitSQL();

    // 打开数据库
    QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
    db.setDatabaseName("./lyshark.db");
    if (!db.open())
        return;

    this->setCentralWidget(ui->tableView);
    ui->tableView->setSelectionBehavior(QAbstractItemView::SelectItems);
    ui->tableView->setSelectionMode(QAbstractItemView::SingleSelection);
    ui->tableView->setAlternatingRowColors(true);

    // 打开数据表
    tabModel=new QSqlRelationalTableModel(this,DB);
    tabModel->setTable("Student");                              // 设置数据表
    tabModel->setEditStrategy(QSqlTableModel::OnManualSubmit);  // OnManualSubmit
    tabModel->setSort(0,Qt::AscendingOrder);

    tabModel->setHeaderData(0,Qt::Horizontal,"学号");
    tabModel->setHeaderData(1,Qt::Horizontal,"姓名");
    tabModel->setHeaderData(2,Qt::Horizontal,"学院");

    // 设置代码字段的查询关系数据表
    // 打开Departments表,关联ID和department
    tabModel->setRelation(2,QSqlRelation("Departments","departID","department"));
    theSelection=new QItemSelectionModel(tabModel);

    ui->tableView->setModel(tabModel);
    ui->tableView->setSelectionModel(theSelection);
    ui->tableView->setItemDelegate(new QSqlRelationalDelegate(ui->tableView)); // 为关系型字段设置缺省代理组件

    tabModel->select();                                                        // 打开数据表
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

最终绑定效果如下图:

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211209173215373-690889321.gif)
