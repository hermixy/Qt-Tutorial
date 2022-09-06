在Qt中对话框分为两种形式，一种是标准对话框，另一种则是自定义对话框，在一般开发过程中标准对话框使用是最多的了，标准对话框一般包括 QMessageBox,QInputDialog,QFileDialog 这几种，这里我将总结本人在开发过程中常用到的标准对话框的使用技巧。

Qt框架下,常用的标准对话框有下面这几种:

 - QMessageBox 提示信息框
 - QInputDialog 基本输入对话框(文本输入,整数输入,浮点数输入,单选框输入)
 - QFileDialog 文件选择对话框(选择文件,多选文件,保存文件)

<br>

**QMessageBox 消息弹窗:** 消息对话框用于提示用户，常见的有四种分别是:提示,警告,错误,确认,代码归纳如下所示。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QMessageBox>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// By : LyShark
// https://www.cnblogs.com/lyshark
// 弹出各种MessageBox
void MainWindow::on_pushButton_clicked()
{
    QString dlgTitle="消息框";
    QString strInfo="文件已被修改，是否保存修改 ?";

    QMessageBox::StandardButton defaultBtn = QMessageBox::NoButton; // 缺省按钮
    QMessageBox::StandardButton result;                             // 返回选择的按钮

    // 弹窗分类 Question information warning critical
    result=QMessageBox::question(this, dlgTitle, strInfo,QMessageBox::Yes|QMessageBox::No |QMessageBox::Cancel,defaultBtn);

    if (result==QMessageBox::Yes)
        ui->plainTextEdit->appendPlainText("Question消息框: Yes 被选择");
    else if(result==QMessageBox::No)
        ui->plainTextEdit->appendPlainText("Question消息框: No 被选择");
    else if(result==QMessageBox::Cancel)
        ui->plainTextEdit->appendPlainText("Question消息框: Cancel 被选择");
    else
        ui->plainTextEdit->appendPlainText("Question消息框: 无选择");
}

// 弹出关于提示
void MainWindow::on_pushButton_2_clicked()
{
    QString dlgTitle="about 消息框";
    QString strInfo="我开发的数据查看软件 V1.0 \n 保留所有版权";
    QMessageBox::about(this, dlgTitle, strInfo);
}
```

![image](https://user-images.githubusercontent.com/52789403/188528716-98a9d735-3038-4f23-8470-606223c5e366.png)


**QMessageBox 退出事件:** 弹窗组件还可以配合QCloseEvent实现事件通知机制，例如当窗体被关闭则提示用户是否关闭窗体。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QMessageBox>
#include <QCloseEvent>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

// 窗口关闭时询问是否退出
void MainWindow::closeEvent(QCloseEvent *event)
{
   QMessageBox::StandardButton result=QMessageBox::question(this, "确认", "确定要退出本程序吗？",
                      QMessageBox::Yes|QMessageBox::No |QMessageBox::Cancel,
                      QMessageBox::No);

    if (result==QMessageBox::Yes)
        event->accept();
    else
        event->ignore();
}

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::~MainWindow()
{
    delete ui;
}
```

