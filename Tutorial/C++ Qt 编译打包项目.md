Qt程序编译后，需要去qt目录拷贝几个文件，与qt程序放在一起该程序才可以脱离开发环境而独立运行下去，在开发环境下编译好代码以后，还需要进行以下操作将其打包才可以在别的机器上正常运行。

 - QT的下载地址如下:
  - https://download.qt.io/new_archive/qt/5.11/5.11.3/
  - https://download.qt.io/archive/qt/5.14/5.14.2/

Qt项目打包有两种方式，第一种是自己打包项目，此方法需要将我们需要用到的库手动拷贝出来，并放入工程目录下。
 - 1.去Qt安装目录的bin目录中将libgcc_s_dw2-1.dll 、libstdc++-6.dll、libwinpthread-1.dll、Qt5Core.dll、Qt5Gui.dll 和 Qt5Widgets.dll 这 6 个文件复制出来。
 - 2.将`C:\Qt\Qt5.11.3\5.11\mingw49_32\plugins`目录中的`platforms`文件夹复制出来，里面只需要保留 qwindows.dll 文件即可。

<br>

如果是自动打包我们可以进入Qt提供的命令行页面，跳转到需要打包程序的目录下，执行以下命令。
 - 打包命令: `windeployqt untitled.exe`

<br>

如果打包时需要去掉不需要的库文件，我们可以指定`--no-`参数排除多余的动态链接库。
 - 打包命令: `windeployqt --no-angle --no-opengl-sw untitled.exe`

当我们打包完成后，可以手动删除多余文件，只保留如下文件即可，其他的可全部裁掉。

![image](https://user-images.githubusercontent.com/52789403/188526768-1401c06c-27dc-4855-a41e-3c9eb94433af.png)
