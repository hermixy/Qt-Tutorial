在上一篇博文`《C/C++ Qt ListWidget 列表框组件应用》`中介绍了ListWidget组件的基本使用技巧，本次将给ListWidget组件增加一个右键菜单，当用户在ListWidget组件中的任意一个子项下右键，我们让其弹出这个菜单，并根据选择提供不同的功能。

为了增加菜单，我们首先需要在程序全局增加`QAction`其中每一个QAction则代表一个菜单选项指针。
```C
// 全局下设置增加菜单
QAction *NewAction;
QAction *InsertAction;
QAction *DeleteAction;
```
其次则是通过代码的方式在程序中动态创建一个基础的右键菜单，并对该菜单设置子菜单以及所对应的图标组，最后就是将信号连接到指定的全局菜单指针上即可，这个代码实现如下。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QMenuBar>
#include <QMenu>
#include <QToolBar>
#include <iostream>

// 全局下设置增加菜单
QAction *NewAction;
QAction *InsertAction;
QAction *DeleteAction;

// By: LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 使用 customContextMenuRequested 信号则需要设置
    ui->listWidget->setContextMenuPolicy(Qt::CustomContextMenu);

    // 隐藏菜单栏上的右击菜单
    this->setContextMenuPolicy(Qt::NoContextMenu);

    // 创建基础顶部菜单
    QMenuBar *bar = menuBar();
    this->setMenuBar(bar);
    QMenu * fileMenu = bar->addMenu("菜单1");
    bar->setVisible(false);   // 隐藏顶部菜单栏

    // 添加子菜单
     NewAction = fileMenu->addAction("增加IP地址");
     InsertAction = fileMenu->addAction("插入IP地址");
     DeleteAction = fileMenu->addAction("删除IP地址");

    // 分别设置图标
    NewAction->setIcon(QIcon(":/image/1.ico"));
    InsertAction->setIcon(QIcon(":/image/2.ico"));
    DeleteAction->setIcon(QIcon(":/image/3.ico"));

    // 绑定槽函数
    connect(NewAction,&QAction::triggered,this,[=](){
        std::cout << "new action" << std::endl;
        ui->plainTextEdit->appendPlainText(QString("新建触发"));
    });

    connect(InsertAction,&QAction::triggered,this,[=](){
        std::cout << "insert action" << std::endl;
        ui->plainTextEdit->appendPlainText(QString("插入触发"));
    });

    // 以删除为例,演示如何删除选中行
    connect(DeleteAction,&QAction::triggered,this,[=](){
        int row = ui->listWidget->currentRow();
        QListWidgetItem *aItem = ui->listWidget->takeItem(row);
        delete aItem;
        std::cout << "delete action" << std::endl;
        ui->plainTextEdit->appendPlainText(QString("删除触发"));
    });
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 当listWidget被右键点击时则触发
void MainWindow::on_listWidget_customContextMenuRequested(const QPoint &pos)
{
    std::cout << "x pos = "<< pos.x() << "y pos = " << pos.y() << std::endl;
    Q_UNUSED(pos);

    // 新建Menu菜单
    QMenu *ptr = new QMenu(this);

    // 添加Actions创建菜单项
    ptr->addAction(NewAction);
    ptr->addAction(InsertAction);
    // 添加一个分割线
    ptr->addSeparator();
    ptr->addAction(DeleteAction);

    // 在鼠标光标位置显示右键快捷菜单
    ptr->exec(QCursor::pos());

    // 手工创建的指针必须手工删除
    delete ptr;
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529762-2abe6ecc-2cbc-4f56-9918-7970816458b9.png)

ListWidget同样支持一图标方式显示列表框内的元素，只需要设置`setViewMode(QListView::IconMode)`属性即可实现图标显示，我们按照如上代码简单改进即可，代码如下:
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QMenuBar>
#include <QMenu>
#include <QToolBar>
#include <iostream>

// 全局下设置增加删除菜单
QAction *NewAction;
QAction *InsertAction;
QAction *DeleteAction;

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 使用 customContextMenuRequested 信号则需要设置
    ui->listWidget_2->setContextMenuPolicy(Qt::CustomContextMenu);

    // 隐藏菜单栏上的右击菜单
    this->setContextMenuPolicy(Qt::NoContextMenu);

    // 创建基础顶部菜单
    QMenuBar *bar = menuBar();
    this->setMenuBar(bar);
    QMenu * fileMenu = bar->addMenu("菜单1");
    bar->setVisible(false);   // 隐藏顶部菜单栏

    // 添加子菜单
     NewAction = fileMenu->addAction("增加IP地址");
     InsertAction = fileMenu->addAction("插入IP地址");
     DeleteAction = fileMenu->addAction("删除IP地址");

    // 分别设置图标
    NewAction->setIcon(QIcon(":/image/1.ico"));
    InsertAction->setIcon(QIcon(":/image/2.ico"));
    DeleteAction->setIcon(QIcon(":/image/3.ico"));

    // 绑定槽函数
    connect(NewAction,&QAction::triggered,this,[=](){
        std::cout << "new action" << std::endl;
    });

    connect(InsertAction,&QAction::triggered,this,[=](){
        std::cout << "insert action" << std::endl;
    });

    // 以删除为例,演示如何删除选中行
    connect(DeleteAction,&QAction::triggered,this,[=](){
        int row = ui->listWidget_2->currentRow();
        QListWidgetItem *aItem = ui->listWidget_2->takeItem(row);
        delete aItem;
        std::cout << "delete action" << std::endl;
    });
	
	// 第二个ListWidget_使用图标方式展示
    ui->listWidget_2->setViewMode(QListView::IconMode);

    // 每一行是一个QListWidgetItem
    QListWidgetItem *aItem;

    // 设置ICON的图标
    QIcon aIcon;
    aIcon.addFile(":/image/1.ico");

    ui->listWidget_2->clear();
    for(int x=0;x<10;x++)
    {
        QString str = QString::asprintf("admin_%d",x);
        aItem = new QListWidgetItem();   // 新建一个项

        aItem->setText(str);                   // 设置文字标签
        aItem->setIcon(aIcon);                 // 设置图标
        //aItem->setCheckState(Qt::Checked);     // 设为选中状态
        aItem->setFlags(Qt::ItemIsSelectable |  // 设置为不可编辑状态
                         Qt::ItemIsUserCheckable
                        |Qt::ItemIsEnabled);

        ui->listWidget_2->addItem(aItem); //增加项
    }
}

MainWindow::~MainWindow()
{
    delete ui;
}

// By: LyShark
// https://www.cnblogs.com/lyshark
void MainWindow::on_listWidget_2_customContextMenuRequested(const QPoint &pos)
{
    std::cout << "x pos = "<< pos.x() << "y pos = " << pos.y() << std::endl;
    Q_UNUSED(pos);

    // 新建Menu菜单
    QMenu *ptr = new QMenu(this);

    // 添加Actions创建菜单项
    ptr->addAction(NewAction);
    ptr->addAction(InsertAction);
    // 添加一个分割线
    ptr->addSeparator();
    ptr->addAction(DeleteAction);

    // 在鼠标光标位置显示右键快捷菜单
    ptr->exec(QCursor::pos());

    // 手工创建的指针必须手工删除
    delete ptr;
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529791-790f88d1-9706-4352-9eba-06c073ec0c30.png)
