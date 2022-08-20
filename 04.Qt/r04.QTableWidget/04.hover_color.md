
#### WidgetItem
```cpp
#include <QTableWidget>
#include <QHeaderView>
#include <QTableView>
#include <QColor>
#include <QResizeEvent>
#include <QVBoxLayout>
#include <QScrollBar>
#include <QTableView>
#include <QPushButton>
#include <QDebug>


class TableItemBtn : public QWidget
{
    Q_OBJECT

public:
    TableItemBtn(QString serPathName,int i,int j)
        :m_row(i)
        ,m_col(j)
    {
        m_btn=new QPushButton;
        m_btn->setFlat(true);
        m_btn->setStyleSheet("border:none;border-radius:0px;");
        m_btn->setSizePolicy(QSizePolicy::Expanding,QSizePolicy::Expanding);
        m_btn->setIcon(QIcon(":/Table/Resources/CheckBox_checked_2.png"));

        this->setStyleSheet("QWidget{background:transparent;}");

        QVBoxLayout *pLayout=new QVBoxLayout(this);
        pLayout->addWidget(m_btn);
        pLayout->setMargin(0);
    }
    void changeState()
    {
    
    }
signals:
    void sigEnter(int,int);
protected:
    void enterEvent(QEvent *e)
    {
        qDebug()<<m_row<<m_col;
        emit sigEnter(m_row,m_col);
    }
private:
    ~TableItemBtn(){}
    QPushButton *m_btn;
    int m_row,m_col;
};
```


#### TableWidget
```cpp
class Table : public QTableWidget
{
    Q_OBJECT

public:
    Table(QWidget *parent=0);
    ~Table();
    void addRow(QString str);
public slots:
    void enterSlot(int row,int column);

private:
    void initTable(QTableWidget * table);
    void setRowColor(int row, QColor color,bool isEnter=false);

    QColor m_transparentColor;//存储item之前的颜色，这里是透明的，默认设置为透明
    QColor m_hoverColor, m_BgColor,m_alternateBgColor;
    int m_preRow;// 鼠标移动过的上一行的行号
};
```
```cpp
Table::Table(QWidget *parent)
    : QTableWidget(parent)
    ,m_transparentColor(QColor(0x00,0xff,0x00,0x00))//透明颜色
    ,m_preRow(-1)
    ,m_hoverColor(QColor(255,0,0))
    ,m_BgColor(QColor(23,81,93))
    ,m_alternateBgColor(QColor(33,97,107))
{
    
    initTable(this);
    setColumnCount(5);
}

Table::~Table()
{

}

void Table::addRow(QString str)
{
    int row=rowCount();
    setRowCount(row + 1);    

    for(int j = 0; j < 5; j ++)
    {
        if (j==1)
        {
            TableItemBtn * widgetItem=new TableItemBtn("q000000000000000q", row, j);
            QObject::connect(widgetItem,&TableItemBtn::sigEnter,
                [=](int row,int col)
            {
                enterSlot(row,col);
            });
            setCellWidget(row,j,widgetItem);
            continue;
        }
        QTableWidgetItem *item = new QTableWidgetItem();
        setItem(row,j,item);
    }

}

void Table::enterSlot(int row,int column)
{
    //还原上一行颜色
    this->setRowColor(m_preRow, m_transparentColor);

    //设置当前行的颜色
    this->setRowColor(row, m_hoverColor,true);

    //设置行的索引
    m_preRow = row;
}

void Table::initTable(QTableWidget * table)
{
    table->verticalHeader()->setVisible(false);  //隐藏列表头;
    //table->horizontalHeader()->setVisible(false); 

    //表格主题样式设置
    table->setFrameShape(QFrame::NoFrame); //设置无边框;
    table->setShowGrid(false); //设置不显示格子线;
    table->setEditTriggers(QAbstractItemView::NoEditTriggers);    //单元格禁止编辑
    table->setAlternatingRowColors(true);        //设置每隔一行，背景色不同
    //整行选中
//     table->setSelectionMode(QAbstractItemView::SingleSelection);
//     table->setSelectionBehavior(QAbstractItemView::SelectRows);

    table->setStyleSheet(
        QString("QTableWidget{color:white;background:rgba(%1,%2,%3);alternate-background-color:rgba(%4,%5,%6);}")
            .arg(m_BgColor.red()).arg(m_BgColor.green()).arg(m_BgColor.blue()).arg(m_alternateBgColor.red()).arg(m_alternateBgColor.green()).arg(m_alternateBgColor.blue())
        );

    //表头样式
    table->horizontalHeader()->setSectionResizeMode(QHeaderView::Interactive); 
    table->horizontalHeader()->setStretchLastSection(true);
    table->horizontalHeader()->setStyleSheet(""
        "QHeaderView::section{background-color: rgba(28, 107, 157, .2);font-color:rgba(18,150,219,1);font-size:14px;}"
        "border:none;"
        );

    //滚动条样式
    table->verticalScrollBar()->setStyleSheet(
        QString("QScrollBar:vertical{width:10px;margin:0px,0px,0px,0px;padding-top:0px;padding-bottom:0px;}")
        +QString("QScrollBar::sub-line:vertical,QScrollBar::add-line:vertical{height:0px;background:transparent ;}")//上下箭头
        +QString("QScrollBar::handle:vertical{width:10px;background-color:rgba(24, 150, 224, .5);border-radius:5px;min-height:20px;}")    //滑块
        +QString("QScrollBar::add-page:vertical,QScrollBar::sub-page:vertical{border-image:url(:/icon/ScrollBar_vertical .png)}")        //滚动条
        );

    //鼠标悬浮
    table->setMouseTracking(true);//设置可捕获鼠标移动事件，很重要
    connect(this,&QTableWidget::cellEntered,
        [=](int row,int column)
    {
        enterSlot(row,column);
    });
}

void Table::setRowColor(int row, QColor color,bool isEnter/*=false*/)
{
    if(row>=rowCount())return;

    for (int col=0; col<this->columnCount(); col++)
    {
        QTableWidgetItem *item = this->item(row, col);
        if(item)
        {
            item->setBackgroundColor(color);
        }
        else
        {
            QWidget *widget=this->cellWidget(row,col);
            if(widget)
            {
                if(isEnter)
                {
                    widget->setStyleSheet(
                        QString("background:rgb(%1,%2,%3);").arg(m_hoverColor.red()).arg(m_hoverColor.green()).arg(m_hoverColor.blue())
                        );
                    continue;
                }

                if(row%2==0)
                {
                    widget->setStyleSheet(
                        QString("background:rgb(%1,%2,%3);").arg(m_BgColor.red()).arg(m_BgColor.green()).arg(m_BgColor.blue())
                        );
                }
                else
                    widget->setStyleSheet(
                        QString("background:rgb(%1,%2,%3);").arg(m_alternateBgColor.red()).arg(m_alternateBgColor.green()).arg(m_alternateBgColor.blue())
                        );


            }
        }
    }
}

```


#### 使用
```cpp
Table w;

for(int i = 0; i < 10; i ++)
{
    w.addRow("asf");
}
w.show();
w.resize(500,300);
```