---
layout: post
title: 【Qt】複数のウィジェットを含むカスタムウィジェットを作る方法
tags:
  - C++
  - hatenablog
---

今回は、Qtについての備忘録です。
前回と同様、学習のアウトプット目的で書いています。
ご指摘等ありましたら、コメントに書いていただけると幸いです。

動作環境は以下の通りです。
<b>【OS】</b>Windows10 64bit
<b>【CPU】</b>Intel core i5-4590
<b>【Qt】</b>5.11.2 64-bit for Desktop (MSVC 2017)
<b>【Qt Creator】</b>4.7.1 (Community)

では、本題です。
[:contents]


*　カスタムウィジェットとは <hr>
Qtには既存のウィジェットに独自の機能を持たせた
&nbsp;&nbsp;&nbsp;&nbsp;<b>カスタムウィジェット</b>
という機能があります。

基本的にカスタムウィジェットは、
<u>既存のウィジェットから</u>派生したクラスを登録することで扱えるようになります。


登録方法としては、二つ方法があります。

<span style="font-size: 100%"> 1. <b>格上げ</b></span>
<span style="font-size: 100%"> 2. <b>Custom Widget Plugin</b></span>

<span style="font-size: 100%">1は</span>比較的簡単で、環境もそのままで行えます。

<span style="font-size: 100%">2は</span>難易度としては高いですが、より細かいカスタムが行えます。
しかしWindowsの場合、環境構築が大変です。

