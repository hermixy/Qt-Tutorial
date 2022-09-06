QT 是一个跨平台C++图形界面开发库，利用QT可以快速开发跨平台窗体应用程序，在QT中我们可以通过拖拽的方式将不同组件放到指定的位置，实现图形化开发极大的方便了开发效率。

目前，QT开发中常用的基础组件有以下几种：

 - PushButton 按钮组件
 - LineEdit 单行输入组件
 - SpinBox 数值组件
 - HorizontalSlider 滑块条组件
 - LCDNumber 数码表与LCD屏幕
 - ComBox 下拉框组件
 - ProgressBar 进度条与定时器
 - DateTime 日期与时间组件
 - PlainTextEdit 多行文本框
 - RadioButton 单选框分组

如上方列表中提到的的组件，就是在开发中经常被使用的，这些组件我将通过一个个小案例，帮助大家理解组件的应用方式与应用场景。
<br>

**PushButton 按钮组件:** 在QT中任何组件都可以用两种创建方式，我们可以通过使用`new`关键字动态创建按钮，也可以使用QT的图形化工具自动生成。

首先我们通过命令行的方式生成几个按钮，导入`QPushButton`包，然后定义如下代码，通过调用`connect()`可实现对特定按钮赋予特定的函数事件。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QPushButton>

// 设置函数,用于绑定事件
void Print()
{
    std::cout << "hello lyshark" << std::endl;
}

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 创建[退出]按钮
    QPushButton * btn = new QPushButton;   // 创建一个按钮
    // btn->show();                        // 用顶层方法弹出按钮
    btn->setParent(this);                  // 设置父窗体(将btn内嵌到主窗体中)
    btn->setText("退出");                   // 设置按钮text显示
    btn->move(100,200);                    // 移动按钮位置
    btn->resize(100,50);                   // 设置按钮大小
    btn->setEnabled(true);                 // 设置是否可被点击

    // 创建[触发信号]按钮
    QPushButton * btn2 = new QPushButton("触发信号",this);
    btn2->setParent(this);
    btn2->move(100,100);
    btn2->resize(100,50);

    // 设置主窗体常用属性
    this->resize(500,400);            // 重置窗口大小,调整主窗口大小
    this->setWindowTitle("我的窗体");  // 重置主窗体的名字
    this->setFixedSize(1024,300);     // 固定窗体大小(不让其修改)
    // this->showFullScreen();        // 设置窗体全屏显示

    // 设置主窗体特殊属性
    // setWindowFlags(Qt::FramelessWindowHint | Qt::WindowStaysOnTopHint); // 隐藏标题栏

    // 为按钮绑定事件 connect(信号的发送者,发送的信号,信号的接受者,处理的函数(槽函数))
    connect(btn,&QPushButton::clicked,this,&QWidget::close);

    // 将窗体中的 [触发信号] 按钮,连接到Print函数中.
    connect(btn2,&QPushButton::clicked,this,&Print);
}

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::~MainWindow()
{
    delete ui;
}
```

![](/image/1379525-20211122201559616-1267936846.png)

<br>

**LineEdit 单行输入组件:** 单行输入框`LineEdit()`组件用来输入一行文本内容，`GroupBox()`组件用来实现分组，`QString`类是String类的二次封装版，通过两者配合实现两个简单的数值转换器。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QString>
#include <QPushButton>

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 设置计算和编辑框不可修改
    ui->NumberSum->setEnabled(false);
    ui->lineEdit_hex->setEnabled(false);
    ui->lineEdit_bin->setEnabled(false);

    // 设置为密码输入
    ui->NumberSum->setEchoMode(QLineEdit::Password);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// By : LyShark
// https://www.cnblogs.com/lyshark
// 当点击计算按钮后完成计算
void MainWindow::on_pushButton_clicked()
{
    // 得到两个编辑框的数据
    QString string_total;
    QString Number_One = ui->numberA->text();
    QString Number_Two = ui->NumberB->text();

    if(Number_One.length() == 0 || Number_Two.length() == 0)
    {
        ui->label_3->setText("参数不能为空");
    }
    else
    {
        // 类型转换并赋值
        int number_int = Number_One.toInt();
        float number_float = Number_Two.toFloat();

        // 计算结果并放入到第三个编辑框中
        float total = number_int * number_float;

        string_total = string_total.sprintf("%.2f",total);
        ui->NumberSum->setText(string_total);
    }
}

// 当点击进制转换按钮后触发事件
void MainWindow::on_pushButton_2_clicked()
{
    QString str = ui->lineEdit->text();
    int value = str.toUInt();

    // 转十六进制
    str = str.setNum(value,16);     // 转为16进制
    str = str.toUpper();            // 变为大写
    ui->lineEdit_hex->setText(str); // 设置hex编辑框

    // 转二进制
    str = str.setNum(value,2);        // 第一种方式转换
    str = QString::number(value,2);   // 第二种方式转换
    ui->lineEdit_bin->setText(str);   // 设置bin编辑框
}
```

