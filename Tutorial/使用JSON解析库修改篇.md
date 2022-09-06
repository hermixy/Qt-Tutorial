JSON是一种轻量级的数据交换格式,它是基于ECMAScript的一个子集,使用完全独立于编程语言的文本格式来存储和表示数据,简洁清晰的的层次结构使得JSON成为理想的数据交换语言,Qt库为JSON的相关操作提供了完整的类支持.

<!--more-->

创建一个解析文件,命名为`config.json`我们将通过代码依次解析这个JSON文件中的每一个参数,具体解析代码如下:
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

实现`修改单层根节点`下面指定的节点元素,修改的原理是读入到内存替换后在全部写出到文件.
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

// 写出JSON到文件
bool writeonly_string(QString file_path, QString file_data)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.open(QIODevice::WriteOnly | QIODevice::Text))
    {
        return false;
    }

    QByteArray write_byte = file_data.toUtf8();

    this_file_ptr.write(write_byte,write_byte.length());
    this_file_ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读文件
    QString readonly_config = readonly_string("d:/config.json");

    // 开始解析 解析成功返回QJsonDocument对象否则返回null
    QJsonParseError err_rpt;
    QJsonDocument root_document = QJsonDocument::fromJson(readonly_config.toUtf8(), &err_rpt);
    if (err_rpt.error != QJsonParseError::NoError && !root_document.isNull())
    {
        return 0;
    }

    // 获取根节点
    QJsonObject root = root_document.object();

    // 修改根节点下的子节点
    root["blog"] = "https://www.baidu.com";
    root["enable"] = false;
    root["status"] = 2048;

    // 将object设置为本文档的主要对象
    root_document.setObject(root);

    // 紧凑模式输出
    // https://www.cnblogs.com/lyshark
    QByteArray root_string_compact = root_document.toJson(QJsonDocument::Compact);
    std::cout << "紧凑模式: " << root_string_compact.toStdString() << std::endl;

    // 规范模式输出
    QByteArray root_string_indented = root_document.toJson(QJsonDocument::Indented);
    std::cout << "规范模式: " << root_string_indented.toStdString() << std::endl;

    // 分别写出两个配置文件
    writeonly_string("d:/indented_config.json",root_string_indented);
    writeonly_string("d:/compact_config.json",root_string_compact);

    return a.exec();
}
```
实现`修改单层对象与数组`下面指定的节点元素,如上配置文件中的`GetDict/GetList`既是我们需要解析的内容.
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

// 写出JSON到文件
bool writeonly_string(QString file_path, QString file_data)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.open(QIODevice::WriteOnly | QIODevice::Text))
    {
        return false;
    }

    QByteArray write_byte = file_data.toUtf8();

    this_file_ptr.write(write_byte,write_byte.length());
    this_file_ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读文件
    QString readonly_config = readonly_string("d:/config.json");

    // 开始解析 解析成功返回QJsonDocument对象否则返回null
    QJsonParseError err_rpt;
    QJsonDocument root_document = QJsonDocument::fromJson(readonly_config.toUtf8(), &err_rpt);
    if (err_rpt.error != QJsonParseError::NoError && !root_document.isNull())
    {
        return 0;
    }

    // 获取根节点
    QJsonObject root = root_document.object();

    // --------------------------------------------------------------------
    // 修改GetDict对象内容
    // --------------------------------------------------------------------

    // 找到对象地址,并修改
    QJsonObject get_dict_ptr = root.find("GetDict").value().toObject();

    // 修改对象内存数据
    get_dict_ptr["address"] = "127.0.0.1";
    get_dict_ptr["username"] = "lyshark";
    get_dict_ptr["password"] = "12345678";
    get_dict_ptr["update"] = "2021-09-25";

    // 赋值给根
    root["GetDict"] = get_dict_ptr;

    // 将对象设置到根
    root_document.setObject(root);

    // 规范模式
    QByteArray root_string_indented = root_document.toJson(QJsonDocument::Indented);

    // 写配置文件
    writeonly_string("d:/indented_config.json",root_string_indented);

    // --------------------------------------------------------------------
    // 修改GetList数组内容
    // --------------------------------------------------------------------

    // 找到数组并修改
    QJsonArray get_list_ptr = root.find("GetList").value().toArray();

    // 替换指定下标的数组元素
    get_list_ptr.replace(0,22);
    get_list_ptr.replace(1,33);

    // 将当前数组元素直接覆盖到原始位置
    QJsonArray item = {"admin","root","lyshark"};
    get_list_ptr = item;

    // 设置到原始数组
    root["GetList"] = get_list_ptr;
    root_document.setObject(root);

    QByteArray root_string_list_indented = root_document.toJson(QJsonDocument::Indented);
    writeonly_string("d:/indented_config.json",root_string_list_indented);

    return a.exec();
}
```
实现`修改对象内对象Value列表`下面指定的节点元素,如上配置文件中的`ObjectInArrayJson`既是我们需要解析的内容.
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

