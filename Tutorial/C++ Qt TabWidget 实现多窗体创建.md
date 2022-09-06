在开发窗体应用时通常会伴随分页，TabWidget组件配合自定义Dialog组件，可实现一个复杂的多窗体分页结构，此类结构也是ERP等软件通用的窗体布局方案。

首先先来实现一个只有`TabWidget`分页的简单结构，如下窗体布局，布局中空白部分是一个`TabWidget`组件，下方是一个按钮，当用户点击按钮时，自动将该窗体新增到`TabWidget`组件中。

![image](https://user-images.githubusercontent.com/52789403/188530695-70f1a654-335d-4da4-afd4-33020ebdb19f.png)


该页面关联代码如下所示，当用户点击`on_pushButton_clicked()`时自动新增一个窗体并将窗体的`Tab`设置为指定的IP地址。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <iostream>

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->tabWidget->setVisible(false);
    ui->tabWidget->clear();//清除所有页面
    ui->tabWidget->tabsClosable(); //Page有关闭按钮，可被关闭
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 定义函数来获取当前Table名字
QString MainWindow::GetTableNumber()
{
    QString ref = QString(ui->tabWidget->currentIndex());
    return ref;
}

// https://www.cnblogs.com/lyshark
void MainWindow::on_pushButton_clicked()
{
    FormDoc *ptr = new FormDoc(this);                // 新建选项卡
    ptr->setAttribute(Qt::WA_DeleteOnClose);         // 关闭时自动销毁

    int cur = ui->tabWidget->addTab(ptr,QString::asprintf(" 192.168.1.%d",ui->tabWidget->count()));

    ui->tabWidget->setTabIcon(cur,QIcon(":/image/1.ico"));

    ui->tabWidget->setCurrentIndex(cur);
    ui->tabWidget->setVisible(true);
}

// 关闭Tab时执行
void MainWindow::on_tabWidget_tabCloseRequested(int index)
{
    if (index<0)
        return;
    QWidget* aForm=ui->tabWidget->widget(index);
    aForm->close();
}

// 在无Tab页面是默认禁用
void MainWindow::on_tabWidget_currentChanged(int index)
{
    Q_UNUSED(index);
    bool en=ui->tabWidget->count()>0;
    ui->tabWidget->setVisible(en);
}
```
其中的每一个`Dialog`子窗体，都需要动态获取父窗体指针，当需要操作时则可以根据指针对自身进行操作，子窗体代码如下.
```C
#include "formdoc.h"
#include "ui_formdoc.h"
#include "mainwindow.h"

#include <QVBoxLayout>
#include <iostream>

FormDoc::FormDoc(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::FormDoc)
{
    ui->setupUi(this);

    QVBoxLayout *Layout = new QVBoxLayout();
    Layout->setContentsMargins(2,2,2,2);
    Layout->setSpacing(2);
    this->setLayout(Layout);

    MainWindow *parWind = (MainWindow*)parentWidget(); //获取父窗口指针
    QString ref = parWind->GetTableNumber();           // 获取选中标签索引
    std::cout << ref.toStdString().data() << std::endl;   // By: LyShark
}

FormDoc::~FormDoc()
{
    delete ui;
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530724-326d96e0-7661-46f1-9009-bbeb6531d30b.png)


Tab组件如果配合ToolBar组件可以实现更多有意思的功能，例如下面这个案例:

![image](https://user-images.githubusercontent.com/52789403/188530738-c55b67fe-9701-4145-b611-bfc3f4ec782c.png)
