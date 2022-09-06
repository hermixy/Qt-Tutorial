QtCharts 组件是QT中提供图表绘制的模块，该模块可以方便的绘制常规图形，Qtcharts 组件基于`GraphicsView`模式实现，其核心是`QChartView`和`QChart`的二次封装版。

在使用绘图模块时需要在pro文件中包含`QT += charts`来引入绘图类库。

![](/image/1379525-20211119143048200-1663845826.png)

然后还需在头文件中定义`QT_CHARTS_USE_NAMESPACE`宏，这样才可以正常的使用绘图功能。

![](/image/1379525-20211119143121462-1243071534.png)

一般情况下我们会在`mainwindows.h`头文件中增加如下代码段。
```C
#include <QMainWindow>
#include <QtCharts>
QT_CHARTS_USE_NAMESPACE

// 解决MSVC编译时，界面汉字乱码的问题
#if _MSC_VER >= 1600
#pragma execution_character_set("utf-8")
#endif
```
由于QT中不存在单独的绘图画布，因此在绘图前我们需要在窗体中放入一个`graphicsView`组件。

![](/image/1379525-20211119143152756-259992224.png)

并在该组件上右键将其提升为`QChartView`

![](/image/1379525-20211119143329404-1120423394.png)

输入需要提升的组件名称，即可将该组件提升为全局绘图组件。

![](/image/1379525-20211119143709118-392120401.png)

<br>

**绘制折线图:** 折线图的使用非常广泛，如下代码我们首先使用`InitChart()`将画布初始化，接着调用`SetData()`实现在画布中填充数据，完整代码如下。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

// 初始化Chart图表
void MainWindow::InitChart()
{
    // 创建图表的各个部件
    QChart *chart = new QChart();
    chart->setTitle("系统性能统计图");

    // 将Chart添加到ChartView
    ui->graphicsView->setChart(chart);
    // this->setCentralWidget( ui->graphicsView);
    ui->graphicsView->setRenderHint(QPainter::Antialiasing);

    // 设置图表主题色
    ui->graphicsView->chart()->setTheme(QChart::ChartTheme(0));

    // 创建曲线序列
    QLineSeries *series0 = new QLineSeries();
    QLineSeries *series1 = new QLineSeries();

    series0->setName("一分钟负载");
    series1->setName("五分钟负载");

    // 序列添加到图表
    chart->addSeries(series0);
    chart->addSeries(series1);

    // 其他附加参数
    series0->setPointsVisible(false);       // 设置数据点可见
    series1->setPointLabelsVisible(false);  // 设置数据点数值可见

    // 创建坐标轴
    QValueAxis *axisX = new QValueAxis;    // X轴
    axisX->setRange(1, 100);               // 设置坐标轴范围
    axisX->setTitleText("X轴标题");         // 标题
    axisX->setLabelFormat("%d %");         // 设置x轴格式
    axisX->setTickCount(3);               // 设置刻度
    axisX->setMinorTickCount(3);

    QValueAxis *axisY = new QValueAxis;    // Y轴
    axisY->setRange(0, 100);               // Y轴范围(-1 - 20)
    axisY->setTitleText("Y轴标题");         // 标题

    // 设置X于Y轴数据集
    chart->setAxisX(axisX, series0);   // 为序列设置坐标轴
    chart->setAxisY(axisY, series0);

    chart->setAxisX(axisX, series1);   // 为序列设置坐标轴
    chart->setAxisY(axisY, series1);

    // 图例被点击后触发
    foreach (QLegendMarker* marker, chart->legend()->markers())
    {
       QObject::disconnect(marker, SIGNAL(clicked()), this, SLOT(on_LegendMarkerClicked()));
       QObject::connect(marker, SIGNAL(clicked()), this, SLOT(on_LegendMarkerClicked()));
    }
}

// 为序列生成数据
void MainWindow::SetData()
{
    // 获取指针
    QLineSeries *series0=(QLineSeries *)ui->graphicsView->chart()->series().at(0);
    QLineSeries *series1=(QLineSeries *)ui->graphicsView->chart()->series().at(1);

    // 清空图例
    series0->clear();
    series1->clear();

    // 赋予数据
    qreal t=0,intv=1;
    for(int i=1;i<100;i++)
    {
       series0->append(t,i);       // 设置轴粒度以及数据
       series1->append(t,i+10);    // 此处用随机数替代
       t+=intv;                    // X轴粒度
    }
}

// 将添加的widget控件件提升为QChartView类
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    InitChart();
    SetData();
}

MainWindow::~MainWindow()
{
    delete ui;
}

