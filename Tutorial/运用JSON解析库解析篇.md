JSON是一种简单的轻量级数据交换格式，Qt库为JSON的相关操作提供了完整的类支持，使用JSON解析文件之前需要先通过`TextStream`流将文件读入到字符串变量内，然后再通过`QJsonDocument`等库对该JSON格式进行解析，以提取出我们所需字段。

首先创建一个解析文件,命名为`config.json`我们将通过代码依次解析这个JSON文件中的每一个参数,具体解析代码如下:
```C
{
    "blog": "https://www.cnblogs.com/lyshark",
    "enable": true,
    "status": 1024,
    
    "GetDict": {"address":"192.168.1.1","username":"root","password":"123456","update":"2020-09-26"},
    "GetList": [1,2,3,4,5,6,7,8,9,0],
    
    "ObjectInArrayJson":
    {
        "One": ["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"],
        "Two": ["Sunday","Monday","Tuesday"]
    },
    
    "ArrayJson": [
        ["192.168.1.1","root","22"],
        ["192.168.1.2","root","23"],
        ["192.168.1.3","root","24"],
        ["192.168.1.4","root","25"],
        ["192.168.1.5","root","26"]
    ],
    
    "ObjectJson": [
        {"address":"192.168.1.1","username":"admin"},
        {"address":"192.168.1.2","username":"root"},
        {"address":"192.168.1.3","username":"lyshark"}
    ],
    
    "ObjectArrayJson": [
        {"uname":"root","ulist":[1,2,3,4,5]},
        {"uname":"lyshark","ulist":[11,22,33,44,55,66,77,88,99]}
    ],
    
    "NestingObjectJson": [
        {
            "uuid": "1001",
            "basic": {
                "lat": "12.657", 
                "lon": "55.789"
            }
        },
        {
            "uuid": "1002",
            "basic": {
                "lat": "31.24", 
                "lon": "25.55"
            }
        }
    ],
    
    "ArrayNestingArrayJson":
    [
        {
            "telephone": "1323344521",
            "path": [
                [
                    11.5,22.4,56.9
                ],
                [
                    19.4,34.6,44.7
                ]
            ]
        }
    ]
}
```

首先实现读写文本文件,通过QT中封装的`<QFile>`库可实现对文本文件的读取操作,读取JSON文件可使用该方式.
```C
#include <QCoreApplication>
#include <iostream>
#include <QString>
#include <QTextStream>
#include <QFile>
#include <QDir>
#include <QFileInfo>

#include <QJsonDocument>
#include <QJsonParseError>
#include <QJsonObject>
#include <QJsonArray>
#include <QJsonValue>
#include <QJsonValueRef>

// 传入文本路径,读取并输出
// https://www.cnblogs.com/lyshark
int readonly_string_file(QString file_path)
{
    QFile this_file_ptr(file_path);

    // 判断文件是否存在
    if(false == this_file_ptr.exists())
    {
        std::cout << "文件不存在" << std::endl;
        return 0;
    }

    /*
     * 文件打开属性包括如下
     * QIODevice::ReadOnly  只读方式打开
     * QIODevice::WriteOnly 写入方式打开
     * QIODevice::ReadWrite 读写方式打开
     * QIODevice::Append    追加方式打开
     * QIODevice::Truncate  截取方式打开
     * QIODevice::Text      文本方式打开
     */

    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        std::cout << "打开失败" << std::endl;
        return 0;
    }

    // 读取到文本中的字符串
    QString string_value = this_file_ptr.readAll();

    std::cout << "读入长度: " << this_file_ptr.size() << std::endl;
    std::cout << "字符串: " << string_value.toStdString() << std::endl;
    this_file_ptr.close();
}

// 逐行读取文本文件
void read_line_file()
{
    QFile this_file_ptr("d:/config.json");

    if(this_file_ptr.open((QIODevice::ReadOnly | QIODevice::Text)))
    {
        QByteArray byte_array;

        while(false == this_file_ptr.atEnd())
        {
            byte_array += this_file_ptr.readLine();
        }

        std::cout << "完整文本: " << QString(byte_array).toStdString() << std::endl;
        this_file_ptr.close();
    }
}

// 传入文本路径与写入内容,写入到文件
// https://www.cnblogs.com/lyshark
void write_string_file(QString file_path, QString string_value)
{
    QFile this_file_ptr(file_path);

    // 判断文件是否存在
    if(false == this_file_ptr.exists())
    {
        return;
    }

    // 打开失败
    if(false == this_file_ptr.open(QIODevice::ReadWrite | QIODevice::Text))
    {
        return;
    }

    //写入内容，注意需要转码，否则会报错
    QByteArray write_string = string_value.toUtf8();

    //写入QByteArray格式字符串
    this_file_ptr.write(write_string);
    this_file_ptr.close();
}

// 计算文件或目录大小
unsigned int GetFileSize(QString path)
{
    QFileInfo info(path);
    unsigned int ret = 0;
    if(info.isFile())
    {
        ret = info.size();
    }
    else if(info.isDir())
    {
        QDir dir(path);
        QFileInfoList list = dir.entryInfoList();
        for(int i = 0; i < list.count(); i++)
        {
            if((list[i].fileName() != ".") && (list[i].fileName() != ".."))
            {
                ret += GetFileSize(list[i].absoluteFilePath());
            }
        }
    }
    return ret;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    readonly_string_file("d:/config.json");

    // 计算文件或目录大小
    unsigned int file_size = GetFileSize("d:/xunjian");
    std::cout << "获取文件或目录大小: " << file_size << std::endl;

    // 覆盖写入文件
    QString write_file_path = "d:/test.json";
    QString write_string = "hello lyshark";
    write_string_file(write_file_path,write_string);

    return a.exec();
}
```