![image](https://user-images.githubusercontent.com/52789403/188528700-7d06556a-e175-4ebf-9d02-5f1778c5fe1c.png)


**QInputDialog 对话框:** 该对话框长用于输入一段特殊的文本,浮点数,或者选择一个列表框中的选项，该功能用于简单的用户交互场景。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QLineEdit>
#include <QInputDialog>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 文本输入对话框
void MainWindow::on_pushButton_clicked()
{
    QString dlgTitle="输入文字对话框";
    QString txtLabel="请输入文件名";
    QString defaultInput="新建文件.txt";
    QLineEdit::EchoMode echoMode=QLineEdit::Normal;       // 正常文字输入
    // QLineEdit::EchoMode echoMode=QLineEdit::Password;  // 密码输入

    bool flag = false;
    QString text = QInputDialog::getText(this, dlgTitle,txtLabel, echoMode,defaultInput, &flag);
    if (flag && !text.isEmpty())
    {
        ui->plainTextEdit->appendPlainText(text);
    }
}

// 整数数值输入对话框
// By : LyShark
// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_2_clicked()
{
    QString dlgTitle="输入整数对话框";
    QString txtLabel="设置字体大小";
    int defaultValue=ui->plainTextEdit->font().pointSize();   // 现有字体大小
    int minValue=6, maxValue=50, stepValue=1;                 // 范围(步长)
    bool flag=false;
    int inputValue = QInputDialog::getInt(this, dlgTitle,txtLabel,defaultValue, minValue,maxValue,stepValue,&flag);
    if (flag)
    {
        QFont font=ui->plainTextEdit->font();
        font.setPointSize(inputValue);
        ui->plainTextEdit->setFont(font);
    }
}

// 浮点数输入对话框
void MainWindow::on_pushButton_3_clicked()
{
    QString dlgTitle="输入浮点数对话框";
    QString txtLabel="输入一个浮点数";
    float defaultValue=3.13;

    float minValue=0, maxValue=10000;  // 范围
    int decimals=2;                    // 小数点位数

    bool flag=false;
    float inputValue = QInputDialog::getDouble(this, dlgTitle,txtLabel,defaultValue, minValue,maxValue,decimals,&flag);
    if (flag)
    {
        QString str=QString::asprintf("输入了一个浮点数:%.2f",inputValue);
        ui->plainTextEdit->appendPlainText(str);
    }
}

// 单选框条目选择对话框
void MainWindow::on_pushButton_4_clicked()
{
    QStringList items;                        // 列表内容
    items <<"优秀"<<"良好"<<"合格"<<"不合格";    // 放入列表

    QString dlgTitle="条目选择对话框";
    QString txtLabel="请选择级别";
    int curIndex=0; //初始选择项
    bool editable=false;                       // 是否可编辑
    bool flag=false;
    QString text = QInputDialog::getItem(this, dlgTitle,txtLabel,items,curIndex,editable,&flag);

    if (flag && !text.isEmpty())
    {
        ui->plainTextEdit->appendPlainText(text);
    }
}
```

![image](https://user-images.githubusercontent.com/52789403/188528683-b9ada131-a21a-4557-919f-10a51af15919.png)


**QFileDialog 对话框:** 该对话框用于对文本的操作，例如打开文件，保存文件，选择文件夹等，当点击选择后，对话框会自动提取出文件路径。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QFileDialog>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 选择单个文件对话框
void MainWindow::on_pushButton_clicked()
{
    QString curPath=QDir::currentPath();                                       // 获取系统当前目录
//  QString  curPath=QCoreApplication::applicationDirPath();                   // 获取应用程序的路径
    QString dlgTitle="选择一个文件";                                             // 对话框标题
    QString filter="文本文件(*.txt);;图片文件(*.jpg *.gif *.png);;所有文件(*.*)";  // 文件过滤器

    QString aFileName=QFileDialog::getOpenFileName(this,dlgTitle,curPath,filter);

    if (!aFileName.isEmpty())
    {
        ui->plainTextEdit->appendPlainText(aFileName);
    }
}

// 选择多个文件对话框
// By : LyShark
// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_2_clicked()
{
    // QString curPath=QCoreApplication::applicationDirPath();                // 获取应用程序的路径
    QString curPath=QDir::currentPath();                                      // 获取系统当前目录
    QString dlgTitle="选择多个文件";                                            // 对话框标题
    QString filter="文本文件(*.txt);;图片文件(*.jpg *.gif *.png);;所有文件(*.*)"; // 文件过滤器

    QStringList fileList=QFileDialog::getOpenFileNames(this,dlgTitle,curPath,filter);
    for (int i=0; i<fileList.count();i++)
    {
        // 循环将文件路径添加到列表中
        ui->plainTextEdit->appendPlainText(fileList.at(i));
    }
}

// 选择文件夹
void MainWindow::on_pushButton_3_clicked()
{
    QString curPath=QCoreApplication::applicationDirPath();    // 获取应用程序的路径
    // QString curPath=QDir::currentPath();                    // 获取系统当前目录

    // 调用打开文件对话框打开一个文件
    QString dlgTitle="选择一个目录";                             // 对话框标题
    QString selectedDir=QFileDialog::getExistingDirectory(this,dlgTitle,curPath,QFileDialog::ShowDirsOnly);
    if (!selectedDir.isEmpty())
    {
        ui->plainTextEdit->appendPlainText(selectedDir);
    }
}

// 保存文件对话框
void MainWindow::on_pushButton_4_clicked()
{
    QString curPath=QCoreApplication::applicationDirPath();                  // 获取应用程序的路径
    QString dlgTitle="保存文件";                                              // 对话框标题
    QString filter="文本文件(*.txt);;h文件(*.h);;C++文件(.cpp);;所有文件(*.*)"; // 文件过滤器
    QString aFileName=QFileDialog::getSaveFileName(this,dlgTitle,curPath,filter);
    if (!aFileName.isEmpty())
    {
        ui->plainTextEdit->appendPlainText(aFileName);
    }
}
```

![image](https://user-images.githubusercontent.com/52789403/188528667-544db447-da7e-4555-8ca4-d4acdd03638a.png)
