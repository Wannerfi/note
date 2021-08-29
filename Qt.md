##### QMainWindow

* 菜单栏 最多有一个
* * QMenuBar *bar = MenuBar();
  * setMenuBar(bar);
  * QMenu *fileMenu = bar->addMenu("file") //创建菜单
  * QAction *newAction = fileMenu->addAction("new");//创建菜单项
  * 添加分割线 fileMenu->addSeparator();
* 工具栏 多个
* * QToolBar *toolbar = new QToolBar(this);
  * addToolBar(Qt::LeftToolBarArea, toolbar);
  * 设置 后期停靠区域，浮动，移动
  * 添加菜单项或小控件
* 状态栏
* * QStatusBar *stBar = statusBar();
  * 设置到窗口中  setStatusBar(stBar);
  * stBar->addWidget(label);放左侧信息
  * stBar->addPermanentWidget(label)放右侧信息
* 铆接部件 浮动窗口 多个
* * QDockWidget
  * addDockWidget(默认停靠区域，浮动窗口指针)
  * 设置后期停靠区域
* 设置核心部件
* * setCentralWidget(edit)



##### 资源文件

* 将图片文件拷贝到项目位置下
* 右键项目 - 添加新文件 - Qt - Qt recourse             - 给资源文件起名
* res生成res.qrc
* open in editor 编辑资源
* 添加前缀，添加文件
* 使用"：+q前缀名+文件名"

```c++
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    ui->actionnew->setIcon(QIcon(":/icon/新建.svg"));
    ui->actionopen->setIcon(QIcon(":/icon/打开.svg"));
}
```





##### 对话框

* 分类

* * 模态对话框 不可以对其他窗口进行操作
  * QDialog dif(this)
  * dig.exec()

  ```c++
  MainWindow::MainWindow(QWidget *parent)
      : QMainWindow(parent)
      , ui(new Ui::MainWindow)
  {
      ui->setupUi(this);
      ui->actionnew->setIcon(QIcon(":/icon/新建.svg"));
  
      connect((ui->actionnew), &QAction::triggered, [=]{
          QDialog dlg(this);
          //dlg.resize(200, 100);
          dlg.exec();
      });
  }
  ```

* * 非模态对话框 可以对其他窗口进行操作

  ```c++
  MainWindow::MainWindow(QWidget *parent)
      : QMainWindow(parent)
      , ui(new Ui::MainWindow)
  {
      ui->setupUi(this);
      ui->actionnew->setIcon(QIcon(":/icon/新建.svg"));
  
      connect((ui->actionnew), &QAction::triggered, [=]{
          QDialog *dlg = new QDialog(this);
          dlg->resize(200, 100);
          dlg->show();
          dlg->setAttribute(Qt::WA_DeleteOnClose);//55号 属性
          //窗口关闭时释放，防止内存泄漏
      });
  }
  ```

  * new对象，防止变量在函数结束的时候被释放
  * 返回值也是StandarButton类型，利用返回值判断用户输入

  ```c++
  MainWindow::MainWindow(QWidget *parent)
      : QMainWindow(parent)
      , ui(new Ui::MainWindow)
  {
      ui->setupUi(this);
      ui->actionnew->setIcon(QIcon(":/icon/新建.svg"));
  
      connect((ui->actionnew), &QAction::triggered, [=]{
          //QMessageBox::critical(this, "critical", "错误");
          if(QMessageBox::Save ==
                  QMessageBox::question(this, "question", "提问", QMessageBox::Save | QMessageBox::Cancel, QMessageBox::Cancel))
          {
              qDebug() << "Save";
          } else {
              qDebug() << "Cancel";
          }
      });
  }
  ```

  

* 标准对话框

> QColorDialog, 选择颜色
>
> QFileDialog 选择文件或者目录
>
> QFontDialog 选择字体
>
> QInputDialog  允许用户输入一个值，并返回该值
>
> QMessageBox 模态对话框，用于显示信息，询问问题
>
> QPageSetupDialog  为打印机提供纸张相关的选项
>
> QPrintDialog 打印机配置
>
> QPrintPreviewDialog 打印预览
>
> QProgressDialog 显示操作过程

* 其他对话框

  > ```
  > QColor color = QColorDialog::getColor(QColor());
  > QFileDialog::getOpenFileName(this, "打开文件", "*.txt");//txt限制可选文件类型，返回值是选取的路径，QString
  > QFontDialog::getFont(&flag, QFont("楷体", 35));//字体对话框
  > ```



##### 界面布局

##### 控件

* 按钮组

* * QPushButton 常用按钮

  * QToolButton  工具按钮，用于显示图片

    toolButtonStyle 凸起风格 autoRaise

  * radioButton单选按钮，设置默认ui->rBtnMan->setChecked(true);

  * checkbox多选按钮，监听状态，0未选，1半选，2选中

* QListWidget 列表容器

