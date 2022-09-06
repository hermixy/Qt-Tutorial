在上一篇博文`《C/C++ Qt TreeWidget 单层树形组件应用》`中给大家演示了如何使用`TreeWidget`组件创建单层树形结构，并给这个树形组件增加了右键菜单功能，接下来将继续延申树形组件的使用，并实现对树形框多节点的各种操作，如下笔记是本人在开发中经常用到的一些基本操作技巧。

常用树形框节点间的操作方法如下:

 - TreeView 节点遍历
 - TreeWidget 初始化节点
 - TreeWidget 单击双击节点
 - TreeWidget 添加根节点
 - TreeWidget 添加子节点
 - TreeWidget 修改选中节点
 - TreeWidget 删除选中节点
 - TreeWidget 枚举全部节点
 - TreeWidget 枚举选中节点
 - TreeWidget 获取节点子节点

<br>

**简单的节点遍历:** 首先我们还是使用`TreeView`组件实现一个简单的多层嵌套树结构，代码运行后，首先循环设置3个外层节点，接着循环内层节点，并将内层中的`QStandardItem`追加到外层上面。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QTreeView>
#include <QStandardItemModel>

// By: LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    QStandardItemModel *tree = new QStandardItemModel(0,3,this);

    ui->treeView->setColumnWidth(0,50);      // 设置第1列长度
    ui->treeView->setColumnWidth(1,200);     // 设置第2列长度
    ui->treeView->setColumnWidth(2,200);     // 设置第3列长度

    tree->setHeaderData(0, Qt::Horizontal, tr("序号"));
    tree->setHeaderData(1, Qt::Horizontal, tr("姓名"));
    tree->setHeaderData(2, Qt::Horizontal, tr("年龄"));

    ui->treeView->setModel(tree);

    for (int i = 0; i < 4; ++i)
    {
        // 设置3个外层节点
        QList<QStandardItem *> items;
        for (int i = 0; i < 3; ++i)
        {
            QStandardItem *item = new QStandardItem(QString("%0").arg(i));

            if (0 == i)
                item->setCheckable(true);

            items.push_back(item);
        }
        tree->appendRow(items);

        // 设置内层
        for (int i = 0; i < 2; ++i)
        {
            QList<QStandardItem *> childItems;
            for (int i = 0; i < 3; ++i)
            {
             QStandardItem *item = new QStandardItem(QString("lyshark"));
             if (0 == i)
                 item->setCheckable(true);
             childItems.push_back(item);
            }
            items.at(0)->appendRow(childItems);
        }
    }
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

代码运行效果如下:

![](/image/1379525-20211127145331942-2034147543.png)

<br>

**初始化树形节点:** 首先在开始操作元素之前,我们可以在`MainWindow::MainWindow`中对树形节点进行简单的初始化,插入几个测试节点.
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QTreeWidgetItem>
#include <QString>

// By: LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->treeWidget->clear();

    // 设置QTreeWidget的列数
    ui->treeWidget->setColumnCount(1);
    // 设置QTreeWidget标题隐藏
    ui->treeWidget->setHeaderHidden(true);

    // 创建QTreeWidget的朋友节点,父节点是tree
    QTreeWidgetItem *Friend = new QTreeWidgetItem(ui->treeWidget,QStringList(QString("朋友")));
    Friend->setIcon(0,QIcon(":/image/4.ico"));  // 添加一个图标
    Friend->setFlags(Qt::ItemIsSelectable | Qt::ItemIsUserCheckable
                     | Qt::ItemIsEnabled | Qt::ItemIsAutoTristate);
    Friend->setCheckState(0,Qt::Checked);

    // 给Friend添加一个子节点frd
    QTreeWidgetItem *frd = new QTreeWidgetItem(Friend);
    frd->setText(0,"www.lyshark.com");
    frd->setIcon(0,QIcon(tr(":/image/1.ico")));
    frd->setCheckState(0,Qt::Checked);               // 默认选中状态

    QTreeWidgetItem *frs = new QTreeWidgetItem(Friend);
    frs->setText(0,"cdn.lyshark.com");
    frs->setIcon(0,QIcon(tr(":/image/1.ico")));
    frs->setCheckState(0,Qt::Unchecked);            // 默认未选中

    // ----------------------------------------------------------
    // 创建名叫同学节点,父节点同样是tree
    QTreeWidgetItem * ClassMate = new QTreeWidgetItem(ui->treeWidget,QStringList(QString("同学")));
    ClassMate->setIcon(0,QIcon(":/image/5.ico"));  // 添加一个图标
    ClassMate->setCheckState(0,Qt::Checked);       // 默认选中

    //Fly是ClassMate的子节点
    QTreeWidgetItem *Fly = new QTreeWidgetItem(QStringList(QString("nas.lyshark.com")));
    Fly->setIcon(0,QIcon(tr(":/image/2.ico")));
    //创建子节点的另一种方法
    ClassMate->addChild(Fly);
    Fly->setCheckState(0,Qt::Checked);       // 设置为选中

    QTreeWidgetItem *Fls = new QTreeWidgetItem(QStringList(QString("lyshark.cnblogs.com")));
    Fls->setIcon(0,QIcon(tr(":/image/2.ico")));
    ClassMate->addChild(Fls);
    Fls->setCheckState(0,Qt::Checked);       // 设置为选中

    // ----------------------------------------------------------
    // 陌生人单独一栏
    QTreeWidgetItem  *Strange = new QTreeWidgetItem(true);
    Strange->setText(0,"陌生人");
    Strange->setIcon(0,QIcon(":/image/6.ico"));  // 添加一个图标

    ui->treeWidget->addTopLevelItem(ClassMate);
    ui->treeWidget->addTopLevelItem(Strange);

    // 增加文本到编辑框
    ui->plainTextEdit->appendPlainText("hello lyshark");

    //展开QTreeWidget的所有节点
    //ui->treeWidget->expandAll();
    //ui->treeWidget->resize(271,401);
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

