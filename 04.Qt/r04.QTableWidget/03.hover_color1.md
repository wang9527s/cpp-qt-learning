```cpp
#ifndef TABLE_H
#define TABLE_H

#include <QWidget>
#include <QTableView>
#include <QTableWidget>
#include <QVBoxLayout>
#include <QScrollBar>
#include <QPushButton>
#include <QLabel>
#include <QDebug>

#include <QHeaderView>
#include <QColor>
#include <QPushButton>
#include <QVBoxLayout>

class SWTableWidget : public QTableWidget
{
    Q_OBJECT
public:
    explicit SWTableWidget(QWidget *parent = nullptr)
    {
        m_transparentColor = QColor(0x00,0xff,0x00,0x00);//透明颜色
        m_preRow = -1;
        m_color1=QColor(23,81,93);
        m_color1=QColor(33,97,107);

        initTable(this);

        /*******************本段代码只是测试用，真正使用时需注释掉********************************/
        this->setRowCount(10);   //设置行数为10
        this->setColumnCount(5); //设置列数为5

        for(int i = 0; i < 10; i ++)
        {
            for(int j = 0; j < 5; j ++)
            {
                if (j==1)
                {
                    this->setCellWidget(i,j,buttonItem("qafq"));
                    continue;
                }
                QTableWidgetItem *item = new QTableWidgetItem();
                this->setItem(i,j,item);
            }
        }
    }
private:
    QColor m_transparentColor;//存储item之前的颜色，这里是透明的，默认设置为透明
    QColor m_color1,m_color2;
    int m_preRow;// 鼠标移动过的上一行的行号

    void initTable(QTableWidget * table)
    {
        table->verticalHeader()->setVisible(false);  //隐藏列表头;
        table->horizontalHeader()->setVisible(false); 

        //表格主题样式设置
        table->setFrameShape(QFrame::NoFrame); //设置无边框;
        table->setShowGrid(false); //设置不显示格子线;
        table->setEditTriggers(QAbstractItemView::NoEditTriggers);    //单元格禁止编辑
        table->setSelectionMode(QAbstractItemView::NoSelection);
        table->setAlternatingRowColors(true);        //设置每隔一行，背景色不同
        //整行选中
//         table->setSelectionMode(QAbstractItemView::SingleSelection);
//         table->setSelectionBehavior(QAbstractItemView::SelectRows);

        table->setStyleSheet(
            QString("QTableWidget{color:white;background:rgb(23,81,93);alternate-background-color:rgb(33,97,107);}")
            /* 和整行选择配合，修改整行选中的颜色  */
            //+QString("QTableWidget{selection-background-color:lightblue;}")
            //+QString("QTableWidget::item:selected { background-color: rgb(255, 0, 0);}")
            /* 单元格悬浮变色 */
            //+QString("QTableWidget::item:hover { background-color: rgb(255, 0, 0);}")
            );

        //表头样式
        table->horizontalHeader()->setStyleSheet(
            ""
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
            //还原上一行颜色
            QTableWidgetItem *item = this->item(m_preRow, 0);
            if (item != 0)
            {
                this->setRowColor(m_preRow, m_transparentColor);
            }

            //设置当前行的颜色
            item = this->item(row, column);
            if (item != 0 && !item->isSelected())
            {
                this->setRowColor(row, QColor(193,210,240),true);
            }

            //设置行的索引
            m_preRow = row;
        });
    }
    void setRowColor(int row, QColor color,bool isEnter=false)
    {
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
                            QString("background:rgb(%1,%2,%3);").arg(193).arg(210).arg(240)+m_ss
                            );
                        continue;
                    }

                    if(row%2!=1)
                    {
                        widget->setStyleSheet(
                            QString("background:rgb(%1,%2,%3);").arg(23).arg(81).arg(93)+m_ss
                            );
                    }
                    else
                        //widget->setStyleSheet("QWidget{border-image:url(:/Table/Resources/dark.png);}"+m_ss);
                        widget->setStyleSheet(
                        QString("background:rgb(%1,%2,%3);").arg(33).arg(97).arg(107)+m_ss
                        );


                }
            }
        }
    }

    QWidget * buttonItem(QString content)
    {
        QWidget * widget=new QWidget;
        QVBoxLayout *layout=new QVBoxLayout(widget);
        QPushButton * btn=new QPushButton;
        btn->setFlat(true);
        btn->setText(content);
        layout->addWidget(btn);    
        btn->setIcon(QIcon(":/Table/Resources/CheckBox_checked_2.png"));

        m_ss="QPushButton{border:none;}";

        widget->setStyleSheet(
            m_ss);
        return widget;
    }
    QString m_ss;
};


#endif // TABLE_H

```