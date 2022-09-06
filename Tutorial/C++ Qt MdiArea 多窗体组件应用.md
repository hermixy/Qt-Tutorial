MDI多窗体组件，主要用于设计多文档界面应用程序，该组件具备有多种窗体展示风格，其实现了在父窗体中内嵌多种子窗体的功能，使用MDI组件需要在UI界面中增加mdiArea控件容器，我们所有的窗体创建与操作都在这个容器内进行，如下我们将具体介绍该组件的常用使用技巧。

MDI窗体控件类似于画布，该控件只具备展示窗体的功能，无法实现生成窗体，所以我们需要在项目中手动增加自定义的`Dialog`对话框，并对该对话框进行一定的定制。

![image](https://user-images.githubusercontent.com/52789403/188530059-e4548352-e08f-4330-9ab1-a23796245a24.png)


这个Dialog对话框我们只增加两个功能，一个`Dialog::currentFileName()`获取窗体标题，另一个`Dialog::SetData(QString data)`设置数据到编辑框，代码实现如下.
```C
#include "dialog.h"
#include "ui_dialog.h"

Dialog::Dialog(QWidget *parent) :QDialog(parent),ui(new Ui::Dialog)
{
    ui->setupUi(this);

    this->setWindowTitle("New Doc <By: LyShark >");           // 窗口标题
    this->setAttribute(Qt::WA_DeleteOnClose);  // 关闭时自动删除
    this->setFixedSize(200,100);               // 设置窗体大小
    // this->setWindowIcon(QIcon(":/image/1.ico"));
}

Dialog::~Dialog()
{
    delete ui;
}

// 获取窗体标题
// By: LyShark
QString Dialog::currentFileName()
{
    QString title = this->windowTitle();
    return title;
}

// 设置编辑框内容
// https://www.cnblogs.com/lyshark
void Dialog::SetData(QString data)
{
    ui->lineEdit->setText(data);
}
```

接着我们开始绘制这个程序的主界面，在`toolBar`中增加相应的菜单栏，并在主窗体中放入`mdiArea`容器组件。

![image](https://user-images.githubusercontent.com/52789403/188530091-c4e6c9b1-74a4-4c99-b3ee-0e7b30783bf9.png)


窗体中的顶部菜单栏，我们需要手动定义一下他们所具备的功能名称等。

![image](https://user-images.githubusercontent.com/52789403/188530105-761a61e3-8bed-4661-ac1b-7d1c1455e694.png)


当程序启动后，程序调用`MainWindow`初始化这个窗体，初始化代码如下:
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include "dialog.h"
#include <iostream>
#include <QCloseEvent>

// 如果直接关闭,则清空所有对话框
// https://www.cnblogs.com/lyshark
void MainWindow::closeEvent(QCloseEvent *event)
{
    ui->mdiArea->closeAllSubWindows();
    event->accept();
}

// By: LyShark
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    this->setCentralWidget(ui->mdiArea);
    //this->setWindowState(Qt::WindowMaximized); //窗口最大化显示
    ui->mainToolBar->setToolButtonStyle(Qt::ToolButtonTextUnderIcon);
    ui->mdiArea->setViewMode(QMdiArea::SubWindowView); //子窗口模式
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530123-71b043e8-4a85-4f39-9a16-f5694fc35a2b.png)


用户新建窗体执行`MainWindow::on_actionOpen_triggered()`事件，关闭窗体时则执行`MainWindow::on_actionClose_triggered()`事件。
```C
// 新建窗体
void MainWindow::on_actionOpen_triggered()
{
    Dialog *formDoc = new Dialog(this); //
    ui->mdiArea->addSubWindow(formDoc); //文档窗口添加到MDI
    formDoc->show(); //在单独的窗口中显示
}
// 关闭全部
void MainWindow::on_actionClose_triggered()
{
    ui->mdiArea->closeAllSubWindows(); //关闭所有子窗口
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530150-07847e20-8772-4a30-8774-a7640588d3d8.png)


当用户点击MDI模式时，我们则执行以下代码，将所有已存在的窗体合并为一个类似于`TabWidget`的窗体组件。
```C
// 转为MID模式
void MainWindow::on_actionMID_triggered(bool checked)
{
    // Tab多页显示模式
    if (checked)
    {
        ui->mdiArea->setViewMode(QMdiArea::TabbedView); // Tab多页显示模式
        ui->mdiArea->setTabsClosable(true);             // 页面可关闭
        ui->actionLine->setEnabled(false);
        ui->actionTile->setEnabled(false);
    }
    // 子窗口模式
    else
    {
        ui->mdiArea->setViewMode(QMdiArea::SubWindowView); // 子窗口模式
        ui->actionLine->setEnabled(true);
        ui->actionTile->setEnabled(true);
    }
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530164-666fb24d-2f84-4f4c-a22a-d67a40ec4cee.png)


窗体级联模式则是将窗体并排排列在一起，我们只需要调用`ui->mdiArea->cascadeSubWindows();`方法即可实现.
```C
// 级联模式
void MainWindow::on_actionLine_triggered()
{
    ui->mdiArea->cascadeSubWindows();
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530182-b3741274-3a5b-439f-a815-efff16b2ae8a.png)


平铺模式同样使用`ui->mdiArea->tileSubWindows();`即可实现转换。
```C
// 平铺模式
void MainWindow::on_actionTile_triggered()
{
    ui->mdiArea->tileSubWindows();
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530200-cd98be35-f092-41fc-8716-0a2e704d615a.png)


最后一个功能是主窗体发送数据到子窗体，该功能的实现需要两个函数。

 - on_mdiArea_subWindowActivated 实现设置主窗体名字到自身
 - on_actionSendMsg_triggered 实现主窗体发送消息到子窗体内

```C
// 当子窗体打开时获取到其窗体标题
// By: LyShark
void MainWindow::on_mdiArea_subWindowActivated(QMdiSubWindow *arg1)
{
    Q_UNUSED(arg1);

    // 若子窗口个数为零,则将statusBar置空
    if (ui->mdiArea->subWindowList().count()==0)
    {
        ui->statusBar->clearMessage();
    }
    else
    {
        // 如果不为0则显示主窗口的文件名
        Dialog *formDoc=static_cast<Dialog*>(ui->mdiArea->activeSubWindow()->widget());
        ui->statusBar->showMessage(formDoc->currentFileName());
    }
}

// 对选中窗体发送数据
// https://www.cnblogs.com/lyshark
void MainWindow::on_actionSendMsg_triggered()
{
    // 先获取当前MDI子窗口
    Dialog *formDoc;

    // 如果打开则获取活动窗体
    if (ui->mdiArea->subWindowList().count() > 0)
    {
        formDoc=(Dialog*)ui->mdiArea->activeSubWindow()->widget();
        // 对活动窗体设置数据
        formDoc->SetData("hello lyshark");
    }
}
```

代码运行效果如下:

![image](https://user-images.githubusercontent.com/52789403/188530213-4c627733-01c3-46c7-b283-88f9b654bdd4.png)
