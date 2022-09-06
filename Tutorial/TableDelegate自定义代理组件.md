TableDelegate 自定义代理组件的主要作用是对原有表格进行调整，例如默认情况下Table中的缺省代理就是一个编辑框，我们只能够在编辑框内输入数据，而有时我们想选择数据而不是输入，此时就需要重写编辑框实现选择的效果，代理组件常用于个性化定制Table表格中的字段类型。

代理类的作用是用来实现重写的，例如我们的`TableView`中默认是可编辑的，这个可编辑的组件是QT默认为我们重写了`QLineEdit`组件，也可理解为将组件嵌入到了表格中，实现了对表格的编辑功能。

在自定义代理中`QAbstractItemDelegate`是所有代理类的抽象基类，我们继承任何组件时都必须要包括如下4个函数:

 - CreateEditor() 用于创建编辑模型数据的组件，例如(QSpinBox组件)
 - SetEditorData() 从数据模型获取数据，以供Widget组件进行编辑
 - SetModelData() 将Widget组件上的数据更新到数据模型
 - UpdateEditorGeometry() 给Widget组件设置一个合适的大小

此处我们分别重写三个代理接口，其中两个`ComBox`组件用于选择婚否，`SpinBox`组件用于调节数值范围，先来定义三个重写部件。

先来实现一个代理，代理到`Spin`组件上，首先需要在项目上右键
 - 选择addnew -> C++Class 输入自定义类名称`QWintSpinDelegate`，然后基类继承`QStyledItemDelegate/QMainWindow`，然后下一步结束向导。

