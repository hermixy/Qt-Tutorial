TableWidget 表格结构组件，该组件可以看作是TreeWidget树形组件的高级版，表格组件相比于树结构组件灵活性更高，不仅提供了输出展示二维表格功能，还可以直接对表格元素直接进行编辑与修改操作，表格结构分为表头，表中数据两部分，表格结构可看作一个二维数组，通过数组行列即可锁定特定元素，如下代码是针对表格结构的基本使用方法，分别实现了表头数据的初始化，元素的插入等基本操作。

在研究Widget组件之前先来熟悉一下View组件，View组件相对Widget组件来说只是不具备编辑功能，其他功能保持一致，View组件支持与数据库建立映射关系，如果表格无需更新则最好可以使用View组件，View组件创建表格代码如下。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QStandardItemModel>

QStandardItemModel *model = new QStandardItemModel();

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 初始化tableView表头
    model->setColumnCount(3);
    model->setHeaderData(0,Qt::Horizontal,QString("账号"));
    model->setHeaderData(1,Qt::Horizontal,QString("用户"));
    model->setHeaderData(2,Qt::Horizontal,QString("年龄"));

    ui->tableView->setModel(model);
    ui->tableView->horizontalHeader()->setDefaultAlignment(Qt::AlignLeft);  // 表头居左显示

    //设置列宽
    ui->tableView->setColumnWidth(0,101);
    ui->tableView->setColumnWidth(1,102);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 对表格添加数据
// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_clicked()
{
    for(int i = 0; i < 5; i++)
    {
        model->setItem(i,0,new QStandardItem("20210506"));

        //设置字符颜色
        model->item(i,0)->setForeground(QBrush(QColor(255, 0, 0)));
        //设置字符位置
        model->item(i,0)->setTextAlignment(Qt::AlignCenter);
        model->setItem(i,1,new QStandardItem(QString("lyshark")));

        model->setItem(i,2,new QStandardItem(QString("24")));
    }
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530377-c34a9007-6784-49e1-8f8a-3bbdb427e1f5.png)


Widget组件的初始化与View组件基本保持一致，当程序运行时，首先在构造函数中执行以下代码，对表格进行初始化。
```C
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    QStringList header;
    header << "姓名" << "性别" << "年龄";

    ui->tableWidget->setColumnCount(header.size());                        // 设置表格的列数
    ui->tableWidget->setHorizontalHeaderLabels(header);                    // 设置水平头
    ui->tableWidget->setRowCount(5);                                       // 设置总行数
    ui->tableWidget->setEditTriggers(QAbstractItemView::NoEditTriggers);   // 设置表结构默认不可编辑

    // 初始化右侧的编辑框等属性
    ui->radioButton->setChecked(true);
    ui->lineEdit_1->setText("");
    ui->lineEdit_2->setText("");

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

MainWindow::~MainWindow()
{
    delete ui;
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530404-81ec1e9c-ae3a-4646-8f72-84927a656f90.png)


接着就是对Ui中的按钮增加一些绑定事件，此处我们就通过`connect`绑定信号，绑定以下这几个:
 - ui->pushButton 绑定添加信号
 - ui->pushButton_2 绑定删除信号
 - ui->pushButton_3 绑定获取单元格信号
 - ui->pushButton_4 绑定修改信号

**增加添加按钮信号:** 给添加按钮绑定一个信号槽,点击按钮添加
```C
    connect(ui->pushButton,&QPushButton::clicked,[=](){

        QString Uname = ui->lineEdit_1->text();
        QString Usex = "男";
        int Uage = 0;

        if(ui->radioButton->isChecked())
            Usex = "男";
        if(ui->radioButton_2->isChecked())
            Usex = "女";

        Uage =(ui->lineEdit_2->text()).toInt();

        // 添加之前,先判断Uname是否存在于TableWidget中,如果存在返回0不存在返回1
        bool isEmpty = ui->tableWidget->findItems(Uname,Qt::MatchExactly).empty();
        if(isEmpty)
        {
            ui->tableWidget->insertRow(0);    // 在行首添加一行空列表
            ui->tableWidget->setItem(0,0,new QTableWidgetItem(Uname));
            ui->tableWidget->setItem(0,1,new QTableWidgetItem(Usex));
            ui->tableWidget->setItem(0,2,new QTableWidgetItem( QString::number(Uage)));
        }
    });
```
**增加删除按钮信号:** 点击按钮删除选中行
```C
    connect(ui->pushButton_2,&QPushButton::clicked,[=](){
        bool isEmpty = ui->tableWidget->findItems(ui->lineEdit_1->text(),Qt::MatchExactly).empty();
        if(!isEmpty)
        {
            // 定位到所在行行号
            int row = ui->tableWidget->findItems(ui->lineEdit_1->text(),Qt::MatchExactly).first()->row();
            // 释放资源
            ui->tableWidget->removeRow(row);
        }
    });
```
**增加释放单元格按钮信号:** 获取当前选中单元,并释放当前单格
```C
    connect(ui->pushButton_3,&QPushButton::clicked,[=](){
        int row = ui->tableWidget->currentRow();
        std::cout << row << std::endl;

        QTableWidgetItem *table =  ui->tableWidget->currentItem();
        delete(table);
    });
```
**增加修改单元格按钮信号:** 添加修改指定内容的处理流程
```C
    connect(ui->pushButton_4,&QPushButton::clicked,[=](){
        QTableWidgetItem *cellItem;

        // 取出当前选中行
        int curr_row = ui->tableWidget->currentRow();

        // 循环列数
        // https://www.cnblogs.com/lyshark
        for(int col=0; col<ui->tableWidget->columnCount(); col++)
        {
            // 寻找到当前列的指针
            cellItem = ui->tableWidget->item(curr_row,col);

            // 循环输出列名称
            std::cout << cellItem->text().toStdString().data() << std::endl;

            // 先来处理第一个姓名,读出来并写回到列表第0列
            if(col == 0)
                cellItem->setText(ui->lineEdit_1->text());

            // 判断性别,并分别写回到第1列
            if(col == 1)
            {
                if(ui->radioButton->isChecked())
                    cellItem->setText("男");
                if(ui->radioButton_2->isChecked())
                    cellItem->setText("女");
            }

            // 判断年龄,并写回到第3列
            if(col == 2)
                cellItem->setText(ui->lineEdit_2->text());
        }
    });
```

信号绑定后,代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530420-1838a505-b78d-46ec-ab1d-465b9a7a6c25.png)