Pluginに関して、詳しくは以下をご覧ください。
<span style="font-size: 100%">・[https://qiita.com/argama147/items/2d280a292a8c76ab9b5f:title]</span>
<span style="font-size: 100%">・[https://doc.qt.io/qt-5/qtdesigner-customwidgetplugin-example.html:title]</span>
<span style="font-size: 100%">・[https://stackoverflow.com/questions/16486366/how-do-i-create-a-custom-widget-plugin-for-qt-designer-with-cmake-and-visual:title]</span>

<span style="font-size: 100%"><span style="color: #ff5252">今回は、1の<b>格上げ</b>を用いることにします。</span></span>

* 格上げを用いてカスタムウィジェットを登録する<hr>
<span style="color: #ff5252">格上げには<b>Qt Creator</b>を用いて行います。</span>

まずは、カスタムウィジェットのクラスを作ります。

ここでつくるカスタムウィジェットは、
できるだけそのウィジェット内で完結するようにしたいので、
<span style="font-size: 100%">ボタンが押されている間、ボタンにcliked!!と表示</span>
といった簡単なものにしたいと思います。

<span style="color: #ff5252">プロジェクトは、Qt ウィジェットアプリケーションです。</span>
新規プロジェクトに以下のヘッダファイルを追加しています。

コードはこちら。
>|cpp|

//custombutton.hpp
#ifndef CUSTOMBUTTON_HPP
#define CUSTOMBUTTON_HPP

#include <QPushButton>

class CustomButton : public QPushButton
{
    Q_OBJECT

public:
    CustomButton( QWidget* parent = nullptr ) :
        QPushButton( parent )
    {
        connect( this, SIGNAL( pressed() ), this, SLOT( ChangeText() ) ) ;
        connect( this, SIGNAL( released() ), this, SLOT( ClearText() ) ) ;
    }

private slots:
    void ChangeText() {
        setText( "clicked!!" ) ;
    }
    void ClearText() {
        setText( "" ) ;
    }
};

#endif // CUSTOMBUTTON_HPP

||<
<span style="font-size: 100%">QPushButtonが押されたとき → ChangeText関数
QPushButtonが離されたとき → ClearText関数</span>
といった具合にconnectしているだけです。

シグナルはスロットでキャッチできます。

次にこのクラスを登録してみましょう。
まずはプロジェクトの<b>.uiファイル</b>を開きます。
<b>Qt Designer</b>でもよいです。

次に<b>基本クラスにあたるウィジェット</b>を置き、<b>その上で右クリック</b>をします。
ここで<span style="color: #ff5252"><b>格上げ先を指定...</b></span>を選択。

[f:id:pit-ray:20181105002553j:plain]

表示されたダイアログに
カスタムウィジェットの<b>基本クラス</b>
カスタムウィジェットの<b>クラス名</b>
カスタムウィジェットクラスがある<b>ヘッダファイル名</b>
を入力し、追加します。
[f:id:pit-ray:20181105002556j:plain]

最後に選択し、格上げすればokです。
[f:id:pit-ray:20181105002559j:plain]

これで実行すると以下のような動作になります。
ここで注意したいのは、これは
<b><span style="font-size: 100%">ウィジェットの機能としての動作</span></b>
であるということです。
[f:id:pit-ray:20181105011848g:plain]

このように、派生クラスを登録することで、任意の機能を持たせたウィジェットを作れます。

ここで元のウィジェットが複数存在する場合は、どうしたらいいのでしょうか。
正直なところ、一つのウィジェットの中に複数のウィジェットがあるというのは、設計的に賛否がありそうです。
しかし、極めて密接に関係していて、そのウィジェットが複数ある場合、
同じクラス内に複数のウィジェットを配置して、内部でデータのやり取りをしたほうがよさそうです。その方がオブジェクト指向のカプセル化を実現できます。


*  複数のウィジェットを扱うカスタムウィジェットをつくる<hr>

結論からいうと、
<span style="color: #ff5252"><b><span style="font-size: 100%">QWidgetを継承する</span></b></span>ことで内部に複数のウィジェットを配置できます。
内部のウィジェットはポインタとして作成し、コード上でレイアウト等を決めます。

Qtのウィジェットは大抵、<b>QWidgetクラス</b>を継承しています。
<b>QWidget</b>は例えるならば、画用紙のようなものです。

実際に<b>.uiファイル</b>や、<b>Qt Designer</b>で確認してみるとわかりますが、
<b>Widgetオブジェクト</b>は透明な枠だけのものです。

[f:id:pit-ray:20181105203950j:plain]

早速ですが、<b>QPushButton</b>と<b>QLabel</b>を用いて、カウンターを作りたいと思います。
ヘッダは以下のようになります。

>|cpp|

//counter.hpp
#ifndef COUNTER_HPP
#define COUNTER_HPP

#include <QWidget>
#include <QPushButton>
#include <QLabel>
#include <QVBoxLayout>
#include <QString>

class Counter : public QWidget
{
    Q_OBJECT
public:
    explicit Counter( QWidget* parent = nullptr ) :
        QWidget( parent ),
        button( new QPushButton( this ) ),
        label( new QLabel( this ) ),
        counter( 0 )
    {
        InitLayout() ;

        //buttonからのclickedシグナルをこのクラスのIncrement()に接続
        connect( button, SIGNAL( clicked() ), this, SLOT( Increment() ) ) ;
    }

    ~Counter() {
        delete button ;
        delete label ;
    }

private:
    QPushButton* button ;
    QLabel*      label ;
    int counter ;

    void UpdateLabel() {
        label->setText( QString::number( counter ) ) ;
    }

    void InitLayout() {
        //縦方向のレイアウト生成
        QVBoxLayout* layout = new QVBoxLayout( this ) ;

        layout->addWidget( button ) ;
        layout->addWidget( label ) ;

        //ウィジェットを中央揃えにする
        layout->setAlignment( button, Qt::AlignCenter ) ;
        layout->setAlignment( label, Qt::AlignCenter ) ;

        setLayout( layout ) ;

        //ウィジェットのサイズを設定。
        //サイズはQt Designer, Qt Creatorのプロパティから確認できる。
        button->setMinimumSize( 180, 60 ) ;
        button->setText( "Increment" ) ;

        label->setMinimumSize( 180, 60 ) ;
        label->setAlignment( Qt::AlignCenter ) ;
        label->setFont( QFont( "Arial", 20, QFont::Bold ) ) ;
        UpdateLabel() ;
    }

private slots:
    void Increment() {
        counter ++ ;
        UpdateLabel() ;
    }
};

#endif // COUNTER_HPP

||<

このクラスは、基本クラスが<b>QWidget</b>であるので、
格上げ元は<b>Widgetオブジェクト</b>にします。

ここで注意したいのは、レイアウトです。
この場合、<b>配置はソースコード上で設定する</b>必要があります。
レイアウトには様々な種類があり、目的に合ったものを選んでください。
<span style="font-size: 100%">・[https://doc.qt.io/qt-5/layout.html:title]</span>
<span style="font-size: 100%">・[https://doc.qt.io/qt-5/qt.html#AlignmentFlag-enum:title]</span>

<b>Layoutオブジェクト</b>をつくり、<b>setLayout</b>で渡します。
ここでは暗黙的に<b>delete</b>しています。
privateメンバとして保持し、コンストラクタで<b>new</b>, デストラクタで明示的に<b>delete</b>でもよいです。
<b>setAlignment</b>は第一引数は、<b><span style="color: #ff5252">内部ウィジェットの</span>ポインタ</b>です。
<span style="font-size: 100%">・[https://doc.qt.io/archives/qt-4.8/qlayout.html#setAlignment:title]</span>

以上のようにすることでこのように動作します。
[f:id:pit-ray:20181105203012g:plain]

このように<b>QWidget</b>を用いることで、
複数のウィジェットを組み合わせたカスタムウィジェットを作ることができます。


最初にも申しましたが、ご指摘は歓迎いたします。

今回の記事が参考になれば幸いです。
ご閲覧ありがとうございました。

* 【参考文献】
（上で紹介したサイトは省略させていただきます）
<span style="font-size: 100%">・[https://stackoverflow.com/questions/8366617/combine-multiple-widgets-into-one-in-qt:title]</span>
<span style="font-size: 100%">・[https://blog.qt.io/jp/2011/07/21/widget-propmotion/:title]</span>