![](/image/1379525-20211122203359117-330296642.png)

如上我们学习总结了按钮组件与编辑框组件的使用，这两个组件组合起来可实现一个简单地页面登录验证界面，代码如下:
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QMessageBox>
#include <QByteArray>
#include <QSettings>
#include <QCryptographicHash>

QString m_user="admin";   // 初始化用户名
QString m_pswd="12345";   // 初始化密码
int m_tryCount=0;         // 试错次数

// 字符串MD5算法加密
QString MainWindow::encrypt(const QString &str)
{
    QByteArray btArray;
    btArray.append(str);                               // 加入原始字符串
    QCryptographicHash hash(QCryptographicHash::Md5);  // Md5加密算法

    hash.addData(btArray);                             // 添加数据到加密哈希值
    QByteArray resultArray =hash.result();             // 返回最终的哈希值
    QString md5 =resultArray.toHex();                  // 转换为16进制字符串
    return  md5;
}

// 读取用户名密码
void MainWindow::ReadString()
{
    QString organization="UserDataBase";           // 注册表
    QString appName="onley";                       // HKEY_CURRENT_USER/Software/UserDataBase/onley
    QSettings settings(organization,appName);      // 创建key-value

    bool saved=settings.value("saved",false).toBool();       // 读取 saved键的值
    m_user=settings.value("Username", "admin").toString();   // 读取 Username 键的值，缺省为admin
    QString defaultPSWD=encrypt("12345");                    // 缺省密码 12345 加密后的数据
    m_pswd=settings.value("PSWD",defaultPSWD).toString();    // 读取PSWD键的值

    if (saved)
    {
        ui->lineEdit_Username->setText(m_user);
    }
    ui->checkBox->setChecked(saved);
}

// 保存用户名密码设置
void MainWindow::WriteString()
{
    QSettings settings("UserDataBase","onley"); // 注册表键组
    settings.setValue("Username",m_user);       // 用户名
    settings.setValue("PSWD",m_pswd);           // 经过加密的密码
    settings.setValue("saved",ui->checkBox->isChecked());
}

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    setFixedSize(this->width(), this->height());             // 窗口不可调节
    ui->lineEdit_Password->setEchoMode(QLineEdit::Password); // 密码输入
    ReadString();
}

MainWindow::~MainWindow()
{
    delete ui;
}

// login
void MainWindow::on_pushButton_clicked()
{
    QString user=ui->lineEdit_Username->text().trimmed();//输入用户名
    QString pswd=ui->lineEdit_Password->text().trimmed(); //输入密码

    QString encrptPSWD=encrypt(pswd); //对输入密码进行加密

    if ((user==m_user)&&(encrptPSWD==m_pswd)) //如果用户名和密码正确
    {
        WriteString();
        QMessageBox::critical(this,"成功","已登录");
    }
    else
    {
        m_tryCount++; //错误次数
        if (m_tryCount>3)
        {
            QMessageBox::critical(this, "错误", "输入错误次数太多，强行退出");
            this->close();
        }
        else
        {
            QMessageBox::warning(this, "错误提示", "用户名或密码错误");
        }

    }
}
```

![](/image/1379525-20211122203654519-1319050419.png)

<br>

**SpinBox 数值组件:** 该控件主要用于整数或浮点数的计数显示，与普通的LineEdit不同，该组件可以在前后增加特殊符号并提供了上下幅度的调整按钮，灵活性更强。

该组件有两个版本，`SpinBox()`用于显示整数与单精度浮点数，`DoubleSpinBox()`则是双精度浮点数，SpinBox有两个特殊参数，`prefix`参数是在前方加入特殊符号，而`suffix`则是在后方加入特殊符号。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QString>
#include <QPushButton>

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->doubleSpinBox->setEnabled(false);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// By : LyShark
// https://www.cnblogs.com/lyshark
// 实现精度计算功能
void MainWindow::on_pushButton_3_clicked()
{
    int x = ui->spinBox->value();
    int y = ui->spinBox_2->value();

    double total = x+y;
    ui->doubleSpinBox->setValue(total);   // 设置SpinBox数值(设置时无需转换)

    QString label_value = ui->label_10->text();  // 获取字符串
    ui->label_10->setNum(total);                 // 设置label标签为数字
}
```

