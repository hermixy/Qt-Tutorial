StringListModel 字符串列表映射组件，该组件用于处理字符串与列表框组件中数据的转换，通常该组件会配合ListView组件一起使用，例如将ListView组件与Model模型绑定，当ListView组件内有数据更新时，我们就可以利用映射将数据模型中的数值以字符串格式提取出来，同理也可实现将字符串赋值到指定的ListView组件内。

首先在UI界面中排版

![image](https://user-images.githubusercontent.com/52789403/188531070-6f8b5d81-36b0-4ce9-b654-1f10aec0134c.png)

默认的`MainWindow::MainWindow`构造函数中，我们首先初始化一个`QStringList`字符串链表并对该链表赋值，通过`new QStringListModel(this);`创建一个数据模型，并通过`ui->listView->setModel(model);`属性将模型与ListView组件绑定，当ListView组件被选中是则触发`on_listView_clicked`事件实现输出当前选中行，其初始化代码部分如下:
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QStringList>
#include <QStringListModel>

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 初始化一个StringList字符串列表
    QStringList theStringList;
    theStringList << "北京" << "上海" << "广州";

    // 创建并使用数据模型
    model = new QStringListModel(this);     // 创建模型
    model->setStringList(theStringList);    // 导入模型数据

    ui->listView->setModel(model);          // 为listView设置模型
    ui->listView->setEditTriggers(QAbstractItemView::DoubleClicked |
                                  QAbstractItemView::SelectedClicked);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 当ListView列表项被选中时，显示QModelIndex的行、列号
void MainWindow::on_listView_clicked(const QModelIndex &index)
{
        ui->LabInfo->setText(QString::asprintf("当前项:row=%d, column=%d",
                            index.row(),index.column()));
}
```

代码运行效果:

![image](https://user-images.githubusercontent.com/52789403/188531052-af0e9308-f168-492a-bc2a-1fdd20c7c4ab.png)


添加代码：需要通过`model->index()`获取到最后一行的索引，然后使用`model->setData()`追加写入数据到最后一条索引位置。
插入代码: 需要通过`ui->listView->currentIndex()`获取到当前光标位置，并调用`model->setData()`插入到指定位置。
删除代码: 直接调用`model->removeRows()`等函数即可将指定位置删除。
```C
// 添加一行
void MainWindow::on_btnListAppend_clicked()
{
    model->insertRow(model->rowCount());                       // 在尾部插入一行
    QModelIndex index = model->index(model->rowCount()-1,0);   // 获取最后一行的索引
    QString LineText = ui->lineEdit->text();
    model->setData(index,LineText,Qt::DisplayRole);            // 设置显示文字
    ui->listView->setCurrentIndex(index);                      // 设置当前行选中
    ui->lineEdit->clear();
}

// 插入一行数据到ListView
void MainWindow::on_btnListInsert_clicked()
{
    QModelIndex index;

    index= ui->listView->currentIndex();             // 获取当前选中行
    model->insertRow(index.row());                   // 在当前行的前面插入一行
    QString LineText = ui->lineEdit->text();
    model->setData(index,LineText,Qt::DisplayRole);             // 设置显示文字
    model->setData(index,Qt::AlignRight,Qt::TextAlignmentRole); // 设置对其方式
    ui->listView->setCurrentIndex(index);                       // 设置当前选中行
}

// 删除当前选中行
void MainWindow::on_btnListDelete_clicked()
{
    QModelIndex index;
    index = ui->listView->currentIndex();    // 获取当前行的ModelIndex
    model->removeRow(index.row());           // 删除选中行
}

// 清除当前列表
void MainWindow::on_btnListClear_clicked()
{
   model->removeRows(0,model->rowCount());
}
```

代码运行效果:

![image](https://user-images.githubusercontent.com/52789403/188531025-122a211f-d45d-42c9-b7dc-6a0c227aa379.png)


如果需要实现将`ListView`数据模型中的数据导出到`plaintextEdit`组件中，则需要通过`model->stringList()`获取到ListView中的每行并将其赋值到`QStringList`字符串链表中，最后通过循环的方式依次插入到`plainTextEdit`中即可，插入时默认会以逗号作为分隔符。
```C
// 显示数据模型文本到QPlainTextEdit
void MainWindow::on_btnTextImport_clicked()
{
    QStringList pList;

    pList = model->stringList();    // 获取数据模型的StringList
    ui->plainTextEdit->clear();     // 先清空文本框

    // 循环追加数据
    for(int x=0;x< pList.count();x++)
    {
        ui->plainTextEdit->appendPlainText(pList.at(x) + QString(","));
    }
}
```

代码运行效果:

![image](https://user-images.githubusercontent.com/52789403/188530995-0c228367-2fda-4de6-b952-a709e5c566da.png)