* * QListWidgetitem *item一行内容
  * ui->listWidget->additem(item)
  * 设置居中方式item->setTextAlignment(Qt::AlignHCenter);
  * additems添加多行

* QTreeWidget 树控件

  ```c++
  MainWindow::MainWindow(QWidget *parent)
      : QMainWindow(parent)
      , ui(new Ui::MainWindow)
  {
      ui->setupUi(this);
  
      ui->treeWidget->setHeaderLabels(QStringList() << "hero" << "intruduction");
      QTreeWidgetItem *item = new QTreeWidgetItem(QStringList() << "力量");
      //加载顶层节点
      ui->treeWidget->addTopLevelItem(item);
      //追加子节点
      QTreeWidgetItem *l1 = new QTreeWidgetItem(QStringList() << "pig" << "好吃懒做");
      item->addChild(l1);
  }
  ```

* TableWidget 表格控件

  ```c++
  MainWindow::MainWindow(QWidget *parent)
      : QMainWindow(parent)
      , ui(new Ui::MainWindow)
  {
      ui->setupUi(this);
  
      //设置列
      ui->tableWidget->setColumnCount(3);
      //设置水平表头
      ui->tableWidget->setHorizontalHeaderLabels(QStringList() << "name" << "gender" << "age");
      //设置行数
      ui->tableWidget->setRowCount((5));
      //设置正文
      ui->tableWidget->setItem(0, 0,new QTableWidgetItem("亚瑟"));
  }
  ```

* 下拉框

  ```c++
  ui->comboBox->addItem("A");
  ui->comboBox->addItem("B");
  //点击按钮选中B
  connect(ui->btn_select, &QPushButton::clicked, [=]{
      //ui->comboBox->setCurrentIndex(1);
      ui->comboBox->setCurrentText("B");
  })
  ```

* QLabel 显示图片

* Qlabel显示gif动图(movie->start()播放)



##### 自定义控件

* 添加新文件 - Qt - 设计师界面类
* ui中设计控件
* Widget中使用自定义控件，拖拽一个Widget，提升为，添加，提升



##### 鼠标事件

* 鼠标进入事件 enterEvent
* 鼠标离开事件 leaveEvent
* 鼠标按下 mousePressEvent(QMouseEvent ev)
* 鼠标释放 mouseReleaseEvent
* 鼠标移动 mouseMoveEvent
* ev->x()x坐标 ev->y()y坐标
* ev->button()判断所有按键
* ev->buttons()判断组合键
* 格式化字符串 QString("%1  %2").arg(111).arg(222)
* 设置鼠标追踪setMouseTracking(true)



##### 定时器1

* 利用时间void timerEvent(QTimerEvent *ev)
* 启动定时器 startTimer(1000) 毫秒
* timerEvent的返回值是定时器的唯一标识，可以和ev->timerid做比较

##### 定时器2

* 利用定时器类Qtimer
* 创建定时器对象QTimer *timer = new QTimer(this)
* 启动定时器timer->start(毫秒)
* 每隔一定毫秒，发送信号 timerout 进行监听
* 暂停 timer->stop

##### event事件

* 用于事件分发
* 也可以做拦截操作，但不建议
* bool event(QEvent *e);
* 返回值如果是true代表用户处理这个时间，不向下分发
* e->type() == 鼠标按下

##### 事件过滤器

* 在程序将事件分发到事件发生器前，利用过滤器做拦截
* * 给控件安装事件过滤器
  * 重写eventFilter函数(obj, ev)

##### OPainter绘图

* 绘图事件 void painEvent()
* 声明画家对象QPainter painter(this) this执行绘图设备
* 画线，圆，矩形，文字
* 设置画笔OPen 设置画笔宽度、风格
* 设置画刷 QBrush设置画刷风格

##### QPainter高级设置

* 抗锯齿 painter.setRenderHint(QPainter::Antialiasing);
* 移动画家
* * painter.translate(100, 0)
  * save保存状态  restore还原状态
* 手动调用绘图事件 update
* 利用画家画图片  painter.drawPixmap(x, y, QPixmap(""))
* 



##### 绘图设备

##### 读写文件

```c++
    connect(ui->pushButton, &QPushButton::clicked, [=]{
        QString path = QFileDialog::getOpenFileName(this, "open");
        ui->lineEdit->setText(path);
        QFile file(path);//默认支持utf-8
        file.open(QIODevice::ReadOnly);
        QByteArray array = file.readAll();
        //编码格式类
        QTextCodec *codec = QTextCodec::codecForName("gbk");
        ui->textEdit->setText(codec->toUnicode(array));
        file.close();
        
        file.open(QIODevice::Append);//追加方式写
        file.write("");
        file.close();
    });
```



##### 文件信息读取

```c++
 //文件信息
QFileInfo info(path);
qDebug() << info.size() << info.suffix() << info.fileName();//大小 后缀 名字
```