![](/image/1379525-20211122205754638-1785234368.png)

我们继续在SpinBox的基础上改进，如上代码中每次都需要点击计算按钮才能出结果，此时我们需求是实现当`SpinBox`中的参数发生变化时自定的完成计算，这里就需要用到信号和槽了，当SpinBox被修改后，自动触发计算信号实现计算。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QString>
#include <QPushButton>

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->doubleSpinBox->setEnabled(false);

    // 将数量和单价两个SpinBox的valueChanged()信号与on_pushButton_clicked()槽关联
    // 只要spinBox中的内容发生变化,则立即触发按钮完成计算
    QObject::connect(ui->spinBox,SIGNAL(valueChanged(int)),this,SLOT(on_pushButton_clicked()));
    QObject::connect(ui->spinBox_2,SIGNAL(valueChanged(int)),this,SLOT(on_pushButton_clicked()));
    QObject::connect(ui->doubleSpinBox,SIGNAL(valueChanged(double)),this,SLOT(on_pushButton_clicked()));
}

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::~MainWindow()
{
    delete ui;
}

// 实现精度计算功能
void MainWindow::on_pushButton_clicked()
{
    int x = ui->spinBox->value();
    int y = ui->spinBox_2->value();

    double total = x+y;
    ui->doubleSpinBox->setValue(total);   // 设置SpinBox数值(设置时无需转换)

    QString label_value = ui->label_10->text();  // 获取字符串
    ui->label_10->setNum(total);                 // 设置label标签为数字
}
```

![](/image/1379525-20211122205835558-327074342.png)

<br>

**HorizontalSlider 滑块条组件:** 根据上面的SpinBox信号与槽函数的绑定，我们还可以将其绑定到滑块条组件上，如下代码实现了，当用户改变滑块条时，右侧的`textEdit`的颜色也会发生相应的改变。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QString>
#include <QPushButton>

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // ----------------------------------------------------------------------------------
    // 将 SliderGreen，SliderBlue，SliderAlpha 与第一个滑块条 SliderRead 关联起来
    // 实现效果为，当其他三个选择条数值改变时，同样会触发on_SliderRed_valueChanged槽函数
    QObject::connect(ui->SliderRed,SIGNAL(valueChanged(int)),this,SLOT(on_SliderRed_valueChanged(int)));
    QObject::connect(ui->SliderGreen,SIGNAL(valueChanged(int)),this,SLOT(on_SliderRed_valueChanged(int)));
    QObject::connect(ui->SliderBlue,SIGNAL(valueChanged(int)),this,SLOT(on_SliderRed_valueChanged(int)));
    QObject::connect(ui->SliderAlpha,SIGNAL(valueChanged(int)),this,SLOT(on_SliderRed_valueChanged(int)));
}

MainWindow::~MainWindow()
{
    delete ui;
}

// By : LyShark
// https://www.cnblogs.com/lyshark
// 当拖动SliderRed滑块条时设置TextEdit底色
void MainWindow::on_SliderRed_valueChanged(int value)
{
    Q_UNUSED(value);
    QColor  color;
    int R=ui->SliderRed->value();      // 读取SliderRed的当前值
    int G=ui->SliderGreen->value();    // 读取 SliderGreen 的当前值
    int B=ui->SliderBlue->value();     // 读取 SliderBlue 的当前值
    int alpha=ui->SliderAlpha->value();// 读取 SliderAlpha 的当前值
    color.setRgb(R,G,B,alpha);         // 使用QColor的setRgb()函数获得颜色

    QPalette pal=ui->textEdit->palette(); // 获取textEdit原有的 palette
    pal.setColor(QPalette::Base,color);   // 设置palette的基色（即背景色）
    ui->textEdit->setPalette(pal);        // 设置为textEdit的palette,改变textEdit的底色
}
```

