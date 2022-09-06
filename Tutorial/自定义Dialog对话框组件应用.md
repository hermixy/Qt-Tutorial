在上一篇博文 `《C/C++ Qt 标准Dialog对话框组件应用》` 中我给大家演示了如何使用Qt中内置的标准对话框组件实现基本的数据输入功能。

但有时候我们需要一次性修改多个数据，使用默认的模态对话框似乎不太够用，此时我们需要自己创建一个自定义对话框，这类对话框也是一种窗体，所以可以在其上面放置任何的通用组件，以实现更多复杂的开发需求。

目前自定义对话框与主窗体的通信有两种方式，一种是通过`函数实现通信`，另一种则是通过`信号实现通信`，我们以通过函数通信为基础，解释一下如何实现跨窗体通信。

首先需要创建一个自定义对话框，对话框具体创建流程如下

 - 选择项目 -> AddNew -> QT -> Qt设计师界面类 -> 选择空白Dialog -> 命名为Dialog保存

![](/image/1379525-20211125162123979-1287081757.png)

直接选中`Dianlog.ui`并绘制界面为以下，一个编辑框，两个按钮。

![](/image/1379525-20211125162447420-1604947219.png)

其次需要在`Dialog`对话框上增加`两个信号`，分别是`点击`和`关闭`，并将信号关联到两个槽函数上，其信号应该写成如下样子。

![](/image/1379525-20211125162528677-1496994988.png)

接着我们点开`dialog.cpp`这个类则是对话框类，类内需要定义两个成员函数，它们的功能如下：
 - 第一个 `GetValue()` 用来获取当前编辑框内的数据并将数据返回给父窗体。
 - 第二个 `SetValue()` 用来接收传入的参数，并将此参数设置到自身窗体中的编辑框内。

```C
#include "dialog.h"
#include "ui_dialog.h"

Dialog::Dialog(QWidget *parent) :QDialog(parent),ui(new Ui::Dialog)
{
    ui->setupUi(this);
}

// 用于MainWindow获取编辑框中的数据
QString Dialog::GetValue()
{
    return ui->lineEdit->text();
}

// 用于设置当前编辑框中的数据为MainWindow
// https://www.cnblogs.com/lyshark
void Dialog::SetValue(QString x)
{
    ui->lineEdit->setText(x);
}

Dialog::~Dialog()
{
    delete ui;
}

void Dialog::on_BtnOk_clicked()
{

}
void Dialog::on_BtnCancel_clicked()
{

}
```
对于主函数来说，当用户点击`on_pushButton_clicked()`按钮时，我们需要动态将自己创建的`Dialog`加载，读取出主窗体编辑框内的值并设置到子窗体内，当用户按下`QDialog::Accepted`时则是获取子窗体内的值，并将其设置到父窗体的编辑框内，主函数代码如下所示.
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include "dialog.h"
#include <iostream>
#include <QDialog>

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->lineEdit->setEnabled(false);
    ui->lineEdit->setText("hello lyshark");
}

MainWindow::~MainWindow()
{
    delete ui;
}

// By: LyShark
// https://www.cnblogs.com/lyshark
// 按钮点击后执行
void MainWindow::on_pushButton_clicked()
{
    // 创建模态对话框
    Dialog *ptr = new Dialog(this);                                 // 创建一个对话框
    Qt::WindowFlags flags = ptr->windowFlags();                     // 需要获取返回值
    ptr->setWindowFlags(flags | Qt::MSWindowsFixedSizeDialogHint);  // 设置对话框固定大小

    // 读取MainWindows参数并设置到Dialog
    QString item = ui->lineEdit->text();
    ptr->SetValue(item);

    int ref = ptr->exec();             // 以模态方式显示对话框
    if (ref==QDialog::Accepted)        // OK键被按下,对话框关闭
    {
        // 当BtnOk被按下时,则设置对话框中的数据
        QString the_value = ptr->GetValue();
        std::cout << "value = " << the_value.toStdString().data() << std::endl;
        ui->lineEdit->setText(the_value);
    }

    // 删除释放对话框句柄
    delete ptr;
}
```

具体演示代码如下所示:

![](/image/1379525-20211125164225983-1615458964.gif)

<br>

而对于`信号版`来说，我们需要在`dialog.h`头文件中增加`sendText()`信号，以及`on_pushButton_clicked()`槽函数的声明。
```C
#ifndef DIALOG_H
#define DIALOG_H

#include <QDialog>

namespace Ui {
class Dialog;
}

class Dialog : public QDialog
{
    Q_OBJECT

public:
    explicit Dialog(QWidget *parent = nullptr);
    ~Dialog();

// By: LyShark
// https://www.cnblogs.com/lyshark
private:
    Ui::Dialog *ui;


// 定义信号(信号只需声明无需实现)
signals:
    void sendText(QString str);

private slots:
    void on_pushButton_clicked();
};

#endif // DIALOG_H
```
`dialog.cpp`中则在构造函数中建立连接，并提供一个发送到MainWindow中的按钮.
```C
#include "dialog.h"
#include "ui_dialog.h"

// By: LyShark
// https://www.cnblogs.com/lyshark
Dialog::Dialog(QWidget *parent) :QDialog(parent),ui(new Ui::Dialog)
{
    ui->setupUi(this);
    connect(ui->pushButton, SIGNAL(clicked()), this, SLOT(onBtnClick()));
}

Dialog::~Dialog()
{
    delete ui;
}

// 发送信号到MainWindow
void Dialog::on_pushButton_clicked()
{
    QString send_data = ui->lineEdit->text();
    emit sendText(send_data);
}
```
主窗体头文件`mainwindow.h`中定义`receiveMsg`接受数据的槽函数.
```C
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>

namespace Ui {
class MainWindow;
}

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

// By: LyShark
// https://www.cnblogs.com/lyshark
private slots:
    // 定义槽函数
    void receiveMsg(QString str);
    void on_pushButton_clicked();

private:
    Ui::MainWindow *ui;
};

#endif // MAINWINDOW_H
```
并在`mainwindow.cpp`中实现这个槽函数。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include "dialog.h"
#include <QDialog>

// By: LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->lineEdit->setEnabled(false);
}

// 接收信号并设置到LineEdit上
void MainWindow::receiveMsg(QString str)
{
    ui->lineEdit->setText(str);
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_pushButton_clicked()
{
    Dialog *subwindow = new Dialog(this);
    // 当收到sendText信号时使用receiveMsg槽函数处理
    connect(subwindow, SIGNAL(sendText(QString)), this, SLOT(receiveMsg(QString)));
    subwindow->show();
}
```
代码运行后与基于函数版的基本一致，但在灵活性上来说信号版更好一些。

![](/image/1379525-20211125165731172-1586209119.gif)

自定义对话框基本就这些内容，灵活运行这些组件，很容易就能实现一些有用的表格编辑器。

![](/image/1379525-20211126100031481-527949606.gif)
