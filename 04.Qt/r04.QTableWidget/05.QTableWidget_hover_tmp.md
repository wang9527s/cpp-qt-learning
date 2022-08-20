

```cpp
class Table : public QWidget
{
    Q_OBJECT

public:
    Table(QWidget *parent = 0)
    {
        ui.setupUi(this);
        m_pTableWidget=new QTableWidget;
        initTable(m_pTableWidget);

        //布局
        QVBoxLayout *pLayout=new QVBoxLayout(this);
        pLayout->addWidget(m_pTableWidget);

        //鼠标悬停
        m_pTableWidget->setMouseTracking(true);
        connect(m_pTableWidget,&QTableWidget::cellEntered,
            [=](int row,int col)
        {
            /* 
                setCellWidget的item无法触发
                setItem的item可以触发
            */
            qDebug()<<row<<", "<<col;
        });
        connect(m_pTableWidget,&QTableWidget::cellClicked,
            [=](int row,int col)
        {
            qDebug()<<row<<", "<<col;
        });
        addRow(QStringLiteral("麦克疯狂"),m_pTableWidget);
        addRow(QStringLiteral("阿发给他"),m_pTableWidget);
        addRow(QStringLiteral("效果"),m_pTableWidget);
    };

    void initTable(QTableWidget *table)
    {
        table->setColumnCount(4);

        table->verticalHeader()->setVisible(false);  //隐藏列表头;
        table->horizontalHeader()->setVisible(false); 

        //表格主题样式设置
        table->setFrameShape(QFrame::NoFrame); //设置无边框;
        table->setShowGrid(false); //设置不显示格子线;
        table->setEditTriggers(QAbstractItemView::NoEditTriggers);    //单元格禁止编辑
        table->setSelectionMode(QAbstractItemView::NoSelection);
        table->setAlternatingRowColors(true);        //设置每隔一行，背景色不同
        //整行选中
        table->setSelectionMode(QAbstractItemView::SingleSelection);
        table->setSelectionBehavior(QAbstractItemView::SelectRows);

        table->setStyleSheet(
            QString("QTableWidget{color:white;background:rgb(23,81,93);alternate-background-color:rgb(33,97,107);}")
            /* 和整行选择配合，修改整行选中的颜色  */
            //+QString("QTableWidget::item:selected { background-color: rgb(255, 0, 0);}")
            +QString("QTableWidget::item:hover { background-color: rgb(255, 0, 0);}")
            );

        //表头样式
        table->horizontalHeader()->setStyleSheet(
            "QHeaderView::section{background-color: qlineargradient(spread:pad, x1:0, y1:1, x2:0, y2:0, stop:0 rgba(2,63,74, 255), stop:1 rgba(12,75,90, 255));"
            "border:0px;"
            "border-left:1px solid qlineargradient(spread:pad, x1:0, y1:0, x2:0, y2:1, stop:0 rgba(255, 255, 255, 0), stop:0.227273 rgba(255, 255, 255, 0), stop:0.238636 rgba(255, 255, 255, 0.2), stop:0.784091 rgba(255, 255, 255, 0.2), stop:0.795455 rgba(255, 255, 255, 0), stop:1 rgba(255, 255, 255, 0));}"
            );

        //滚动条样式
        table->verticalScrollBar()->setStyleSheet(
            QString("QScrollBar:vertical{width:10px;margin:0px,0px,0px,0px;padding-top:0px;padding-bottom:0px;}")
            +QString("QScrollBar::sub-line:vertical,QScrollBar::add-line:vertical{height:0px;background:transparent ;}")//上下箭头
            +QString("QScrollBar::handle:vertical{width:10px;background-color:rgba(24, 150, 224, .5);border-radius:5px;min-height:20px;}")    //滑块
            +QString("QScrollBar::add-page:vertical,QScrollBar::sub-page:vertical{border-image:url(:/icon/ScrollBar_vertical .png)}")        //滚动条
            );
    }

    void addRow(QString content,QTableWidget *table)
    {
        int curIndex=table->rowCount();
        table->setRowCount(curIndex + 1);
        table->setCellWidget(curIndex, 1, creatItemLabel(QStringLiteral("QLabel文字")));
        //文件名
        table->setItem(curIndex, 2, new QTableWidgetItem(content));
        //删除
        QString css("QPushButton:hover{image:url(:/Table/Resources/delBtn.png);}QPushButton{border:none;}");
        table->setCellWidget(curIndex, 3, createItem(css));
    }
    QWidget* createItem(QString css)
    {
        QWidget * widget=new QWidget;
        widget->setStyleSheet("background-color:transparent ");
        QVBoxLayout *layout=new QVBoxLayout(widget);
        QPushButton *btn=new QPushButton;
        //btn->setAttribute(Qt::WA_TranslucentBackground);    //用不着
        btn->setStyleSheet(css);
        layout->addWidget(btn);
        return widget;
    }
    QWidget *creatItemLabel(QString content)
    {
        QWidget * widget=new QWidget;
        //widget->setAttribute(Qt::WA_TranslucentBackground);    //会出现widget上有控件的部分，颜色不透明（好像是QTableWidget的底色）
        widget->setStyleSheet("background-color:transparent ");
        QVBoxLayout *layout=new QVBoxLayout(widget);
        QLabel * pLabel=new QLabel;
        pLabel->setText(content);
        layout->addWidget(pLabel);
        return widget;
    }
    
    void clearTable(QTableWidget * table)
    {
        table->clearContents();
        table->setRowCount(0);
    }
    ~Table(){};
protected:
    void resizeEvent(QResizeEvent *e)
    {
        m_pTableWidget->horizontalHeader()->setSectionResizeMode(QHeaderView::Interactive);
        int wid=m_pTableWidget->size().width()/100;
        m_pTableWidget->setColumnWidth(0,wid*5);//空白
        m_pTableWidget->setColumnWidth(1,wid*30);
        m_pTableWidget->setColumnWidth(2,wid*50);
        m_pTableWidget->setColumnWidth(3,wid*15);
        m_pTableWidget->horizontalHeader()->setStretchLastSection(true);//使行列头自适应宽度，所有列平均分来填充空白部分

        return QWidget::resizeEvent(e);
    }

private:
    Ui::TableClass ui;
//    QTableView *m_pTableView;
    QTableWidget *m_pTableWidget;
};
```