![](/image/1379525-20211122210628691-1498096615.png)

<br>

**数码表与LCD屏幕:** 这是两个比较有趣的组件，如下布局中圆形的是`dial`组件，其右侧则是一个`LCD Number`组件，两者可以灵活的结合在一起使用，当拨动齿轮时自动影响LCD数码屏幕的显示。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QString>
#include <QPushButton>
#include <QRadioButton>

MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::~MainWindow()
{
    delete ui;
}


// 当圆形选择框数值改变时设置数码表显示
void MainWindow::on_dial_valueChanged(int value)
{
   ui->LCDDisplay->display(value);
}


// 选中时设置为十进制显示
void MainWindow::on_radioBtnDec_clicked()
{
    ui->LCDDisplay->setDigitCount(3);   // 设置位数
    ui->LCDDisplay->setDecMode();       // 十进制
}


// 选中设置为二进制显示
void MainWindow::on_radioBtnBin_clicked()
{
    ui->LCDDisplay->setDigitCount(8);
    ui->LCDDisplay->setBinMode();
}


// 选中设置为八进制显示
void MainWindow::on_radioBtnOct_clicked()
{
    ui->LCDDisplay->setDigitCount(5);
    ui->LCDDisplay->setOctMode();
}


// 选中设置为十六进制显示
void MainWindow::on_radioBtnHex_clicked()
{
    ui->LCDDisplay->setDigitCount(3);
    ui->LCDDisplay->setHexMode();
}
```

![](/image/1379525-20211122211358335-54790051.png)

<br>

**CheckBox 多选框:** 多选框`CheckBox`组件也是最常用的组件，多选框支持三态选择，选中半选中和未选中状态。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->checkBox->setTristate();       // 启用三态选择框
    ui->checkBox->setEnabled(true);    // 设置为可选状态
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 三态选择框状态
void MainWindow::on_checkBox_stateChanged(int state)
{
    // 选中状态
    if (state == Qt::Checked)
    {
       ui->checkBox->setText("选中");
    }
    // 半选状态
    else if(state == Qt::PartiallyChecked)
    {
       ui->checkBox->setText("半选");
    }
    // 未选中
    else
    {
       ui->checkBox->setText("未选中");
    }
}

// 设置取消选中
void MainWindow::on_pushButton_clicked()
{
    int check = ui->checkBox->isCheckable();
    if(check == 1)
    {
        ui->checkBox->setChecked(false);
    }
}

// 关联式多选框
void MainWindow::on_checkBox_master_stateChanged(int state)
{
    // 选中所有子框
    if(state == Qt::Checked)
    {
        ui->checkBox_sub_a->setChecked(true);
        ui->checkBox_sub_b->setChecked(true);
    }
    // 取消子框全选状态
    if(state == Qt::Unchecked)
    {
        ui->checkBox_sub_a->setChecked(false);
        ui->checkBox_sub_b->setChecked(false);
    }
}
```

![](/image/1379525-20211122211458297-336263192.png)

<br>

**ComBox 下拉框组件:** 该组件提供了下拉列表供用户选择，ComBox组件除了可以显示下拉列表外，每个项还可以关联一个QVariant类型的变量用于存储不可见数据。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <iostream>
#include <QList>
#include <QMap>