// 写出JSON到文件
// https://www.cnblogs.com/lyshark
bool writeonly_string(QString file_path, QString file_data)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.open(QIODevice::WriteOnly | QIODevice::Text))
    {
        return false;
    }

    QByteArray write_byte = file_data.toUtf8();

    this_file_ptr.write(write_byte,write_byte.length());
    this_file_ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读文件
    QString readonly_config = readonly_string("d:/config.json");

    // 开始解析 解析成功返回QJsonDocument对象否则返回null
    QJsonParseError err_rpt;
    QJsonDocument root_document = QJsonDocument::fromJson(readonly_config.toUtf8(), &err_rpt);
    if (err_rpt.error != QJsonParseError::NoError && !root_document.isNull())
    {
        return 0;
    }

    // 获取根节点
    QJsonObject root = root_document.object();

    // --------------------------------------------------------------------
    // 修改对象中的列表
    // --------------------------------------------------------------------

    // 找到对象地址并修改
    QJsonObject get_dict_ptr = root.find("ObjectInArrayJson").value().toObject();

    // 迭代器输出对象中的数据
    QJsonObject::iterator it;
    for(it=get_dict_ptr.begin(); it!= get_dict_ptr.end(); it++)
    {
        QString key = it.key();
        QJsonArray value = it.value().toArray();

        std::cout << "Key = " << key.toStdString() << " ValueCount = " << value.count() << std::endl;

        // 循环输出元素
        for(int index=0; index < value.count(); index++)
        {
            QString val = value.toVariantList().at(index).toString();
            std::cout << "-> " << val.toStdString() << std::endl;
        }
    }

    // 迭代寻找需要修改的元素并修改
    for(it=get_dict_ptr.begin(); it!= get_dict_ptr.end(); it++)
    {
        QString key = it.key();

        // 如果找到了指定的Key 则将Value中的列表替换到其中
        if(key.toStdString() == "Two")
        {
            // 替换整个数组
            QJsonArray value = {1,2,3,4,5};
            get_dict_ptr[key] = value;
            break;
        }
        else if(key.toStdString() == "One")
        {
            // 替换指定数组元素
            QJsonArray array = get_dict_ptr[key].toArray();

            array.replace(0,"lyshark");
            array.replace(1,"lyshark");
            array.removeAt(1);

            // 写回原JSON字符串
            get_dict_ptr[key] = array;
        }
    }

    // 赋值给根
    root["ObjectInArrayJson"] = get_dict_ptr;

    // 将对象设置到根
    root_document.setObject(root);

    // 规范模式
    QByteArray root_string_indented = root_document.toJson(QJsonDocument::Indented);

    // 写配置文件
    writeonly_string("d:/indented_config.json",root_string_indented);

    return a.exec();
}
```
实现`修改匿名数组中的数组元素`下面指定的节点元素,如上配置文件中的`ArrayJson`既是我们需要解析的内容.
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

// 写出JSON到文件
bool writeonly_string(QString file_path, QString file_data)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.open(QIODevice::WriteOnly | QIODevice::Text))
    {
        return false;
    }

    QByteArray write_byte = file_data.toUtf8();

    this_file_ptr.write(write_byte,write_byte.length());
    this_file_ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读文件
    QString readonly_config = readonly_string("d:/config.json");

    // 开始解析 解析成功返回QJsonDocument对象否则返回null
    QJsonParseError err_rpt;
    QJsonDocument root_document = QJsonDocument::fromJson(readonly_config.toUtf8(), &err_rpt);
    if (err_rpt.error != QJsonParseError::NoError && !root_document.isNull())
    {
        return 0;
    }

    // 获取根节点
    QJsonObject root = root_document.object();

    // --------------------------------------------------------------------
    // 修改数组中的数组
    // --------------------------------------------------------------------

    QJsonArray array_value = root.value("ArrayJson").toArray();
    int insert_index = 0;

    // 循环查找IP地址,找到后将其弹出
    for(int index=0; index < array_value.count(); index++)
    {
        QJsonValue parset = array_value.at(index);

        if(parset.isArray())
        {
            QString address = parset.toArray().at(0).toString();

            // 寻找指定行下标,并将其弹出
            if(address == "192.168.1.3")
            {
                std::cout << "找到该行下标: " << index << std::endl;
                insert_index = index;
                array_value.removeAt(index);
            }
        }
    }

    // 新增一行IP地址
    QJsonArray item = {"192.168.1.3","lyshark","123456"};
    array_value.insert(insert_index,item);

    // 赋值给根
    root["ArrayJson"] = array_value;

    // 将对象设置到根
    root_document.setObject(root);

    // 规范模式
    QByteArray root_string_indented = root_document.toJson(QJsonDocument::Indented);

    // 写配置文件
    // https://www.cnblogs.com/lyshark
    writeonly_string("d:/indented_config.json",root_string_indented);

    return a.exec();
}
```
实现`修改数组中对象元素`下面指定的节点元素,如上配置文件中的`ObjectJson`既是我们需要解析的内容.
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

