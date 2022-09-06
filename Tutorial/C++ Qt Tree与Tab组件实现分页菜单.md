虽然`TreeWidget`组件可以实现多节点的增删改查，但多节点操作显然很麻烦，在一般的应用场景中基本上只使用一层结构即可解决大部分开发问题，`TreeWidget`组件通常可配合`TabWidget`组件，实现一个类似于树形菜单栏的功能，当用户点击菜单栏中的选项时则会跳转到不同的页面上。

首先在Qt的Ui编辑界面左侧加入`TreeWidget`组件，右侧加入`TabWidget`组件，将页面中的`TabWidget`组件增加指定页，效果如下。

![image](https://user-images.githubusercontent.com/52789403/188529923-4c33deb9-758e-4d2a-b19b-c24ed7e7bbed.png)

在`MainWindow::MainWindow`主函数中我们对其中的两个组件进行初始化操作。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QStyleFactory>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->treeWidget->clear();

    ui->treeWidget->setColumnCount(1);
    ui->treeWidget->setHeaderHidden(true);
    ui->tabWidget->tabBar()->hide();
    // 增加线条
    ui->treeWidget->setStyle(QStyleFactory::create("windows"));

// ----------------------------------------------------------
// By: LyShark
    // 创建 [系统设置] 父节点
    QTreeWidgetItem *system_setup = new QTreeWidgetItem(ui->treeWidget,QStringList(QString("系统位置")));
    system_setup->setFlags(Qt::ItemIsSelectable | Qt::ItemIsUserCheckable | Qt::ItemIsEnabled | Qt::ItemIsAutoTristate);

    // 给父节点添加子节点
    QTreeWidgetItem *system_setup_child_node_1 = new QTreeWidgetItem(system_setup);
    system_setup_child_node_1->setText(0,"修改密码");
    QTreeWidgetItem *system_setup_child_node_2 = new QTreeWidgetItem(system_setup);
    system_setup_child_node_2->setText(0,"设置菜单");

// ----------------------------------------------------------
// https://www.cnblogs.com/lyshark
    // 创建 [页面布局] 父节点
    QTreeWidgetItem *page_layout = new QTreeWidgetItem(ui->treeWidget,QStringList(QString("页面布局")));
    page_layout->setFlags(Qt::ItemIsSelectable | Qt::ItemIsUserCheckable | Qt::ItemIsEnabled | Qt::ItemIsAutoTristate);

    QTreeWidgetItem *page_layout_clild_1 = new QTreeWidgetItem(page_layout);
    page_layout_clild_1->setText(0,"页面配置");
    QTreeWidgetItem *page_layout_clild_2 = new QTreeWidgetItem(page_layout);
    page_layout_clild_2->setText(0,"页面参数");

    ui->treeWidget->expandAll();
}

MainWindow::~MainWindow()
{
    delete ui;
}
```
接着增加`TreeWidget`组件的右键点击事件，当右键点击节点时，先判断节点是哪一个，并自动将`TabWidget`组件切换到指定的页上。
```C
// 当treeWidget空间双击后根据不同的菜单项选择不同的TabView页
void MainWindow::on_treeWidget_itemDoubleClicked(QTreeWidgetItem *item, int column)
{
    QString str = item->text(column);

    if(str == "修改密码")
    {
        ui->tabWidget->setCurrentIndex(0);
    }
    if(str == "设置菜单")
    {
        ui->tabWidget->setCurrentIndex(1);
    }
    if(str == "页面配置")
    {
        ui->tabWidget->setCurrentIndex(2);
    }
    if(str == "页面参数")
    {
        ui->tabWidget->setCurrentIndex(3);
    }
}
```

代码实现起来很简单，具体实现效果如下所示:

![image](https://user-images.githubusercontent.com/52789403/188529948-237f9bc7-e582-4153-9eef-4b26ce6a1977.png)