![](https://img2020.cnblogs.com/blog/1379525/202112/1379525-20211203141715362-1279837663.png)

重写接口`spindelegate.cpp`代码如下.
```C
#include "spindelegate.h"
#include <QSpinBox>

QWIntSpinDelegate::QWIntSpinDelegate(QObject *parent):QStyledItemDelegate(parent)
{
}

// https://www.cnblogs.com/lyshark
QWidget *QWIntSpinDelegate::createEditor(QWidget *parent,const QStyleOptionViewItem &option, const QModelIndex &index) const
{
//创建代理编辑组件
    Q_UNUSED(option);
    Q_UNUSED(index);

    QSpinBox *editor = new QSpinBox(parent); //创建一个QSpinBox
    editor->setFrame(false); //设置为无边框
    editor->setMinimum(0);
    editor->setMaximum(10000);
    return editor;  //返回此编辑器
}

void QWIntSpinDelegate::setEditorData(QWidget *editor,const QModelIndex &index) const
{
//从数据模型获取数据，显示到代理组件中
//获取数据模型的模型索引指向的单元的数据
    int value = index.model()->data(index, Qt::EditRole).toInt();

    QSpinBox *spinBox = static_cast<QSpinBox*>(editor);  //强制类型转换
    spinBox->setValue(value); //设置编辑器的数值
}

void QWIntSpinDelegate::setModelData(QWidget *editor, QAbstractItemModel *model, const QModelIndex &index) const
{
//将代理组件的数据，保存到数据模型中
    QSpinBox *spinBox = static_cast<QSpinBox*>(editor); //强制类型转换
    spinBox->interpretText(); //解释数据，如果数据被修改后，就触发信号
    int value = spinBox->value(); //获取spinBox的值

    model->setData(index, value, Qt::EditRole); //更新到数据模型
}

void QWIntSpinDelegate::updateEditorGeometry(QWidget *editor, const QStyleOptionViewItem &option, const QModelIndex &index) const
{
//设置组件大小
    Q_UNUSED(index);
    editor->setGeometry(option.rect);
}
```
重写接口`floatspindelegate.cpp`代码如下.
```C
#include "floatspindelegate.h"
#include <QDoubleSpinBox>

QWFloatSpinDelegate::QWFloatSpinDelegate(QObject *parent):QStyledItemDelegate(parent)
{
}

QWidget *QWFloatSpinDelegate::createEditor(QWidget *parent,
   const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    Q_UNUSED(option);
    Q_UNUSED(index);

    QDoubleSpinBox *editor = new QDoubleSpinBox(parent);
    editor->setFrame(false);
    editor->setMinimum(0);
    editor->setDecimals(2);
    editor->setMaximum(10000);

    return editor;
}

void QWFloatSpinDelegate::setEditorData(QWidget *editor,
                      const QModelIndex &index) const
{
    float value = index.model()->data(index, Qt::EditRole).toFloat();
    QDoubleSpinBox *spinBox = static_cast<QDoubleSpinBox*>(editor);
    spinBox->setValue(value);
}

// https://www.cnblogs.com/lyshark
void QWFloatSpinDelegate::setModelData(QWidget *editor, QAbstractItemModel *model, const QModelIndex &index) const
{
    QDoubleSpinBox *spinBox = static_cast<QDoubleSpinBox*>(editor);
    spinBox->interpretText();
    float value = spinBox->value();
    QString str=QString::asprintf("%.2f",value);

    model->setData(index, str, Qt::EditRole);
}

void QWFloatSpinDelegate::updateEditorGeometry(QWidget *editor, const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    editor->setGeometry(option.rect);
}
```
重写接口`comboxdelegate.cpp`代码如下.
```C
#include "comboxdelegate.h"
#include <QComboBox>

QWComboBoxDelegate::QWComboBoxDelegate(QObject *parent):QItemDelegate(parent)
{
}

QWidget *QWComboBoxDelegate::createEditor(QWidget *parent,const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    QComboBox *editor = new QComboBox(parent);

    editor->addItem("已婚");
    editor->addItem("未婚");
    editor->addItem("单身");

    return editor;
}

// https://www.cnblogs.com/lyshark
void QWComboBoxDelegate::setEditorData(QWidget *editor, const QModelIndex &index) const
{
    QString str = index.model()->data(index, Qt::EditRole).toString();

    QComboBox *comboBox = static_cast<QComboBox*>(editor);
    comboBox->setCurrentText(str);
}

void QWComboBoxDelegate::setModelData(QWidget *editor, QAbstractItemModel *model, const QModelIndex &index) const
{
    QComboBox *comboBox = static_cast<QComboBox*>(editor);
    QString str = comboBox->currentText();
    model->setData(index, str, Qt::EditRole);
}

void QWComboBoxDelegate::updateEditorGeometry(QWidget *editor,const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    editor->setGeometry(option.rect);
}
```
将部件导入到`mainwindow.cpp`中，并将其通过`ui->tableView->setItemDelegateForColumn(0,&intSpinDelegate);`关联部件到指定的table下标索引上面。
```C
#include "mainwindow.h"
#include "ui_mainwindow.h"

// https://www.cnblogs.com/lyshark
MainWindow::MainWindow(QWidget *parent): QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    // 初始化模型数据
    model = new QStandardItemModel(4,6,this);      // 初始化4行,每行六列
    selection = new QItemSelectionModel(model);    // 关联模型

    ui->tableView->setModel(model);
    ui->tableView->setSelectionModel(selection);

    // 添加表头
    QStringList HeaderList;
    HeaderList << "序号" << "姓名" << "年龄" << "性别" << "婚否" << "薪资";
    model->setHorizontalHeaderLabels(HeaderList);

    // 批量添加数据
    QStringList DataList[3];
    QStandardItem *Item;

    DataList[0] << "1001" << "admin" << "24" << "男" << "已婚" << "4235.43";
    DataList[1] << "1002" << "lyshark" << "23" << "男" << "未婚" << "10000.21";
    DataList[2] << "1003" << "lucy" << "37" << "女" << "单身" << "8900.23";

    int Array_Length = DataList->length();                          // 获取每个数组中元素数
    int Array_Count = sizeof(DataList) / sizeof(DataList[0]);       // 获取数组个数

    for(int x=0; x<Array_Count; x++)
    {
        for(int y=0; y<Array_Length; y++)
        {
            // std::cout << DataList[x][y].toStdString().data() << std::endl;
            Item = new QStandardItem(DataList[x][y]);
            model->setItem(x,y,Item);
        }
    }

    // 为各列设置自定义代理组件
    // 0，4，5 代表第几列 后面的函数则是使用哪个代理类的意思
    ui->tableView->setItemDelegateForColumn(0,&intSpinDelegate);
    ui->tableView->setItemDelegateForColumn(4,&comboBoxDelegate);
    ui->tableView->setItemDelegateForColumn(5,&floatSpinDelegate);

}

MainWindow::~MainWindow()
{
    delete ui;
}
```

代理部件关联后，再次运行程序，会发现原来的`TableWidget`组件中的编辑框已经替换为了选择框等组件:

![](/image/1379525-20211201142450302-1262041489.gif)
