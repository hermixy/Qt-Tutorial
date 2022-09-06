ToolBar工具栏在所有窗体应用程序中都广泛被使用，使用ToolBar可以很好的规范菜单功能分类，用户可根据菜单栏来选择不同的功能，Qt中默认自带ToolBar组件，当我们以默认方式创建窗体时，ToolBar就被加入到了窗体中，一般是以QToolBar的方式存在于对象菜单栏，如下所示。

![image](https://user-images.githubusercontent.com/52789403/188528456-8b82fc68-88ed-4c85-ac8a-bf9851135b23.png)

QToolBar组件在开发中我遇到了以下这些功能，基本上可以应对大部分开发需求了，这里就做一个总结。

顶部工具栏`ToolBar`组件的定义有多种方式,我们可以直接通过代码生成,也可以使用图形界面UI拖拽实现,但使用代码时间则更加灵活一些,ToolBar组件可以表现出多种形态.

首先来看一个简单的生成案例,如下代码中我们通过属性`setAllowedAreas()`可以实现将ToolBar组件放置到上下左右四个不同的方位上面.
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QMenuBar>
#include <QToolBar>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

// ----------------------------------------------------------
// 创建菜单栏
    QMenuBar *bar = menuBar();
    this->setMenuBar(bar);                      // 将菜单栏放入主窗口
    QMenu * fileMenu = bar->addMenu("文件");     // 创建父节点

    // 添加子菜单
    QAction *newAction = fileMenu->addAction("新建文件");     // 设置名字
    //newAction->setIcon(QIcon("://image/1.ico"));           // 设置可用图标

    fileMenu->addSeparator();                                // 添加分割线
    QAction *openAction = fileMenu->addAction("打开文件");     // 设置名字
    //openAction->setIcon(QIcon("://image/2.ico"));          // 设置可用图标

// ----------------------------------------------------------
//创建工具栏
    QToolBar *toolBar = new QToolBar(this);  // 创建工具栏
    addToolBar(Qt::LeftToolBarArea,toolBar); // 设置默认停靠范围 [默认停靠左侧]

    toolBar->setAllowedAreas(Qt::TopToolBarArea |Qt::BottomToolBarArea);   // 允许上下拖动
    toolBar->setAllowedAreas(Qt::LeftToolBarArea |Qt::RightToolBarArea);   // 允许左右拖动

    toolBar->setFloatable(false);       // 设置是否浮动
    toolBar->setMovable(false);         // 设置工具栏不允许移动

    // 工具栏添加菜单项
    toolBar->addAction(newAction);
    toolBar->addSeparator();
    toolBar->addAction(openAction);

// By : LyShark
// https://www.cnblogs.com/lyshark
// ----------------------------------------------------------
// 绑定槽函数
    connect(newAction,&QAction::triggered,this,[=](){
        std::cout << "new action" << std::endl;
    });

    connect(openAction,&QAction::triggered,this,[=](){
        std::cout << "open action" << std::endl;
    });
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

![image](https://user-images.githubusercontent.com/52789403/188528436-c988aec8-548e-4b86-a89e-c46c223c2668.png)

接着通过代码的方式实现一个顶部菜单栏，该菜单栏中可以通过`SetIcon(QIcon("://image/1.ico"));`指定图标，也可以使用`setShortcut(Qt::CTRL | Qt::Key_C);`为其指定特殊的快捷键。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QMenuBar>
#include <QToolBar>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

// ----------------------------------------------------------
// 创建菜单栏
    QMenuBar *bar = menuBar();
    this->setMenuBar(bar);  //将菜单栏放入主窗口
    QMenu * fileMenu = bar->addMenu("文件");

// By : LyShark
// https://www.cnblogs.com/lyshark
    // 添加子菜单
    QAction *newAction = fileMenu->addAction("新建文件");      // 添加名字
    newAction->setIcon(QIcon(":/image/1.ico"));              // 设置ICO图标
    newAction->setShortcut(Qt::CTRL | Qt::Key_A);            // 设置快捷键ctrl+a

    fileMenu->addSeparator();                                // 添加分割线

    QAction *openAction = fileMenu->addAction("打开文件");
    openAction->setIcon(QIcon(":/image/2.ico"));
    openAction->setShortcut(Qt::CTRL | Qt::Key_C);          // 设置快捷键ctrl+c

// ----------------------------------------------------------
// 创建工具栏(可屏蔽掉,屏蔽掉后底部将失去控件栏位)

    QToolBar *toolBar = new QToolBar(this);       // 创建工具栏
    addToolBar(Qt::BottomToolBarArea,toolBar);    // 设置默认停靠范围(停靠在底部)
    toolBar->setFloatable(false);                 // 设置是否浮动为假
    toolBar->setMovable(false);                   // 设置工具栏不允许移动

    // 工具栏添加菜单项
    toolBar->addAction(newAction);               // 工具栏添加[新建文件]
    toolBar->addSeparator();                     // 添加分割线
    toolBar->addAction(openAction);              // 添加[打开文件]

// ----------------------------------------------------------
// 绑定信号和槽
   connect(newAction,&QAction::triggered,this,[=](){
       std::cout << "new file slot" << std::endl;
   });

   connect(openAction,&QAction::triggered,this,[=](){
       std::cout << "open file slot" << std::endl;
   });
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

![image](https://user-images.githubusercontent.com/52789403/188528402-307ad5e0-977d-44ee-ac5c-12d94540ee1b.png)


实现顶部菜单栏二级菜单，二级顶部菜单与一级菜单完全一致，只是在一级菜单的基础上进行了延申，如下代码则是定义了一个二级菜单。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QMenuBar>
#include <QToolBar>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

// ----------------------------------------------------------
// 多层菜单导航栏
       QMenuBar *MainMenu = new QMenuBar(this);
       this->setMenuBar(MainMenu);

       // 1.定义父级菜单
       QMenu *EditMenu = MainMenu->addMenu("编辑");

       // 1.1 定义 EditMemu 下面的子菜单
       QAction *text = new QAction(EditMenu);
       text->setText("编辑文件");                     // 设置文本内容
       text->setShortcut(Qt::CTRL | Qt::Key_A);      // 设置快捷键ctrl+a
       text->setIcon(QIcon(":/image/1.ico"));        // 增加图标
       EditMenu->addAction(text);

       EditMenu->addSeparator();                      // 在配置模式与编辑文件之间增加虚线

       QAction *option = new QAction(EditMenu);
       option->setText("配置模式");
       option->setIcon(QIcon(":/image/2.ico"));
       EditMenu->addAction(option);

       // 1.1.2 定义Option配置模式下的子菜单
       QMenu *childMenu = new QMenu();
       QAction *set_file = new QAction(childMenu);
       set_file->setText("设置文件内容");
       set_file->setIcon(QIcon(":/image/3.ico"));

       childMenu->addAction(set_file);

       QAction *read_file = new QAction(childMenu);
       read_file->setText("读取文件内容");
       read_file->setIcon(QIcon(":/image/2.ico"));
       childMenu->addAction(read_file);
// ----------------------------------------------------------
// 注册菜单到窗体中
// By : LyShark
// https://www.cnblogs.com/lyshark

       // 首先将childMenu注册到option中
       option->setMenu(childMenu);
       // 然后再将childMenu加入到EditMenu中
       EditMenu->addMenu(childMenu);

// ----------------------------------------------------------
// 绑定信号和槽
       connect(text,&QAction::triggered,this,[=](){
           std::cout << "edit file slot" << std::endl;
       });

       connect(set_file,&QAction::triggered,this,[=](){
           std::cout << "set file slot" << std::endl;
       });

       connect(read_file,&QAction::triggered,this,[=](){
          std::cout << "read file slot" << std::endl;
       });
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

![image](https://user-images.githubusercontent.com/52789403/188528387-72395a9d-53e7-4179-a026-a1e7ce253a91.png)

Qt中的菜单还可以实现任意位置的弹出，例如我们可以将右击`customContextMenuRequested()`事件，绑定到主窗口中，实现在窗体任意位置右击都可以弹出菜单栏，代码如下。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QMenuBar>
#include <iostream>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    this->setContextMenuPolicy(Qt::CustomContextMenu);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 在主界面右击->转到customContextMenuRequested槽
// By : LyShark
// https://www.cnblogs.com/lyshark
void MainWindow::on_MainWindow_customContextMenuRequested(const QPoint &pos)
{
    // 创建菜单对象
    QMenu *pMenu = new QMenu(this);

    QAction *pNewTask = new QAction(tr("新建"), this);
    QAction *pEditTask = new QAction(tr("编辑"), this);
    QAction *pDeleteTask = new QAction(tr("删除"), this);

    // 设置属性值编号: 1=>新建 2=>设置 3=>删除
    pNewTask->setData(1);
    pEditTask->setData(2);
    pDeleteTask ->setData(3);

    // 把QAction对象添加到菜单上
    pMenu->addAction(pNewTask);
    pMenu->addAction(pEditTask);
    pMenu->addAction(pDeleteTask);

    // 增加图标
    pNewTask->setIcon(QIcon(":/image/1.ico"));
    pEditTask->setIcon(QIcon(":/image/2.ico"));
    pDeleteTask->setIcon(QIcon(":/image/3.ico"));

    // 连接鼠标右键点击信号
    connect(pNewTask, SIGNAL(triggered()), this, SLOT(onTaskBoxContextMenuEvent()));
    connect(pEditTask, SIGNAL(triggered()), this, SLOT(onTaskBoxContextMenuEvent()));
    connect(pDeleteTask, SIGNAL(triggered()), SLOT(onTaskBoxContextMenuEvent()));

    // 在鼠标右键点击的地方显示菜单
    pMenu->exec(QCursor::pos());

    //释放内存
    QList<QAction*> list = pMenu->actions();
    foreach (QAction* pAction, list) delete pAction;
    delete pMenu;
}

// 处理发送过来的信号
void MainWindow::onTaskBoxContextMenuEvent()
{
    // this->sender()就是信号发送者 QAction
    QAction *pEven = qobject_cast<QAction *>(this->sender());

    // 获取编号: 1=>新建 2=>设置 3=>删除
    int iType = pEven->data().toInt();

    switch (iType)
    {
    case 1:
        std::cout << "新建任务" << std::endl;
        break;
    case 2:
        std::cout << "设置任务" << std::endl;
        break;
    case 3:
        std::cout << "删除任务" << std::endl;
        break;
    default:
        break;
    }
}
```

![image](https://user-images.githubusercontent.com/52789403/188528358-e7cf9207-df13-4540-bab2-a22123e0fda9.png)

还可以将顶部的菜单通过`bar->setVisible(false);`属性将其隐藏起来，对外只展示出一个ToolBar控件栏位，ToolBar控件栏中只保留ICO图标与底部文字描述，这样能显得更加清爽一些。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QMenuBar>
#include <QToolBar>
#include <iostream>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

// ----------------------------------------------------------
    // 隐藏菜单栏上的右击菜单
    this->setContextMenuPolicy(Qt::NoContextMenu);

    // 创建基础顶部菜单并让其隐藏
    QMenuBar *bar = menuBar();
    this->setMenuBar(bar);
    QMenu * fileMenu = bar->addMenu("Ptr");
    bar->setVisible(false);                 // 隐藏菜单

    // 添加子菜单
    QAction *NewAction = fileMenu->addAction("新建文件");
    QAction *OpenAction = fileMenu->addAction("打开文件");
    QAction *ReadAction = fileMenu->addAction("读入文件");

    // 分别设置图标
    NewAction->setIcon(QIcon(":/image/1.ico"));
    OpenAction->setIcon(QIcon(":/image/2.ico"));
    ReadAction->setIcon(QIcon(":/image/3.ico"));

    // 创建工具栏
    QToolBar *toolBar = new QToolBar(this);
    addToolBar(Qt::TopToolBarArea,toolBar);

    // 将菜单项依次添加到工具栏
    toolBar->addAction(NewAction);
    toolBar->addAction(OpenAction);
    toolBar->addAction(ReadAction);

    // 设置禁止移动属性,工具栏默认贴在上方
    toolBar->setFloatable(false);
    toolBar->setMovable(false);
    toolBar->setToolButtonStyle(Qt::ToolButtonTextUnderIcon);

// ----------------------------------------------------------
// 绑定槽函数
// By : LyShark
// https://www.cnblogs.com/lyshark
    connect(NewAction,&QAction::triggered,this,[=](){
        std::cout << "new action" << std::endl;
    });

    connect(OpenAction,&QAction::triggered,this,[=](){
        std::cout << "open action" << std::endl;
    });

    connect(ReadAction,&QAction::triggered,this,[=](){
        std::cout << "read action" << std::endl;
    });
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

![image](https://user-images.githubusercontent.com/52789403/188528342-d4893665-b213-46a2-ae8e-fbd6043b3bf0.png)

