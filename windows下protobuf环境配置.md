本示例用C++版本
protobuf的安装要用源码，否则使用时会找不到库

### 必备工具
vs2019，cmake，git
先下载安装好以上工具，并将cmake，git添加到环境变量里面
### 安装过程
- 先[下载protobuf](https://github.com/google/protobuf/releases)，注意下载源码版本
- 解压后在cmake文件夹下，有官方的安装教程，但是那个在使用cmake命令时会报`The system is: Windows - 10.0.14393 - AMD64`错误，原因在于cmake的错误使用。
- 打开cmke-gui
![cmake界面](https://img-blog.csdn.net/20180810101234276?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hwX2NwcA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
- 上面时cmake的文件目录，下面路径是生成路径，生成路径建议新开文件夹为build
新版本的cmake界面不同，generator这里我选择的是`Visual Studio 16 2019`
![generator界面](https://img-blog.csdn.net/20180810101735955?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hwX2NwcA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![生成generator](https://img-blog.csdn.net/20180810101947760?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hwX2NwcA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
- 打开工程后，只编译以下工程
![编译工程](https://img-blog.csdn.net/20180810102556613?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hwX2NwcA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
- 在生成的文件夹中找到protoc.exe，并将路径加入环境变量
- 打开cmd，输入protoc，不是未识别即成功

### 参考
https://blog.csdn.net/dai_jing/article/details/83010324
https://blog.csdn.net/Dillon2015/article/details/83896385
