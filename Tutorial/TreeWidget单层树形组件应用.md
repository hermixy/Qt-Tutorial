TreeWidget 目录树组件，该组件适用于创建和管理目录树结构，在开发中我们经常会把它当作一个升级版的`ListView`组件使用，因为`ListView`每次只能显示一列数据集，而使用`TableWidget`组件显示多列显得不够美观，此时使用Tree组件显示单层结构是最理想的方式，本章博文将通过`TreeWidget`实现多字段显示，并增加一个自定义菜单，通过在指定记录上右键可弹出该菜单并对指定记录进行操作。

1.通过`TreeView`组件实现一个只读属性的树形目录，该目录中指定三个字段，分别用来表示`ID,IP地址,用户名`字段.

初始化Tree组件
 - 1.初始化并设置treeView属性
 - 2.设置列头长度
 - 3.设置列头数据
 - 4.设置表中元素

```C
#include <QSplitter>
#include <QTreeView>
#include <QTextCodec>
#include <QStandardItemModel>

// By: LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    QStandardItemModel *tree = new QStandardItemModel(0,3,this);

    // 设置treeView属性
    ui->treeView->setColumnWidth(0,300);                               // 设置最后一列宽度自适应
    ui->treeView->setIndentation(1);                                   // 设置表头缩进为1
    ui->treeView->setEditTriggers(QAbstractItemView::NoEditTriggers);  // 节点不可编辑

    // 设置列头长度
    ui->treeView->setColumnWidth(0,50);      // 设置第1列长度
    ui->treeView->setColumnWidth(1,200);     // 设置第2列长度
    ui->treeView->setColumnWidth(2,200);     // 设置第3列长度

    // 设置列头数据
    tree->setHeaderData(0, Qt::Horizontal, tr("ID"));
    tree->setHeaderData(1, Qt::Horizontal, tr("IP地址"));
    tree->setHeaderData(2, Qt::Horizontal, tr("用户"));

    ui->treeView->setModel(tree);           // 将表头设置到模型

    // 设置表中元素
    QList<QStandardItem *> ptr;

    QStandardItem *item_uid = new QStandardItem("1001");
    item_uid->setIcon(QIcon(":/image/1.ico"));
    ptr.push_back(item_uid);

    QStandardItem *item_addr = new QStandardItem("192.168.1.1");
    ptr.push_back(item_addr);

    QStandardItem *item_username = new QStandardItem("lyshark");
    ptr.push_back(item_username);
    tree->appendRow(ptr);
}
```

代码运行后，如下所示：

![](/image/1379525-20211126164634851-1859687141.png)

2.使用`TreeWidget`组件，自己定义一个菜单，并将该菜单绑定到Tree组件内，具体实现代码如下。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 在MainWindow中使用右击菜单需要添加此项
    ui->treeWidget->setContextMenuPolicy(Qt::CustomContextMenu);

    // 创建基础顶部菜单
    QMenuBar *bar = menuBar();
    this->setMenuBar(bar);
    QMenu * fileMenu = bar->addMenu("菜单1");

    // 实现只隐藏菜单1其他的不受影响
    fileMenu->menuAction()->setVisible(false);

    // 添加子菜单
    GetColumnAction = fileMenu->addAction("获取列号");
    GetRowDataAction = fileMenu->addAction("获取本行数据");
    GetLineAction = fileMenu->addAction("获取行号");

    // 分别设置图标
    GetColumnAction->setIcon(QIcon(":/image/1.ico"));
    GetRowDataAction->setIcon(QIcon(":/image/2.ico"));
    GetLineAction->setIcon(QIcon(":/image/3.ico"));

    // 为子菜单绑定热键
    GetColumnAction->setShortcut(Qt::CTRL | Qt::Key_A);
    GetRowDataAction->setShortcut(Qt::SHIFT | Qt::Key_S);
    GetLineAction->setShortcut(Qt::CTRL | Qt::SHIFT | Qt::Key_B);

    // 绑定槽函数: 获取选中列
    connect(GetColumnAction,&QAction::triggered,this,[=](){
        int col = ui->treeWidget->currentColumn();
        std::cout << col << std::endl;
    });

    // 绑定槽函数: 获取选中的第0行的数据内容
    connect(GetRowDataAction,&QAction::triggered,this,[=](){
        QString msg = ui->treeWidget->currentItem()->text(0);
        std::cout << msg.toStdString().data() << std::endl;
    });

    // 绑定槽函数: 获取当前选中的索引值
    connect(GetLineAction,&QAction::triggered,this,[=](){
        int row  = ui->treeWidget->currentIndex().row();
        std::cout << row << std::endl;
    });

    // 设置treeWidget属性
    ui->treeWidget->setColumnCount(4);         // 设置总列数
    ui->treeWidget->setColumnWidth(0,300);     // 设置最后一列宽度自适应
    ui->treeWidget->setIndentation(1);         // 设置表头缩进为1

    // 设置表头数据
    QStringList headers;
    headers.append("文件名");
    headers.append("更新时间");
    headers.append("文件类型");
    headers.append("文件大小");
    ui->treeWidget->setHeaderLabels(headers);

    // 模拟插入数据到表中
    for(int x=0;x<100;x++)
    {
        QTreeWidgetItem* item=new QTreeWidgetItem();
        item->setText(0,"<lyshark.com>");
        item->setIcon(0,QIcon(":/image/1.ico"));
        item->setText(1,"2020-12-11");
        item->setText(2,"*.pdf");
        item->setText(3,"102MB");
        item->setIcon(3,QIcon(":/image/2.ico"));
        ui->treeWidget->addTopLevelItem(item);
    }
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 当treeWidget中的右键被点击时则触发
// By: LyShark
// https://www.cnblogs.com/lyshark
void MainWindow::on_treeWidget_customContextMenuRequested(const QPoint &pos)
{
    std::cout << "x pos = "<< pos.x() << "y pos = " << pos.y() << std::endl;
    Q_UNUSED(pos);

    // 新建Menu菜单
    QMenu *ptr = new QMenu(this);

    // 添加Actions创建菜单项
    ptr->addAction(GetColumnAction);
    ptr->addAction(GetLineAction);

    // 添加一个分割线
    ptr->addSeparator();
    ptr->addAction(GetRowDataAction);

    // 在鼠标光标位置显示右键快捷菜单
    ptr->exec(QCursor::pos());
    // 手工创建的指针必须手工删除
    delete ptr;
}
```

最终我们实现的效果如下所示。

![](/image/1379525-20211126165614619-525263083.gif)