实现解析`根对象`中的`单一`的`键值对`,例如解析配置文件中的`blog,enable,status`等这些独立的字段值.
```C
// 读取JSON文本
// https://www.cnblogs.com/lyshark
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();
    this_file_ptr.close();
    return string_value;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    QString config = readonly_string("d:/config.json");
    if(config == "None")
    {
        return 0;
    }

    // 字符串格式化为JSON
    QJsonParseError err_rpt;
    QJsonDocument  root_document = QJsonDocument::fromJson(config.toUtf8(), &err_rpt);
    if(err_rpt.error != QJsonParseError::NoError)
    {
        std::cout << "JSON格式错误" << std::endl;
        return 0;
    }

    // 获取到Json字符串的根节点
    QJsonObject root_object = root_document.object();

    // 解析blog字段
    QString blog = root_object.find("blog").value().toString();
    std::cout << "字段对应的值 = > "<< blog.toStdString() << std::endl;

    // 解析enable字段
    bool enable = root_object.find("enable").value().toBool();
    std::cout << "是否开启状态: " << enable << std::endl;

    // 解析status字段
    int status = root_object.find("status").value().toInt();
    std::cout << "状态数值: " << status << std::endl;

    return a.exec();
}
```
实现解析简单的`单对象`与`单数组`结构,如上配置文件中的`GetDict`与`GetList`既是我们需要解析的内容.
```C
// 读取JSON文本
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();
    this_file_ptr.close();
    return string_value;
}

// https://www.cnblogs.com/lyshark
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    QString config = readonly_string("d:/config.json");
    if(config == "None")
    {
        return 0;
    }

    // 字符串格式化为JSON
    QJsonParseError err_rpt;
    QJsonDocument  root_document = QJsonDocument::fromJson(config.toUtf8(), &err_rpt);
    if(err_rpt.error != QJsonParseError::NoError)
    {
        std::cout << "JSON格式错误" << std::endl;
        return 0;
    }

    // 获取到Json字符串的根节点
    QJsonObject root_object = root_document.object();

    // 解析单一对象
    QJsonObject get_dict_ptr = root_object.find("GetDict").value().toObject();
    QVariantMap map = get_dict_ptr.toVariantMap();

    if(map.contains("address") && map.contains("username") && map.contains("password") && map.contains("update"))
    {
        QString address = map["address"].toString();
        QString username = map["username"].toString();
        QString password = map["password"].toString();
        QString update = map["update"].toString();

        std::cout
                  << " 地址: " << address.toStdString()
                  << " 用户名: " << username.toStdString()
                  << " 密码: " << password.toStdString()
                  << " 更新日期: " << update.toStdString()
                  << std::endl;
    }

    // 解析单一数组
    QJsonArray get_list_ptr = root_object.find("GetList").value().toArray();

    for(int index=0; index < get_list_ptr.count(); index++)
    {
        int ref_value = get_list_ptr.at(index).toInt();
        std::cout << "输出数组元素: " << ref_value << std::endl;
    }

    return a.exec();
}
```
实现解析`对象嵌套对象`且`对象中嵌套数组`结构,如上配置文件中的`ObjectInArrayJson`既是我们需要解析的内容.
```C
// 读取JSON文本
// https://www.cnblogs.com/lyshark
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();
    this_file_ptr.close();
    return string_value;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    QString config = readonly_string("d:/config.json");
    if(config == "None")
    {
        return 0;
    }

    // 字符串格式化为JSON
    QJsonParseError err_rpt;
    QJsonDocument  root_document = QJsonDocument::fromJson(config.toUtf8(), &err_rpt);
    if(err_rpt.error != QJsonParseError::NoError)
    {
        std::cout << "JSON格式错误" << std::endl;
        return 0;
    }

    // 获取到Json字符串的根节点
    QJsonObject root_object = root_document.object();

    // 找到Object对象
    QJsonObject one_object_json = root_object.find("ObjectInArrayJson").value().toObject();

    // 转为MAP映射
    QVariantMap map = one_object_json.toVariantMap();

    // 寻找One键
    QJsonArray array_one = map["One"].toJsonArray();

    for(int index=0; index < array_one.count(); index++)
    {
        QString value = array_one.at(index).toString();

        std::cout << "One => "<< value.toStdString() << std::endl;
    }

    // 寻找Two键
    QJsonArray array_two = map["Two"].toJsonArray();

    for(int index=0; index < array_two.count(); index++)
    {
        QString value = array_two.at(index).toString();

        std::cout << "Two => "<< value.toStdString() << std::endl;
    }

    return a.exec();
}
```
实现解析`数组中的数组`结构,如上配置文件中的`ArrayJson`既是我们需要解析的内容.
```C
// 读取JSON文本
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();
    this_file_ptr.close();
    return string_value;
}

// https://www.cnblogs.com/lyshark
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    QString config = readonly_string("d:/config.json");
    if(config == "None")
    {
        return 0;
    }

    // 字符串格式化为JSON
    QJsonParseError err_rpt;
    QJsonDocument  root_document = QJsonDocument::fromJson(config.toUtf8(), &err_rpt);
    if(err_rpt.error != QJsonParseError::NoError)
    {
        std::cout << "json 格式错误" << std::endl;
        return 0;
    }

    // 获取到Json字符串的根节点
    QJsonObject root_object = root_document.object();

    // 获取MyJson数组
    QJsonValue array_value = root_object.value("ArrayJson");

    // 验证节点是否为数组
    if(array_value.isArray())
    {
        // 得到数组个数
        int array_count = array_value.toArray().count();

        // 循环数组个数
        for(int index=0;index <= array_count;index++)
        {
            QJsonValue parset = array_value.toArray().at((index));
            if(parset.isArray())
            {
                QString address = parset.toArray().at(0).toString();
                QString username = parset.toArray().at(1).toString();
                QString userport = parset.toArray().at(2).toString();

                std::cout
                        << "地址: " << address.toStdString()
                        << " 用户名: " << username.toStdString()
                        << " 端口号: " << userport.toStdString()
                << std::endl;
            }
        }
    }

    return a.exec();
}
```
实现解析`数组中的多对象`结构,如上配置文件中的`ObjectJson`既是我们需要解析的内容.
```C
// 读取JSON文本
// https://www.cnblogs.com/lyshark
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();
    this_file_ptr.close();
    return string_value;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    QString config = readonly_string("d:/config.json");
    if(config == "None")
    {
        return 0;
    }

    // 字符串格式化为JSON
    QJsonParseError err_rpt;
    QJsonDocument  root_document = QJsonDocument::fromJson(config.toUtf8(), &err_rpt);
    if(err_rpt.error != QJsonParseError::NoError)
    {
        std::cout << "json 格式错误" << std::endl;
        return 0;
    }

    // 获取到Json字符串的根节点
    QJsonObject root_object = root_document.object();

    // 获取MyJson数组
    QJsonValue object_value = root_object.value("ObjectJson");

    // 验证是否为数组
    if(object_value.isArray())
    {
        // 获取对象个数
        int object_count = object_value.toArray().count();

        // 循环个数
        for(int index=0;index <= object_count;index++)
        {
            QJsonObject obj = object_value.toArray().at(index).toObject();

            // 验证数组不为空
            if(!obj.isEmpty())
            {
                QString address = obj.value("address").toString();
                QString username = obj.value("username").toString();

                std::cout << "地址: " << address.toStdString() << " 用户: " << username.toStdString() << std::endl;
            }
        }
    }

    return a.exec();
}
```
实现解析`数组中对象中的嵌套数组`结构,如上配置文件中的`ObjectArrayJson`既是我们需要解析的内容.
```C
// 读取JSON文本
// https://www.cnblogs.com/lyshark
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();
    this_file_ptr.close();
    return string_value;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    QString config = readonly_string("d:/config.json");
    if(config == "None")
    {
        return 0;
    }

    // 字符串格式化为JSON
    // https://www.cnblogs.com/lyshark
    QJsonParseError err_rpt;
    QJsonDocument  root_document = QJsonDocument::fromJson(config.toUtf8(), &err_rpt);
    if(err_rpt.error != QJsonParseError::NoError)
    {
        std::cout << "json 格式错误" << std::endl;
        return 0;
    }

    // 获取到Json字符串的根节点
    QJsonObject root_object = root_document.object();

    // 获取MyJson数组
    QJsonValue object_value = root_object.value("ObjectArrayJson");

    // 验证是否为数组
    if(object_value.isArray())
    {
        // 获取对象个数
        int object_count = object_value.toArray().count();

        // 循环个数
        for(int index=0;index <= object_count;index++)
        {
            QJsonObject obj = object_value.toArray().at(index).toObject();

            // 验证数组不为空
            if(!obj.isEmpty())
            {
                QString uname = obj.value("uname").toString();
                std::cout << "用户名: " << uname.toStdString() <<  std::endl;

                // 解析该用户的数组
                int array_count = obj.value("ulist").toArray().count();

                std::cout << "数组个数: "<< array_count << std::endl;

                for(int index=0;index < array_count;index++)
                {
                    QJsonValue parset = obj.value("ulist").toArray().at(index);

                    int val = parset.toInt();

                    std::cout << "Value = > "<< val << std::endl;
                }
            }
        }
    }

    return a.exec();
}
```
实现解析`数组嵌套匿名对象嵌套对象`结构,如上配置文件中的`NestingObjectJson`既是我们需要解析的内容.
```C
// 读取JSON文本
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();
    this_file_ptr.close();
    return string_value;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    // https://www.cnblogs.com/lyshark
    QString config = readonly_string("d:/config.json");
    if(config == "None")
    {
        return 0;
    }

    // 字符串格式化为JSON
    QJsonParseError err_rpt;
    QJsonDocument  root_document = QJsonDocument::fromJson(config.toUtf8(), &err_rpt);
    if(err_rpt.error != QJsonParseError::NoError)
    {
        std::cout << "json 格式错误" << std::endl;
        return 0;
    }

    // 获取到Json字符串的根节点
    QJsonObject root_object = root_document.object();

    // 获取NestingObjectJson数组
    QJsonValue array_value = root_object.value("NestingObjectJson");

    // 验证节点是否为数组
    if(array_value.isArray())
    {
        // 得到内部对象个数
        int count = array_value.toArray().count();
        std::cout << "对象个数: " << count << std::endl;

        for(int index=0; index < count; index++)
        {
            // 得到数组中的index下标中的对象
            QJsonObject array_object = array_value.toArray().at(index).toObject();

            // 开始解析basic中的数据
            QJsonObject basic = array_object.value("basic").toObject();

            QString lat = basic.value("lat").toString();
            QString lon = basic.value("lon").toString();

            std::cout << "解析basic中的lat字段: " << lat.toStdString() << std::endl;
            std::cout << "解析basic中的lon字段: " << lon.toStdString()<< std::endl;

            // 解析单独字段
            QString status = array_object.value("status").toString();
            std::cout << "解析字段状态: " << status.toStdString() << std::endl;
        }
    }

    return a.exec();
}
```
实现解析`数组嵌套对象`且`对象内嵌套双层数组`结构,如上配置文件中的`ArrayNestingArrayJson`既我们需要解析的内容.
```C
// 读取JSON文本
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();
    this_file_ptr.close();
    return string_value;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读取文件
    // https://www.cnblogs.com/lyshark
    QString config = readonly_string("d:/config.json");
    if(config == "None")
    {
        return 0;
    }

    // 字符串格式化为JSON
    QJsonParseError err_rpt;
    QJsonDocument  root_document = QJsonDocument::fromJson(config.toUtf8(), &err_rpt);
    if(err_rpt.error != QJsonParseError::NoError)
    {
        std::cout << "json 格式错误" << std::endl;
        return 0;
    }

    // 获取到Json字符串的根节点
    QJsonObject root_object = root_document.object();

    // 获取NestingObjectJson数组
    QJsonValue array_value = root_object.value("ArrayNestingArrayJson");

    // 验证节点是否为数组
    if(array_value.isArray())
    {
        // 得到数组中的0号下标中的对象
        QJsonObject array_object = array_value.toArray().at(0).toObject();

        // 解析手机号字符串
        QString telephone = array_object.value("telephone").toString();
        std::cout << "手机号: " << telephone.toStdString() << std::endl;

        // 定位外层数组
        QJsonArray root_array = array_object.find("path").value().toArray();
        std::cout << "外层循环计数: " << root_array.count() << std::endl;

        for(int index=0; index < root_array.count(); index++)
        {
            // 定位内层数组
            QJsonArray sub_array = root_array.at(index).toArray();
            std::cout << "内层循环计数: "<< sub_array.count() << std::endl;

            for(int sub_count=0; sub_count < sub_array.count(); sub_count++)
            {
                // 每次取出最里层数组元素
                float var = sub_array.toVariantList().at(sub_count).toFloat();

                std::cout << "输出元素: " << var << std::endl;
                // std::cout << sub_array.toVariantList().at(0).toFloat() << std::endl;
            }
        }
    }

    return a.exec();
}
```