代码运行效果如下:

![](/image/1379525-20211127152025254-1640865007.png)

<br>

**单击双击节点反馈:** 当我们将鼠标停靠在指定节点内并点击时,我们需要触发`treeWidget_itemDoubleClicked`属性让其反馈该行标题等基本属性.
```C
// 当我们双击指定的成员时获取到该成员的名字
void MainWindow::on_treeWidget_itemDoubleClicked(QTreeWidgetItem *item, int column)
{
    QString str = item->text(column);
    std::cout << str.toStdString().data() << std::endl;
    ui->plainTextEdit->appendPlainText(str.toStdString().data());
}

// 当我们单击指定成员时获取数据
void MainWindow::on_treeWidget_itemClicked(QTreeWidgetItem *item, int column)
{
    QString str = item->text(column);
    std::cout << str.toStdString().data() << std::endl;
    ui->plainTextEdit->appendPlainText(str.toStdString().data());
}
```

代码运行效果如下:

![](/image/1379525-20211127152714916-999017813.gif)

<br>

**添加 父节点/子节点:** 通过代码的方式当点击`on_pushButton_clicked`时分别实现增加一个父节点和一个子节点的功能。
```C
// 单击按钮添加新的父节点
void MainWindow::on_pushButton_clicked()
{
    QString NodeText = "新的父节点";
    QTreeWidgetItem  *item = new QTreeWidgetItem(true);
    item->setText(0,NodeText);
    item->setIcon(0,QIcon(":/image/7.ico"));
    ui->treeWidget->addTopLevelItem(item);
}

// 单击按钮添加子节点
void MainWindow::on_pushButton_4_clicked()
{
    QTreeWidgetItem * item= ui->treeWidget->currentItem();
        if(item!=NULL)
            AddTreeNode(item,"新子节点","新子节点");
        else
            AddTreeRoot("新子节点","新子节点");
}
```

代码运行效果如下:

![](/image/1379525-20211127154351973-1197216214.gif)

<br>

**删除选中节点:** 首先选中要删除的指定节点，然后可以对该节点进行删除操作，删除子节点直接移除即可，删除父节点需要连同内部子节点一并删掉。
```C
// 删除选中的节点
void MainWindow::on_pushButton_3_clicked()
{
    QTreeWidgetItem *currentItem = ui->treeWidget->currentItem();
    if(currentItem == NULL)
        return;

    // 如果没有父节点则直接删除
    if(currentItem->parent() == NULL)
    {
        delete ui->treeWidget->takeTopLevelItem(ui->treeWidget->currentIndex().row());
        std::cout << ui->treeWidget->currentIndex().row() << std::endl;
    }
    else
    {
        // 如果有父节点就要用父节点的takeChild删除节点
        delete currentItem->parent()->takeChild(ui->treeWidget->currentIndex().row());
    }
}
```

代码运行效果如下:

![](/image/1379525-20211127154445606-2050818273.gif)

<br>

**修改指定节点名称:** 单击后将指定节点修改为Modify并将图标设置为新的
```C
// 修改节点
// By: LyShark
// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_2_clicked()
{
    // 得到当前节点
    QTreeWidgetItem *currentItem = ui->treeWidget->currentItem();
    if(currentItem == NULL)
        return;
    // 修改选中项
    for(int x=0;x<currentItem->columnCount();x++)
    {
        currentItem->setText(x,tr("Modify") + QString::number(x));
        currentItem->setIcon(x,QIcon(":/image/1.ico"));
    }
}
```