// 定义为全局变量
QMap<QString,int> City_Zone;
QMap<QString,QList <QString>> map;
QList<QString> tmp;

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    // -----------------------------------------------------------------
    // 循环方式添加元素
    ui->comboBox_main->clear();
    QIcon icon;
    icon.addFile(":/image/1.ico");
    for(int x=0;x<10;x++)
    {
        ui->comboBox_main->addItem(icon,QString::asprintf("元素_%d",x));
    }

    // -----------------------------------------------------------------
    // 批量添加combox元素
    ui->comboBox_main->clear();
    QStringList str;
    str << "北京" << "上海" << "广州";

    ui->comboBox_main->addItems(str);
    ui->comboBox_main->setItemIcon(0,QIcon(":/image/1.ico"));
    ui->comboBox_main->setItemIcon(1,QIcon(":/image/2.ico"));
    ui->comboBox_main->setItemIcon(2,QIcon(":/image/3.ico"));

    // -----------------------------------------------------------------
    // 实现combox联动效果
    ui->comboBox_main->clear();
    City_Zone.insert("请选择",0);
    City_Zone.insert("北京",1);
    City_Zone.insert("上海",2);
    City_Zone.insert("广州",3);

    // 循环填充一级菜单
    ui->comboBox_main->clear();
    foreach(const QString &str,City_Zone.keys())
    {
        ui->comboBox_main->addItem(QIcon(":/image/1.ico"),str,City_Zone.value(str));
    }

    // -----------------------------------------------------------------
    // 插入二级菜单
    tmp.clear();
    tmp << "大兴区" << "昌平区" << "东城区";
    map["北京"] = tmp;

    tmp.clear();
    tmp << "黄浦区" << "徐汇区" << "长宁区" << "杨浦区";
    map["上海"] = tmp;

    tmp.clear();
    tmp << "荔湾区" << "越秀区";
    map["广州"] = tmp;

    // 设置默认选择第三个
    ui->comboBox_main->setCurrentIndex(3);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 获取当前选中的两级菜单内容
void MainWindow::on_pushButton_clicked()
{
    QString one = ui->comboBox_main->currentText();
    QString two = ui->comboBox_submain->currentText();
    std::cout << one.toStdString().data() << " | " << two.toStdString().data() << std::endl;
}

// 当主ComBox被选择时,自动的填充第2个ComBox中的数据.
void MainWindow::on_comboBox_main_currentTextChanged(const QString &arg1)
{
    ui->comboBox_submain->clear();
    QList<QString> qtmp;

    qtmp = map.value(arg1);
    for(int x=0;x<qtmp.count();x++)
    {
        ui->comboBox_submain->addItem(QIcon(":/image/2.ico"),qtmp[x]);
    }
}
```

![](/image/1379525-20211122212158914-665067253.png)

<br>

**ProgressBar 进度条与定时器:** 进度条`ProgressBar`组件通常会结合`QTimer`定时器组件共同使用，首先我们需要设置一个时钟周期，定时器每经过一定的时间周期则执行对变量或进度条的递增操作，由此实现进度条动态输出效果。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QTimer>

QTimer *my_timer;

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 初始化进度条数值
    ui->progressBar->setValue(0);
    ui->progressBar_2->setValue(100);


    // 声明定时器
    my_timer = new QTimer(this);

    // 绑定一个匿名函数
    connect(my_timer,&QTimer::timeout,[=]{
        static int x = 0;

        // 判断是否到达了进度条的最大值
        if(x != 100)
        {
            x++;
            ui->progressBar->setValue(x);
            ui->progressBar_2->setValue(int(100-x));
        }
        else
        {
            x=0;
            my_timer->stop();
        }
    });
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 启动定时器,并设置周期为100毫秒
void MainWindow::on_pushButton_clicked()
{
    if(my_timer->isActive() == false)
    {
        my_timer->start(100);
    }
}

// 停止定时器
void MainWindow::on_pushButton_2_clicked()
{
    if(my_timer->isActive() == true)
    {
        my_timer->stop();
    }
}

// 将进度条置空
void MainWindow::on_pushButton_3_clicked()
{
    ui->progressBar->setValue(0);
    ui->progressBar_2->setValue(100);
}
```

![](/image/1379525-20211122212400182-81594932.png)

<br>

**DateTime 日期与时间组件:** 时间组件中包括了可以显示时间的`QTime`显示日期的`QDate`以及可同时显示时间与日期的`QDateTime`这三种组件，三种组件的使用上几乎一致，如下代码是开发中最常用的总结。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QDateTime>

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

// By : LyShark

// https://www.cnblogs.com/lyshark
MainWindow::~MainWindow()
{
    delete ui;
}

