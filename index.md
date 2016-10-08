## Qt学习笔记

---

 **编程时遇到不清楚的都可以随时查阅帮助文档，一定要学会用帮助文档，而不是屁大点事就到处瞎问。**

----

## 信号与槽函数

###  1.元对象系统###

- 信号，即为触发的动作等传入的信号，如clicked()

- 槽函数，即为对应需要触发的函数

  - Qt 的关键字 slots 就是槽函数声明段落的标志，槽函数声明段落可以是 private、protected 或者 public 类型的

  - Qt 的 弹窗类函数包含在QMessageBox类中，使用需要包含（同名）头文件。

  - QMessageBox 类提供了静态公有函数，里面预定义好了现成的消息框，只需把参数传给它，就会自动弹窗，声明如下

    ```c++
    StandardButton QMessageBox::information(QWidget * parent, const QString & title, const QString & text, StandardButtons buttons = Ok, StandardButton defaultButton = NoButton)
    ```

    ​

    ​

    ​

- connect函数，将两者关联，，声明如下：

  - 旧式：

    ```c++
    QMetaObject::Connection QObject::connect(const QObject * sender, const char * signal, const QObject * receiver, const char * method, Qt::ConnectionType type = Qt::AutoConnection)
    ```

    头两个参数为源头对象和信号，后两个参数为接受对象和槽函数，例如：

    ```c++
    connect(ui->pushButton, SIGNAL(clicked()), this, SLOT(FoodIsComing()));
    ```

    但是其参数都是字符串，编译时无法检查，实际运行才能检测出错误，因此qt5引入新式的connect函数

  - 新式：

    ```c++
    QMetaObject::Connection QObject::connect(const QObject * sender, PointerToMemberFunction signal, const QObject * receiver, PointerToMemberFunction method, Qt::ConnectionType type = Qt::AutoConnection)
    ```

    ​

    同样头两个参数为源头对象和信号，后两个参数为接受对象和槽函数

    > 与旧语法句式区别就在于 signal 和 method （槽函数），新句式用的是 PointerToMemberFunction ，这个类型名称是不存在的，只是在文档里面显示，方便程序员看的，实际使用的是模板函数。具体模板函数定义比较复杂，比较关键的就是两个函数参数类型需要一致，或者信号声 明时的参数更多更广。因为信号本身是不干活的，它多点参数无所谓，但必须保证槽函数运行时需要的参数是一定有的。
    >
    > 新的语法句式应用到第一个例子就是如下面这样：
    >
    > ```c++
    >  connect(ui->pushButton, &QPushButton::clicked, this, &Widget::FoodIsComing);
    > ```
    >
    > 这里第二个位置是一个函数指针形式，第四个位置也是一个函数指针形式，信号里声明的参数和槽函数声明是一致的，所以关联是匹配的。

- connect  函数 可以实现一对多关联也可以实现多对一关联

  - 一对多，即一个信号触发多个槽函数。实际操作为使用多个connect函数，元对象和信号一样，接受对象和槽函数不都一致。

    例如：    //关联信号到槽函数    

    ```c++
        //接收端是标签控件
        connect(ui->lineEdit, SIGNAL(textEdited(QString)), ui->label, SLOT(setText(QString)));
        //接收端是文本浏览控件
        connect(ui->lineEdit, SIGNAL(textEdited(QString)), ui->textBrowser, SLOT(setText(QString)));
    ```

  - 多对一，即多个不同函数皆可触发同一个槽函数。实际操作为使用多个connect函数，元对象和信号不完全一样，接受对象和槽函数一致。如需要对不同信号进行不同处理，可在槽函数中根据源头对象名来处理，`this->sender()->objectName()`

    例如：    //三个按钮的信号都关联到 FoodIsComing 槽函数

    ```c++
        connect(ui->pushButtonAnderson, SIGNAL(clicked()), this, SLOT(FoodIsComing()));
        connect(ui->pushButtonBruce, SIGNAL(clicked()), this, SLOT(FoodIsComing()));
        connect(ui->pushButtonCastiel, SIGNAL(clicked()), this, SLOT(FoodIsComing()));
        
        void Widget::FoodIsComing()
        { //获取信号源头对象的名称 
          QString strObjectSrc = this->sender()->objectName(); 
          //将要显示的消息
          QString strMsg; 
          //判断是哪个按钮发的信号
          if( "pushButtonAnderson" == strObjectSrc ) 
        { strMsg = tr("Hello Anderson! Your food is coming!");}
      else if( "pushButtonBruce" == strObjectSrc )
          { strMsg = tr("Hello Bruce! Your food is coming!");}
      else if( "pushButtonCastiel" == strObjectSrc )
            { strMsg = tr("Hello Castiel! Your food is coming!");  }
      else 
            {
              //do nothing
                return;  
             }
       //显示送餐消息
        QMessageBox::information(this, tr("Food"), strMsg);
    }
    ```

    ​

- 解除关联，disconnect函数，使用方法与connect函数一致，参数一致

- tips：

  禁用某个按钮可以使用`setEnabled(bool)`函数，例如:  

  ```C++
  ui->pushButton->setEnabled(false);   
  ui->pushButton->setEnabled(true);  
  ```

  ​

### 2.自定义信号和槽

  