// 图例点击后显示与隐藏线条
void MainWindow::on_LegendMarkerClicked()
{
    QLegendMarker* marker = qobject_cast<QLegendMarker*> (sender());

    switch (marker->type())
    {
        case QLegendMarker::LegendMarkerTypeXY:
        {
            marker->series()->setVisible(!marker->series()->isVisible());
            marker->setVisible(true);
            qreal alpha = 1.0;
            if (!marker->series()->isVisible())
                alpha = 0.5;

            QColor color;
            QBrush brush = marker->labelBrush();
            color = brush.color();
            color.setAlphaF(alpha);
            brush.setColor(color);
            marker->setLabelBrush(brush);

            brush = marker->brush();
            color = brush.color();
            color.setAlphaF(alpha);
            brush.setColor(color);
            marker->setBrush(brush);

            QPen pen = marker->pen();
            color = pen.color();
            color.setAlphaF(alpha);
            pen.setColor(color);
            marker->setPen(pen);
            break;
        }
        default:
            break;
    }
}
```

效果如下所示：

![](/image/1379525-20211119143958962-1329280004.png)

<br>

**绘制饼状图:** 饼状图用于统计数据的集的占用百分比，其绘制方式与折线图基本一致，代码如下。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

// 饼状图A
void MainWindow::printA()
{
    // 构造数据 [已用CPU 60%] [剩余CPU 40%]
    QPieSlice *slice_1 = new QPieSlice(QStringLiteral("已使用"), 0.6, this);
    slice_1->setLabelVisible(true);

    QPieSlice *slice_2 = new QPieSlice(QStringLiteral("可用"), 0.4, this);
    slice_2->setLabelVisible(true);

    // 将两个饼状分区加入series
    QPieSeries *series = new QPieSeries(this);
    series->append(slice_1);
    series->append(slice_2);

    // 创建Chart画布
    QChart *chart = new QChart();
    chart->addSeries(series);
    chart->setAnimationOptions(QChart::AllAnimations); // 设置显示时的动画效果
    chart->setTitle("系统CPU利用率");

    // 将参数设置到画布
    ui->graphicsView->setChart(chart);
    ui->graphicsView->setRenderHint(QPainter::Antialiasing);
    ui->graphicsView->chart()->setTheme(QChart::ChartTheme(0));
}

// 饼状图B
void MainWindow::printB()
{
    // 构造数据 [C盘 20%] [D盘 30%] [E盘 50%]
    QPieSlice *slice_c = new QPieSlice(QStringLiteral("C盘"), 0.2, this);
    slice_c->setLabelVisible(true);

    QPieSlice *slice_d = new QPieSlice(QStringLiteral("D盘"), 0.3, this);
    slice_d->setLabelVisible(true);

    QPieSlice *slice_e = new QPieSlice(QStringLiteral("E盘"),0.5,this);
    slice_e->setLabelVisible(true);

    // 将两个饼状分区加入series
    QPieSeries *series = new QPieSeries(this);
    series->append(slice_c);
    series->append(slice_d);
    series->append(slice_e);

    // 创建Chart画布
    QChart *chart = new QChart();
    chart->addSeries(series);
    chart->setAnimationOptions(QChart::AllAnimations); // 设置显示时的动画效果
    chart->setTitle("系统磁盘信息");

    // 将参数设置到画布
    ui->graphicsView_2->setChart(chart);
    ui->graphicsView_2->setRenderHint(QPainter::Antialiasing);
    ui->graphicsView_2->chart()->setTheme(QChart::ChartTheme(3));   // 设置不同的主题
}

// 将添加的widget控件件提升为QChartView类
MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    printA();
    printB();
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

效果如下所示：

![](/image/1379525-20211119144552766-1237700524.png)

<br>

**绘制柱状图:** 柱状图可用于一次展示多个用户数据，大体是使用上与折线图大体一致，其代码如下:
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

MainWindow::MainWindow(QWidget *parent) :QMainWindow(parent),ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 创建人名
    QBarSet *set0 = new QBarSet("张三");
    QBarSet *set1 = new QBarSet("李四");
    QBarSet *set2 = new QBarSet("王五");
    QBarSet *set3 = new QBarSet("苏三");
    QBarSet *set4 = new QBarSet("刘麻子");

    // 分别为不同人添加bu不同数据集
    *set0 << 1 << 2 << 8 << 4 << 6 << 6;
    *set1 << 5 << 2 << 5 << 4 << 5 << 3;
    *set2 << 5 << 5 << 8 << 15 << 9 << 5;
    *set3 << 8 << 6 << 7 << 5 << 4 << 5;
    *set4 << 4 << 7 << 5 << 3 << 3 << 2;

    // 将数据集关联到series中
    QBarSeries *series = new QBarSeries();
    series->append(set0);
    series->append(set1);
    series->append(set2);
    series->append(set3);
    series->append(set4);

    // 增加顶部提示
    QChart *chart = new QChart();
    chart->addSeries(series);
    chart->setTitle("当前人数统计");
    chart->setAnimationOptions(QChart::SeriesAnimations);

    // 创建X轴底部提示
    QStringList categories;
    categories << "周一" << "周二" << "周三" << "周四" << "周五" << "周六";

    QBarCategoryAxis *axis = new QBarCategoryAxis();
    axis->append(categories);
    chart->createDefaultAxes();
    chart->setAxisX(axis, series);

    chart->legend()->setVisible(true);
    chart->legend()->setAlignment(Qt::AlignBottom);

    // 将参数设置到画布
    ui->graphicsView->setChart(chart);
    ui->graphicsView->setRenderHint(QPainter::Antialiasing);
    ui->graphicsView->chart()->setTheme(QChart::ChartTheme(0));
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

效果如下所示：

![](/image/1379525-20211119144736751-1626463244.png)
