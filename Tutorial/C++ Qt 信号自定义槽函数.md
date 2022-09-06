Qt中实现自定义信号与槽函数，信号用于发送并触发槽函数，槽函数则是具体的功能实现，如下我们以老师学生为例子简单学习一下信号与槽函数的使用方法。

**使用无参数信号与槽：** 首先定义一个teacher类，该类中用于发送一个信号，其次student类，定义用于接收该信号的槽函数，最后在widget中使用emit触发信号，当老师说下课时，学生请客吃饭。

teacher.h 中只需要定义信号。定义一个 void hungry(); 信号。
```C
#ifndef TEACHER_H
#define TEACHER_H

#include <QObject>

class Teacher : public QObject
{
    Q_OBJECT
public:
    explicit Teacher(QObject *parent = nullptr);

signals:
    // 定义一个信号,信号必须为void类型,且信号不能实现
    void hungry();
};

#endif // TEACHER_H
```

student中需要定义槽声明，并实现槽。
student.h
```C
#ifndef STUDENT_H
#define STUDENT_H

#include <QObject>

class Student : public QObject
{
    Q_OBJECT
public:
    explicit Student(QObject *parent = nullptr);

signals:

public slots:
    // 自定义槽函数
    // 槽函数必须定义且必须要声明才可以使用
    void treat();
};

#endif // STUDENT_H
```
student.cpp
```C
#include "student.h"
#include <QDebug>

Student::Student(QObject *parent) : QObject(parent)
{

}

// 槽函数的实现过程如下
void Student::treat()
{
    qDebug() << "请老师吃饭";
}
```

Widget.h定义信号发送函数，与类
```C
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include "student.h"
#include "teacher.h"

class Widget : public QWidget
{
    Q_OBJECT

public:
    Widget(QWidget *parent = nullptr);
    ~Widget();

    // 定义学生与老师类
    Teacher *zt;
    Student *st;

    // 定义信号发送函数
    void classIsOver();

};
#endif // WIDGET_H
```
Widget.cpp 具体实现
```C
#include "widget.h"

Widget::Widget(QWidget *parent): QWidget(parent)
{
    zt = new Teacher(this);
    st = new Student(this);

    // zt向st发送信号，信号是&Teacher::hungry 处理槽函数是 &Student::treat
    connect(zt,&Teacher::hungry,st,&Student::treat);

    classIsOver();
}

Widget::~Widget()
{
}

// 触发信号
void Widget::classIsOver()
{
    emit zt->hungry();
}
```

**使用有参信号传递:** 只需要再无参基础上改进

widget.cpp
```C
#include "widget.h"

Widget::Widget(QWidget *parent): QWidget(parent)
{
    zt = new Teacher(this);
    st = new Student(this);

    void(Teacher:: *teacherPtr)(QString) = &Teacher::hungry;
    void(Student:: *studentPtr)(QString) = &Student::treat;
    connect(zt,teacherPtr,st,studentPtr);
    classIsOver();
}

Widget::~Widget()
{
}

// 触发信号
void Widget::classIsOver()
{
    emit zt->hungry("kao'leng'mian烤冷面");
}


```

student.cpp
```C
#include "student.h"
#include <QDebug>

Student::Student(QObject *parent) : QObject(parent)
{

}

// 槽函数的实现过程如下
void Student::treat()
{
    qDebug() << "请老师吃饭";
}


void Student::treat(QString foodName)
{
    qDebug() << "请老师吃饭了！,老师要吃：" << foodName.toUtf8().data() ;
}
```
student.h
```C
#ifndef STUDENT_H
#define STUDENT_H

#include <QObject>

class Student : public QObject
{
    Q_OBJECT
public:
    explicit Student(QObject *parent = nullptr);

signals:

public slots:
    // 自定义槽函数
    // 槽函数必须定义且必须要声明才可以使用
    void treat();
    void treat(QString);
};

#endif // STUDENT_H

```
teacher.h
```C
#ifndef TEACHER_H
#define TEACHER_H

#include <QObject>

class Teacher : public QObject
{
    Q_OBJECT
public:
    explicit Teacher(QObject *parent = nullptr);

signals:
    // 定义一个信号,信号必须为void类型,且信号不能实现
    void hungry();
    void hungry( QString foodName );
};

#endif // TEACHER_H
```
widget.h
```C
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>
#include <QString>
#include <QPushButton>
#include "student.h"
#include "teacher.h"

class Widget : public QWidget
{
    Q_OBJECT

public:
    Widget(QWidget *parent = nullptr);
    ~Widget();

    // 定义学生与老师类
    Teacher *zt;
    Student *st;

    // 定义信号发送函数
    void classIsOver();

};
#endif // WIDGET_H
```

**点击按钮触发信号：** 当我们点击按钮时，自动触发信号。只需需改widget中的内容。
```C
Widget::Widget(QWidget *parent): QWidget(parent)
{
    st = new Student(this);
    tt = new Teacher(this);

    QPushButton *btn = new QPushButton("发送邮件",this);

    void(Teacher:: *teacherPtr)(void) = &Teacher::send_mail;
    void(Student:: *studentPtr)(void) = &Student::read_mail;

    // 将btn绑定到button上，点击后触发tt 里面的teacherPtr -> 产生信号send_mail;
    connect(btn,&QPushButton::clicked,tt,teacherPtr);
    // 接着将产生信号绑定到 st 里面的student -> 也就是read_mail槽函数上。
    connect(tt,teacherPtr,st,studentPtr);
}
```

**匿名函数与槽**
```C
Widget::Widget(QWidget *parent): QWidget(parent), ui(new Ui::Widget)
{

    ui->setupUi(this);

    // 使用Lambda表达式,其中的[=] 对文件内的变量生效,其中[btn]则是对btn按钮生效
    // 默认会调用一次完成初始化,这是由()决定的函数调用。
    [=](){
        this->setWindowTitle("初始化..");
    }();

    // 使用mutable可以改变通过值传递的变量
    int number = 10;

    QPushButton *btn_ptr1 = new QPushButton("改变变量值",this);
    btn_ptr1->move(100,100);

    // 点击按钮改变内部变量的值,由于值传递所以不会影响外部变量的变化
    connect(btn_ptr1,&QPushButton::clicked,this,[=]()mutable{
       number = number + 100;
       std::cout << "inner: " << number << std::endl;
    });

    // 当点击以下按钮时number还是原值
    QPushButton *btn_ptr2 = new QPushButton("测试值传递",this);
    btn_ptr2->move(100,200);
    connect(btn_ptr2,&QPushButton::clicked,this,[=](){
       std::cout << "The Value: " << number << std::endl;
    });

    // 使用Lambda表达式返回值,有时存在返回的情况
    int ref = []()->int{
        return 1000;
    }();
    std::cout << "Return = " << ref << std::endl;
}
```
