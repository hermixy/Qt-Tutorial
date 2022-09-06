Qt窗体中默认会附加一个QstatusBar组件，状态栏组件位于主窗体的最下方，其作用是提供一个工具提示功能，当程序中有提示信息是可以动态的显示在这个区域内，状态栏组件内可以增加任何Qt中的通用组件，只需要通过`addWidget`函数动态追加即可引入到底部，底部状态栏在实际开发中应用非常普遍，以下代码是对该组件基本使用方法的总结。

首先我们通过`new`新增3个`QLabel`组件，并将该组件依次排列在底部状态栏内，实现代码如下所示:
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QLabel>

MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 初始化状态栏
    QLabel *labCellIndex = new QLabel("当前坐标: 0.0",this);
    labCellIndex->setMinimumWidth(250);

    QLabel *labCellType=new QLabel("单元格类型: null",this);
    labCellType->setMinimumWidth(200);

    QLabel *labStudID=new QLabel("学生ID: 0",this);
    labStudID->setMinimumWidth(200);

    // 将初始化的标签添加到底部状态栏上
    ui->statusBar->addWidget(labCellIndex);
    ui->statusBar->addWidget(labCellType);
    ui->statusBar->addWidget(labStudID);
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

运行代码效果如下:

![](/image/1379525-20211203133411677-79416055.png)

QLabel组件除了可以增加提示信息以外，通过设置`setOpenExternalLinks`可以将这个组件设置为以链接形式出现，有利于我们增加网页跳转等功能。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QLabel>

MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 隐藏状态栏下方三角形
    ui->statusBar->setSizeGripEnabled(false);

    // 新增标签栏
    QLabel *label_url = new QLabel(this);
    QLabel *label_about = new QLabel(this);

    // 配置连接
    label_url->setFrameStyle(QFrame::Box | QFrame::Sunken);
    label_url->setText(tr("<a href=\"https://lyshark.cnblogs.com\">访问主页</a>"));
    label_url->setOpenExternalLinks(true);

    label_about->setFrameStyle(QFrame::Box | QFrame::Sunken);
    label_about->setText(tr("<a href=\"https://lyshark.cnblogs.com\">关于我</a>"));
    label_about->setOpenExternalLinks(true);

    // 将信息增加到底部（永久添加）
    ui->statusBar->addPermanentWidget(label_url);
    ui->statusBar->addPermanentWidget(label_about);
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

运行代码效果如下:

![](/image/1379525-20211203125721479-549575758.png)

同理，只要是通用组件都可以被安置到底部菜单栏，如果我们需要增加进度条组件只需要这样写:
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QLabel>
#include <QProgressBar>

QProgressBar *pro;

MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    pro = new QProgressBar(this);

    // 自动计算
    ui->statusBar->addPermanentWidget(pro, 1);

    // 设置进度是否显示
    pro->setTextVisible(true);

    // 设置初始化进度位置
    pro->setValue(0);
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_pushButton_clicked()
{
    qint32 count = pro->value();
    count = count +10;
    pro->setValue(count);
}
```

运行代码效果如下:

![](/image/1379525-20211203132850078-2047417183.png)

接着我们增加一个`tablewidget`并初始化参数，tableWidget组件存在一个`on_tableWidget_currentCellChanged`属性，该属性的作用是，只要Table表格存在变化则会触发，当用户选择不同的表格，我们可以将当前表格行列自动设置到状态栏中，从而实现同步状态栏消息提示，起到时刻动态显示的作用。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QLabel>
#include <QTableWidget>
#include <QTableWidgetItem>

QLabel *labCellIndex;

MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

// ------------------------------------------------------------------------------------
// 初始化状态栏
    labCellIndex = new QLabel("当前坐标: 0.0",this);
    labCellIndex->setMinimumWidth(250);

    // 将初始化的标签添加到底部状态栏上
    ui->statusBar->addWidget(labCellIndex);

// ------------------------------------------------------------------------------------
// 填充数据，对表格进行初始化操作
    QStringList header;
    header << "姓名" << "性别" << "年龄";

    ui->tableWidget->setColumnCount(header.size());                        // 设置表格的列数
    ui->tableWidget->setHorizontalHeaderLabels(header);                    // 设置水平头
    ui->tableWidget->setRowCount(5);                                       // 设置总行数
    ui->tableWidget->setEditTriggers(QAbstractItemView::NoEditTriggers);   // 设置表结构默认不可编辑

    // 填充数据
    QStringList NameList;
    NameList << "lyshark A" << "lyshark B" << "lyshark C";

    QStringList SexList;
    SexList << "男" << "男" << "女";

    qint32 AgeList[3] = {22,23,43};

    // 针对获取元素使用 NameList[x] 和使用 NameList.at(x)效果相同
    for(int x=0;x< 3;x++)
    {
        int col =0;
        // 添加姓名
        ui->tableWidget->setItem(x,col++,new QTableWidgetItem(NameList[x]));
        // 添加性别
        ui->tableWidget->setItem(x,col++,new QTableWidgetItem(SexList.at(x)));
        // 添加年龄
        ui->tableWidget->setItem(x,col++,new QTableWidgetItem( QString::number(AgeList[x]) ) );
    }
}

// 当前选择单元格发生变化时触发响应事件,也就是将底部状态栏标签设置
// https://www.cnblogs.com/lyshark
void MainWindow::on_tableWidget_currentCellChanged(int currentRow, int currentColumn, int previousRow, int previousColumn)
{
    Q_UNUSED(previousRow);
    Q_UNUSED(previousColumn);

    // 显示行与列的变化数值
    //std::cout << "currentRow = " << currentRow << " currentColumn = " << currentColumn << std::endl;
    //std::cout << "pre Row = " << previousRow << " pre Column = " << previousColumn << std::endl;

    // 获取当前单元格的Item
    QTableWidgetItem *item = ui->tableWidget->item(currentRow,currentColumn);
    if(item == NULL)
    return;

    // 设置单元格坐标
    labCellIndex->setText(QString::asprintf("当前坐标: %d 行 | %d 列",currentRow,currentColumn));
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

运行代码效果如下:

![](/image/1379525-20211203131938209-282394017.gif)