// 写出JSON到文件
// https://www.cnblogs.com/lyshark
bool writeonly_string(QString file_path, QString file_data)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.open(QIODevice::WriteOnly | QIODevice::Text))
    {
        return false;
    }

    QByteArray write_byte = file_data.toUtf8();

    this_file_ptr.write(write_byte,write_byte.length());
    this_file_ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读文件
    QString readonly_config = readonly_string("d:/config.json");

    // 开始解析 解析成功返回QJsonDocument对象否则返回null
    QJsonParseError err_rpt;
    QJsonDocument root_document = QJsonDocument::fromJson(readonly_config.toUtf8(), &err_rpt);
    if (err_rpt.error != QJsonParseError::NoError && !root_document.isNull())
    {
        return 0;
    }

    // 获取根节点
    QJsonObject root = root_document.object();

    // --------------------------------------------------------------------
    // 修改数组中的对象数值
    // --------------------------------------------------------------------

    QJsonArray array_value = root.value("ObjectJson").toArray();

    for(int index=0; index < array_value.count(); index++)
    {
        QJsonObject object_value = array_value.at(index).toObject();

        /*
        // 第一种输出方式
        if(!object_value.isEmpty())
        {
            QString address = object_value.value("address").toString();
            QString username = object_value.value("username").toString();
            std::cout << "地址: " << address.toStdString() << " 用户名: " << username.toStdString() << std::endl;
        }

        // 第二种输出方式
        QVariantMap map = object_value.toVariantMap();
        if(map.contains("address") && map.contains("username"))
        {
            QString address_map = map["address"].toString();
            QString username_map = map["username"].toString();
            std::cout << "地址: " << address_map.toStdString() << " 用户名: " << username_map.toStdString() << std::endl;
        }
        */

        // 开始移除指定行
        QVariantMap map = object_value.toVariantMap();
        if(map.contains("address") && map.contains("username"))
        {
            QString address_map = map["address"].toString();

            // 如果是指定IP则首先移除该行
            if(address_map == "192.168.1.3")
            {
                // 首先根据对象序号移除当前对象
                array_value.removeAt(index);

                // 接着增加一个新的对象
                object_value["address"] = "127.0.0.1";
                object_value["username"] = "lyshark";

                // 插入到移除的位置上
                array_value.insert(index,object_value);
                break;
            }
        }
    }

    // 赋值给根
    root["ObjectJson"] = array_value;

    // 将对象设置到根
    root_document.setObject(root);

    // 规范模式
    QByteArray root_string_indented = root_document.toJson(QJsonDocument::Indented);

    // 写配置文件
    writeonly_string("d:/indented_config.json",root_string_indented);

    return a.exec();
}
```
实现`修改对象中数组元素`下面指定的节点元素,如上配置文件中的`ObjectArrayJson`既是我们需要解析的内容.
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

// 写出JSON到文件
// https://www.cnblogs.com/lyshark
bool writeonly_string(QString file_path, QString file_data)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.open(QIODevice::WriteOnly | QIODevice::Text))
    {
        return false;
    }

    QByteArray write_byte = file_data.toUtf8();

    this_file_ptr.write(write_byte,write_byte.length());
    this_file_ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读文件
    QString readonly_config = readonly_string("d:/config.json");

    // 开始解析 解析成功返回QJsonDocument对象否则返回null
    QJsonParseError err_rpt;
    QJsonDocument root_document = QJsonDocument::fromJson(readonly_config.toUtf8(), &err_rpt);
    if (err_rpt.error != QJsonParseError::NoError && !root_document.isNull())
    {
        return 0;
    }

    // 获取根节点
    QJsonObject root = root_document.object();

    // --------------------------------------------------------------------
    // 修改对象中的数组元素
    // --------------------------------------------------------------------

    QJsonArray array_value = root.value("ObjectArrayJson").toArray();

    for(int index=0; index < array_value.count(); index++)
    {
        QJsonObject object_value = array_value.at(index).toObject();

        // 开始移除指定行
        QVariantMap map = object_value.toVariantMap();
        if(map.contains("uname") && map.contains("ulist"))
        {
            QString uname = map["uname"].toString();

            // 如果是指定IP则首先移除该行
            if(uname == "lyshark")
            {
                QJsonArray ulist_array = map["ulist"].toJsonArray();

                // 替换指定数组元素
                ulist_array.replace(0,100);
                ulist_array.replace(1,200);
                ulist_array.replace(2,300);

                // 输出替换后数组元素
                for(int index =0; index < ulist_array.count(); index ++)
                {
                    int val = ulist_array.at(index).toInt();
                    std::cout << "替换后: " << val << std::endl;
                }

                // 首先根据对象序号移除当前对象
                array_value.removeAt(index);

                // 接着增加一个新的对象与新列表
                object_value["uname"] = uname;
                object_value["ulist"] = ulist_array;

                // 插入到移除的位置上
                array_value.insert(index,object_value);
                break;
            }
        }
    }

    // 赋值给根
    root["ObjectArrayJson"] = array_value;

    // 将对象设置到根
    root_document.setObject(root);

    // 规范模式
    QByteArray root_string_indented = root_document.toJson(QJsonDocument::Indented);

    // 写配置文件
    writeonly_string("d:/indented_config.json",root_string_indented);

    return a.exec();
}
```
实现`修改对象嵌套对象嵌套对象`下面指定的节点元素,如上配置文件中的`NestingObjectJson`既是我们需要解析的内容.
```C
// 读取JSON文本
QString readonly_string(QString file_path)
{
    // https://www.cnblogs.com/lyshark
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

// 写出JSON到文件
bool writeonly_string(QString file_path, QString file_data)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.open(QIODevice::WriteOnly | QIODevice::Text))
    {
        return false;
    }

    QByteArray write_byte = file_data.toUtf8();

    this_file_ptr.write(write_byte,write_byte.length());
    this_file_ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读文件
    QString readonly_config = readonly_string("d:/config.json");

    // 开始解析 解析成功返回QJsonDocument对象否则返回null
    QJsonParseError err_rpt;
    QJsonDocument root_document = QJsonDocument::fromJson(readonly_config.toUtf8(), &err_rpt);
    if (err_rpt.error != QJsonParseError::NoError && !root_document.isNull())
    {
        return 0;
    }

    // 获取根节点
    // https://www.cnblogs.com/lyshark
    QJsonObject root = root_document.object();
    int insert_index = 0;

    // --------------------------------------------------------------------
    // 修改对象中嵌套对象嵌套对象
    // --------------------------------------------------------------------

    // 寻找节点中数组位置
    QJsonArray array_object = root.find("NestingObjectJson").value().toArray();
    std::cout << "节点数量: " << array_object.count() << std::endl;

    for(int index=0; index < array_object.count(); index++)
    {
        // 循环寻找UUID
        QJsonObject object = array_object.at(index).toObject();

        QString uuid = object.value("uuid").toString();

        // 如果找到了则移除该元素
        if(uuid == "1002")
        {
            insert_index = index;
            array_object.removeAt(index);
            break;
        }
    }

    // --------------------------------------------------------------------
    // 开始插入新对象
    // --------------------------------------------------------------------

    // 开始创建新的对象
    QJsonObject sub_json;

    sub_json.insert("lat","100.5");
    sub_json.insert("lon","200.5");

    QJsonObject new_json;

    new_json.insert("uuid","1005");
    new_json.insert("basic",sub_json);

    // 将对象插入到原位置上
    array_object.insert(insert_index,new_json);

    // 赋值给根
    root["NestingObjectJson"] = array_object;

    // 将对象设置到根
    root_document.setObject(root);

    // 规范模式
    QByteArray root_string_indented = root_document.toJson(QJsonDocument::Indented);

    // 写配置文件
    writeonly_string("d:/indented_config.json",root_string_indented);

    return a.exec();
}
```
实现`修改对象嵌套多层数组`下面指定的节点元素,如上配置文件中的`ArrayNestingArrayJson`既是我们需要解析的内容.
```C
// 读取JSON文本
QString readonly_string(QString file_path)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.exists())
    {
        return "None";
    }
    // https://www.cnblogs.com/lyshark
    if(false == this_file_ptr.open(QIODevice::ReadOnly | QIODevice::Text))
    {
        return "None";
    }
    QString string_value = this_file_ptr.readAll();

    this_file_ptr.close();
    return string_value;
}

// 写出JSON到文件
bool writeonly_string(QString file_path, QString file_data)
{
    QFile this_file_ptr(file_path);
    if(false == this_file_ptr.open(QIODevice::WriteOnly | QIODevice::Text))
    {
        return false;
    }

    QByteArray write_byte = file_data.toUtf8();

    this_file_ptr.write(write_byte,write_byte.length());
    this_file_ptr.close();
    return true;
}

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // 读文件
    QString readonly_config = readonly_string("d:/config.json");

    // 开始解析 解析成功返回QJsonDocument对象否则返回null
    QJsonParseError err_rpt;
    QJsonDocument root_document = QJsonDocument::fromJson(readonly_config.toUtf8(), &err_rpt);
    if (err_rpt.error != QJsonParseError::NoError && !root_document.isNull())
    {
        return 0;
    }

    // 获取根节点
    QJsonObject root = root_document.object();
    int insert_index = 0;

    // --------------------------------------------------------------------
    // 修改对象中嵌套双层数组
    // --------------------------------------------------------------------

    // 寻找节点中数组位置
    QJsonArray array_object = root.find("ArrayNestingArrayJson").value().toArray();
    std::cout << "节点数量: " << array_object.count() << std::endl;

    for(int index=0; index < array_object.count(); index++)
    {
        // 循环寻找UUID
        QJsonObject object = array_object.at(index).toObject();

        QString uuid = object.value("telephone").toString();

        // 如果找到了则移除该元素
        if(uuid == "1323344521")
        {
            insert_index = index;
            array_object.removeAt(index);
            break;
        }
    }

    // --------------------------------------------------------------------
    // 开始新的数组元素
    // --------------------------------------------------------------------

    QJsonArray array;
    QJsonArray x,y;

    // 追加子数组
    x.append(11.5);
    x.append(22.4);
    x.append(33.6);

    y.append(56.7);
    y.append(78.9);
    y.append(98.4);

    // 追加到外层数组
    array.append(x);
    array.append(y);

    // 创建{}对象节点
    QJsonObject object;

    object["telephone"] = "1323344521";
    object["path"] = array;

    // 将对象插入到原位置上
    array_object.insert(insert_index,object);

    // 赋值给根
    root["ArrayNestingArrayJson"] = array_object;

    // 将对象设置到根
    root_document.setObject(root);

    // 规范模式
    QByteArray root_string_indented = root_document.toJson(QJsonDocument::Indented);

    // 写配置文件
    writeonly_string("d:/indented_config.json",root_string_indented);

    return a.exec();
}
```