// 获取当前日期时间,并初始化到组件中
void MainWindow::on_pushButton_clicked()
{
     QDateTime curDateTime = QDateTime::currentDateTime();

     ui->timeEdit->setTime(curDateTime.time());
     ui->dateEdit->setDate(curDateTime.date());
     ui->dateTimeEdit->setDateTime(curDateTime);

     ui->lineEdit->setText(curDateTime.toString("yyyy-MM-dd hh:mm:ss"));
}

// 将字符串时间日期转换到时间日期组件中
void MainWindow::on_pushButton_2_clicked()
{
    QString str = ui->lineEdit_2->text();
    str = str.trimmed();
    if(!str.isEmpty())
    {
        QDateTime datetime = QDateTime::fromString(str,"yyyy-MM-dd hh:mm:ss");
        ui->dateTimeEdit_string_to_datetime->setDateTime(datetime);
    }
}
```

![](/image/1379525-20211122212537132-1384048214.png)

<br>

**PlainTextEdit 多行文本框:** 多行文本编辑器，用于显示和编辑多行简单文本，如下代码左侧`PlainTextEdit`中输入数据(每行换行)点击按钮后自动将左侧数据放入右侧的`listView`组件中。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QLabel>
#include <QTextBlock>
#include <iostream>
#include <QStringListModel>

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 状态栏的创建
    QStatusBar * stBar = statusBar();
    setStatusBar(stBar);

    QLabel * label = new QLabel("左侧提示信息",this);
    stBar->addWidget(label);

    QLabel * label2 = new QLabel("右侧提示信息",this);
    stBar->addPermanentWidget(label2);
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 点击按钮实现将 plainTextEdit 里面的数据逐行放入到右侧
void MainWindow::on_pushButton_clicked()
{
    QTextDocument* doc = ui->plainTextEdit->document ();  // 文本对象
    int count = doc->blockCount();                        // 定义回车为分隔符

    // 定义data,model用于存储每个文本
    QStringList data;
    QStringListModel *model;

    for(int x=0;x< count;x++)
    {
        QTextBlock textLine = doc->findBlockByNumber(x); // 每次取出plainTextEdit中的一行
        QString str = textLine.text();
        data << str;                                     // 放入链表中
    }

    // 显示到ListView中
    model = new QStringListModel(data);
    ui->listView->setModel(model);
}
```

![](/image/1379525-20211122212725089-1084703589.png)

<br>

**RadioButton 单选框分组:** 单选框是最常用的组件，在一个界面中可以有多种单选框，每种单选框都会对应一个问题，此实我们需要使用`ButtonGroup`组件对单选框进行分组，并通过信号和槽函数相互绑定，从而实现对用户的多种选择进行判断。
```CPP
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <QButtonGroup>
#include <iostream>

QButtonGroup *group_sex;
QButtonGroup *group_hobby;

// By : LyShark
// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 将不同的RadioButton放入不同的ButtonGroup组中
    group_sex = new QButtonGroup(this);
    group_sex->addButton(ui->radioButton_male,0);
    group_sex->addButton(ui->radioButton_female,1);
    group_sex->addButton(ui->radioButton_unknown,2);
    ui->radioButton_unknown->setChecked(true);

    group_hobby = new QButtonGroup(this);
    group_hobby->addButton(ui->radioButton_eat,3);
    group_hobby->addButton(ui->radioButton_drink,4);
    group_hobby->addButton(ui->radioButton_sleep,5);
    ui->radioButton_eat->setChecked(true);

    // 绑定信号和槽
    connect(ui->radioButton_male,SIGNAL(clicked(bool)),this,SLOT(MySlots()));
    connect(ui->radioButton_female,SIGNAL(clicked(bool)),this,SLOT(MySlots()));
    connect(ui->radioButton_unknown,SIGNAL(clicked(bool)),this,SLOT(MySlots()));
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 手动创建一个槽函数,此处的槽函数声明需要加入到头文件private slots中
void MainWindow::MySlots()
{
    switch(group_sex->checkedId())
    {
    case 0:
        std::cout << "male" << std::endl;
        break;
    case 1:
        std::cout << "female" << std::endl;
        break;
    case 2:
        std::cout << "unknown" << std::endl;
        break;
    }
}
```

![](/image/1379525-20211122212812977-917463213.png)
