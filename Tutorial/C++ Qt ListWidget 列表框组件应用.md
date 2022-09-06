ListWidget列表框组件，该组件与TreeWidget有些相似，区别在于TreeWidget可以实现嵌套以及多字段结构，而ListWidget组件则只能实现单字段结构，ListWidget组件常用于显示单条记录，例如只显示IP地址，用户名等数据，如下笔记是本人在开发中经常用到的一些基本操作技巧，包括列表框组件的基本操作方法。

常用节点间的操作方法如下:

 - ListView 组件与应用基础
 - ListWidget 初始化
 - ListWidget 变化行(触发事件)
 - ListWidget 编辑状态设置
 - ListWidget 全选/全不选
 - ListWidget 反选(对错交织)
 - ListWidget 指定位置插入 / 增加一项
 - ListWidget 删除选中项

<br>

**ListView 组件与应用基础:** 该组件与`ListWidget`功能一致，只是`ListView`无法实现编辑只能预览。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QListView>
#include <QStandardItem>
#include <QStringListModel>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 初始化View组件 向ListView组件中填充数据
// By:LyShark
// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_clicked()
{
    QStringList data;
    QStringListModel *model;

    // 追加数据到ListView中
    data << QString("192.168.1.1");
    data << QString("192.168.1.2");
    data << QString("192.168.1.3");
    data << QString("192.168.1.4");

    model = new QStringListModel(data);
    ui->listView->setModel(model);

    // 移除第1个地址
    data.removeAt(0);

    // 再次刷新ListView
    model = new QStringListModel(data);
    ui->listView->setModel(model);
}

// 实现间隔初始化,每一行一种颜色
void MainWindow::on_pushButton_2_clicked()
{
    QStringList data;
    QStandardItemModel *model = new QStandardItemModel();

    // 清空记录
    model->removeRows(0,model->rowCount());

    // 追加数据到ListView中
    data << QString("192.168.1.1");

    // 循环追加
    for(int x=2; x<5; x++)
    {
        data << QString("192.168.1.%0").arg(x);
    }

    // 输出到ListView记录
    int nCount = data.size();
    for(int x=0; x<nCount; x++)
    {
        QString string = static_cast<QString>(data.at((x)));  // 强转为QString类型
        QStandardItem *item = new QStandardItem(string);

        if(x%2 == 0)
        {
            // 设置色彩
            QLinearGradient linear_grad(QPointF(0,0),QPointF(200,200));
            linear_grad.setColorAt(0,Qt::darkGreen);

            QBrush brush(linear_grad);
            item->setBackground(brush);
        }
        // 追加到mode模型
        model->appendRow(item);
    }

    // 设置模型
    ui->listView->setModel(model);
    //ui->listView->setFixedSize(200,300);
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529658-e2d2ec2c-5631-4288-8849-8f9c78136c7b.png)

上方代码中我们多数都是在使用`View`视图组件，接下来将具体分析`Widget`组件的使用细节，View组件与Widget组件看似一致，但却存在本质区别，其大致区别如下:

 - Widget 组件可以直接通过如`AddItem`等一系列函数操作特定数据集，该组件还具有直接编辑的能力。
 - View 组件是基于Model模型映射工作的，每次操作数据时都需要借助`QAbstractListModel`数据模型来操作。

简单来说`View`组件适合于浏览展示数据较多的场景，因为其绑定了链表结构从而在数据的展示上更为灵活，而`Widget`组件更适合于更新或修改数据较多的使用场景。

<br>

**ListWidget 节点初始化:** 节点的初始化就是向widget组件内插入一个`QListWidgetItem`类。
```C
// 初始化列表 listWidget
// By: LyShark
void MainWindow::on_pushButton_clicked()
{
    // 每一行是一个QListWidgetItem
    QListWidgetItem *aItem;

    // 设置ICON的图标
    QIcon aIcon;
    aIcon.addFile(":/image/1.ico");

    ui->listWidget->clear();
    for(int x=0;x<10;x++)
    {
        QString str = QString::asprintf("192.168.1.%d",x);
        aItem = new QListWidgetItem();   // 新建一个项

        aItem->setText(str);                   // 设置文字标签
        aItem->setIcon(aIcon);                 // 设置图标
        aItem->setCheckState(Qt::Checked);     // 设为选中状态
        aItem->setFlags(Qt::ItemIsSelectable |  // 设置为不可编辑状态
                         Qt::ItemIsUserCheckable
                        |Qt::ItemIsEnabled);

        ui->listWidget->addItem(aItem); //增加项
    }
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529676-f3e325eb-3d0e-406c-b91d-8d091f4dba94.png)

<br>

**ListWidget 行内文本变化:** 当我们点击行内任意一个列表选项时，我们让其触发`currentItemChanged`并将变化行更新到窗体上。
```C
// listWidget 当前选中项发生变化
// By: LyShark
void MainWindow::on_listWidget_currentItemChanged(QListWidgetItem *current, QListWidgetItem *previous)
{
    QString str;
    if (current != NULL) //需要检测变量指针是否为空
    {
      if (previous==NULL)  //需要检测变量指针是否为空
      {
          str="当前："+current->text();
          this->setWindowTitle(QString(current->text()));
      }
      else
      {
        str="前一项：" + previous->text() + "; 当前项：" + current->text();
        std::cout << str.toStdString().data() << std::endl;
        this->setWindowTitle(QString(current->text()));
      }
    }
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529385-a93d25b1-3dd8-402b-8805-28d42eac2197.png)


