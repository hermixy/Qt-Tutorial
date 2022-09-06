在Qt中通过使用选择夹组件可以实现在一个页面中集成多种功能，我们以`TabWidget`选择夹组件为例，实现在单个页面中集成多个功能，并给每一个子夹增加对应的Ico图标。

如果我们使用选择夹组件，必须提前拖入UI界面中(无法代码生成)，如下我们找到`TabWidget`并将其拖入UI界面中。

![](/image/1379525-20211123133203300-1131260999.png)

其次需要增加与美化代码对应的子夹数量，这里我们分别增加三个子夹，此处只需要增加不需要重命名。

![](/image/1379525-20211123133358819-2092609339.png)

接着我们需要增加三个子夹对应的图标组，插入图标组需要执行以下步骤。

 - 选择Forms -> 右键(AddNew) -> Qt -> Qt Resource File -> 命名为 res

![](/image/1379525-20211123133709555-2048904733.png)

 - 添加前缀/ -> 添加文件 -> 导入所有ICO文件.

![](/image/1379525-20211123133822044-1400647896.png)

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

![](/image/1379525-20211123133858076-1494608574.png)
