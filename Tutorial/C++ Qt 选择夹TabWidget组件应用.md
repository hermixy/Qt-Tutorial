在Qt中通过使用选择夹组件可以实现在一个页面中集成多种功能，我们以`TabWidget`选择夹组件为例，实现在单个页面中集成多个功能，并给每一个子夹增加对应的Ico图标。

如果我们使用选择夹组件，必须提前拖入UI界面中(无法代码生成)，如下我们找到`TabWidget`并将其拖入UI界面中。

![image](https://user-images.githubusercontent.com/52789403/188528601-b38c9f42-75b2-42cc-89e2-a97328ca78d4.png)

其次需要增加与美化代码对应的子夹数量，这里我们分别增加三个子夹，此处只需要增加不需要重命名。

![image](https://user-images.githubusercontent.com/52789403/188528578-9fe84be4-c214-4b49-92e9-2bd43f4a4dba.png)

接着我们需要增加三个子夹对应的图标组，插入图标组需要执行以下步骤。

 - 选择Forms -> 右键(AddNew) -> Qt -> Qt Resource File -> 命名为 res

![image](https://user-images.githubusercontent.com/52789403/188528558-7a924930-6e81-4a20-859d-d8aa24f0d258.png)

 - 添加前缀/ -> 添加文件 -> 导入所有ICO文件.

![image](https://user-images.githubusercontent.com/52789403/188528538-c78cef23-8f57-4705-8e7f-1778a5b763b3.png)


通过上方的配置后，我们的资源就会被编译为二进制文件，此时通过代码中使用`QIcon(":/image/1.ico")`相对路径即可引入到项目中。

```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 全局配置tabWidget选项卡
    ui->tabWidget->setTabPosition(QTabWidget::North);       // 设置选项卡方位
    ui->tabWidget->setIconSize(QSize(50, 25));              // 设置图标整体大小
    ui->tabWidget->setTabShape(QTabWidget::Triangular);     // 设置选项卡形状
    ui->tabWidget->setMovable(true);                        // 设置选项卡是否可拖动
    ui->tabWidget->usesScrollButtons();                     // 选项卡滚动

    // 设置选项卡1
    ui->tabWidget->setTabText(0,QString("进制转换标签"));           // 设置选项卡文本
    ui->tabWidget->setTabIcon(0,QIcon(":/image/1.ico"));          // 设置选项卡图标
    ui->tabWidget->setTabToolTip(0,QString("SpinBox 与进制转换"));  // 设置鼠标悬停提示

    // 设置选项卡2
    ui->tabWidget->setTabText(1,QString("颜色配置标签"));          // 设置选项卡文本
    ui->tabWidget->setTabIcon(1,QIcon(":/image/2.ico"));         // 设置选项卡图标
    ui->tabWidget->setTabToolTip(1,QString("滑块条的使用"));       // 设置鼠标悬停提示

    // 设置选项卡3
    ui->tabWidget->setTabText(2,QString("系统配置标签"));          // 设置选项卡文本
    ui->tabWidget->setTabIcon(2,QIcon(":/image/3.ico"));         // 设置选项卡图标
    ui->tabWidget->setTabToolTip(2,QString("圆形组件与数码表"));    // 设置鼠标悬停提示
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

我们直接在代码中初始化这些选择夹即可实现增加图标以及字体等功能，运行后代码如下所示。

![image](https://user-images.githubusercontent.com/52789403/188528521-72987a85-a695-4e67-956d-d8b03817f359.png)