<br>

**ListWidget 编辑状态设置:** 默认情况下`ListWidget`组件内所有文件是不可编辑的，我们也可以将编辑属性打开。
```C
// 设置所有项设置为可编辑状态
// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_5_clicked()
{
    int x,cnt;
    QListWidgetItem *aItem;

    cnt = ui->listWidget->count();
    for(x=0;x<cnt;x++)
    {
        aItem = ui->listWidget->item(x);
        aItem->setFlags(Qt::ItemIsSelectable | Qt::ItemIsEditable
                        |Qt::ItemIsUserCheckable |Qt::ItemIsEnabled);
    }
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529363-f2daf6a9-5455-4a61-b22b-f2ac584b2861.png)


<br>

**ListWidget 全选/全不选:** 全选顾名思义就是选中菜单中的所有数据，使用`aItem->setCheckState(Qt::Checked)`实现选中，通过循环计数即可。
```C
// 全选按钮
// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_2_clicked()
{
    int cnt = ui->listWidget->count();   // 获取总数
    for(int x=0;x<cnt;x++)
    {
        QListWidgetItem *aItem = ui->listWidget->item(x);  // 获取到一项指针
        aItem->setCheckState(Qt::Checked);                 // 设置为选中
    }

}

// 全不选
// By: LyShark
void MainWindow::on_pushButton_3_clicked()
{
    int cnt = ui->listWidget->count();   // 获取总数
    for(int x=0;x<cnt;x++)
    {
        QListWidgetItem *aItem = ui->listWidget->item(x);  // 获取到一项指针
        aItem->setCheckState(Qt::Unchecked);               // 设置为非选中
    }
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529346-2272af7b-8708-49aa-8082-7a12e9a3482a.png)


<br>

**ListWidget 反选功能:** 反选的含义是，用户选中菜单反选后会变为未选中状态，未选中则变为选中，只需要增加一个判断即可实现。
```C
// By: LyShark
void MainWindow::on_pushButton_4_clicked()
{
    int x,cnt;
    QListWidgetItem *aItem;

    cnt = ui->listWidget->count();
    for(x=0;x<cnt;x++)
    {
        aItem = ui->listWidget->item(x);
        if(aItem->checkState() != Qt::Checked)
            aItem->setCheckState(Qt::Checked);
        else
            aItem->setCheckState(Qt::Unchecked);
    }
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529334-9323a5bd-699c-4012-8c98-0d94efead495.png)


<br>

**ListWidget 指定位置插入/追加插入:** 在选中行的上方插入一行新的表项，以及追加到末尾一行。
```C
// 指定位置插入一项
// www.cnblogs.com/lyshark
void MainWindow::on_pushButton_8_clicked()
{
    QIcon aIcon;
    aIcon.addFile(":/image/3.ico");

    QListWidgetItem *aItem = new QListWidgetItem("插入的数据");
    aItem->setIcon(aIcon);
    aItem->setCheckState(Qt::Checked);
    aItem->setFlags(Qt::ItemIsSelectable |Qt::ItemIsUserCheckable |Qt::ItemIsEnabled);

    // 在当前行的上方插入一个项
    ui->listWidget->insertItem(ui->listWidget->currentRow(),aItem);
}

// 增加一项,尾部追加
void MainWindow::on_pushButton_7_clicked()
{
    QIcon aIcon;
    aIcon.addFile(":/image/2.ico");

    QListWidgetItem *aItem = new QListWidgetItem("新增的项目");   // 增加项目名
    aItem->setIcon(aIcon);                                       // 设置图标
    aItem->setCheckState(Qt::Checked);                           // 设置为选中
    aItem->setFlags(Qt::ItemIsSelectable |Qt::ItemIsUserCheckable |Qt::ItemIsEnabled);
    ui->listWidget->addItem(aItem);                              // 增加到控件
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529319-d6b380ae-6275-4d48-b5f0-e26e70b42425.png)


<br>

**ListWidget 删除选中项:** 删除当前选中的一项，并清理释放内存。
```C
// 删除选中项
void MainWindow::on_pushButton_6_clicked()
{
    int row = ui->listWidget->currentRow(); // 获取当前行
    QListWidgetItem *aItem = ui->listWidget->takeItem(row);  // 移除指定行的项,但不delete
    delete aItem;                                            // 释放空间
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188529304-2624a457-7592-4a98-83fb-b6623df139dd.png)