代码运行效果如下:

![](/image/1379525-20211127154522103-795727827.gif)

<br>

**枚举所有节点元素:** 枚举当前Tree中的所有节点元素，并将结果输出到右侧编辑框内。
```C
// 枚举所有节点
// By: LyShark
// https://www.cnblogs.com/lyshark
// 枚举所有节点
void MainWindow::on_pushButton_5_clicked()
{
    // 获取到全部的根节点数量
    int size = ui->treeWidget->topLevelItemCount();
    QTreeWidgetItem *child;
    for(int x=0;x<size;x++)
    {
        // 输出所有父节点
        child = ui->treeWidget->topLevelItem(x);
        std::cout << "all root = "<< child->text(0).toStdString().data() << std::endl;
        ui->plainTextEdit->appendPlainText(child->text(0).toStdString().data());

        // 得到所有子节点计数
        int childCount = child->childCount();
        // std::cout << "all child count = " << childCount << std::endl;

        // 输出根节点下面的子节点
        for(int y=0;y<childCount;++y)
        {
            QTreeWidgetItem *grandson = child->child(y);
            std::cout << "--> sub child = "<< grandson->text(0).toStdString().data() << std::endl;
            ui->plainTextEdit->appendPlainText(grandson->text(0).toStdString().data());
        }
    }
}
```

代码运行效果如下:

![](/image/1379525-20211127154944381-1774109395.gif)

<br>

**枚举选中节点元素:** 枚举当前Tree中选中节点的元素，并将结果输出到右侧编辑框内。
```C
// 枚举所有的 【选中】节点
// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_7_clicked()
{
    // 获取到全部的根节点数量
    int size = ui->treeWidget->topLevelItemCount();
    QTreeWidgetItem *child;
    for(int x=0;x<size;x++)
    {
        // 输出所有父节点
        child = ui->treeWidget->topLevelItem(x);

        // 得到所有子节点计数
        int childCount = child->childCount();

        // 输出根节点下面的子节点
        for(int y=0;y<childCount;++y)
        {
            QTreeWidgetItem *grandson = child->child(y);
            // 判断是否选中,如果选中输出父节点与子节点
            if(Qt::Checked == grandson->checkState(0))
            {
                std::cout << "root -> " << child->text(0).toStdString().data()
                          << "--> sub child = "<< grandson->text(0).toStdString().data() << std::endl;

                ui->plainTextEdit->appendPlainText(grandson->text(0).toStdString().data());
            }
        }
    }
}
```

代码运行效果如下:

![](/image/1379525-20211127155349421-2083367768.gif)

<br>

**获取选中子节点的父节点:** 获取子节点的父节点ID,然后根据ID得到子节点名字。
```C
void MainWindow::on_pushButton_6_clicked()
{
    // 取所有的父节点
    QTreeWidgetItem *currentItem = ui->treeWidget->currentItem()->parent();
    int root_count = ui->treeWidget->indexOfTopLevelItem(currentItem);
    std::cout << "root Count = " <<  root_count << std::endl;
    if(root_count != -1)
    {
        // 指定序号对应的父节点名字
        QTreeWidgetItem *child;

        child = ui->treeWidget->topLevelItem(root_count);
        std::cout << "root name= "<< child->text(0).toStdString().data() << std::endl;
        
        ui->plainTextEdit->appendPlainText(child->text(0).toStdString().data());
    }
}
```

代码运行效果如下:

![](/image/1379525-20211127155556020-1854982478.gif)

补充一下节点插入函数的定义,`AddTreeRoot/AddTreeNode`两个函数定义如下所示.
```C
// mainwindow.h 中增加头部声明
    QTreeWidgetItem * AddTreeRoot(QString name,QString desc);
    QTreeWidgetItem * AddTreeNode(QTreeWidgetItem *parent,QString name,QString desc);

// mainwindow.cpp 中增加实现部分
QTreeWidgetItem * MainWindow::AddTreeRoot(QString name,QString desc)
{
    QTreeWidgetItem * item=new QTreeWidgetItem(QStringList()<<name<<desc);
    ui->treeWidget->addTopLevelItem(item);
    return item;
}
QTreeWidgetItem * MainWindow::AddTreeNode(QTreeWidgetItem *parent,QString name,QString desc)
{
    QTreeWidgetItem * item=new QTreeWidgetItem(QStringList()<<name<<desc);
    parent->addChild(item);
    return item;
}
```
