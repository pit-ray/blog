---
layout: post
title: タスクシステムライブラリ tasksyslib リファレンス
tags:
  - C++
  - C
  - hatenablog
---


C/C++対象のタスクシステムライブラリtasksyslibのリファレンスです。
<a href="https://github.com/pit-ray/tasksyslib"><span style="font-size: 150%">GitHubでダウンロードする</span></a>
<a href="https://www.pit-ray.com/entry/tasksyslib"><span style="font-size: 150%">規約等をみる</span></a>
<a name="contents">
[:contents]
</a>

<br>
<!--___________________________________________________________________________________________-->
<a name="1">
**<span style="color: #ff5252"> InitTaskSys関数</span>：タスクシステムの初期化<hr>
</a>

<b>宣言</b>
>|cpp|
TaskSystem* InitTaskSys(
    unsigned int max_task_num
) ;
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステムを初期化する</span>

<b>引数</b>
|<span style="color: #2196f3">max_task_num</span>|タスクシステムに登録できる任意の最大タスク数|

<b>戻り値</b>
|<span style="color: #2196f3">メモリアドレス</span>|成功|
|<span style="color: #2196f3">NULL</span>|失敗|

<b>解説</b>
タスクシステムの規模を引数により設定し、タスクシステムを構築します。
TaskSystemポインタ型のオブジェクトにこの関数を代入して呼び出します。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="2">
** <span style="color: #ff5252">RegisterNewTask関数</span>：タスクの登録<hr>
</a>

<b>宣言</b>
>|cpp|
Task* RegisterNewTask(
    TaskSystem *p_task_sys
) ;
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステムにタスクを登録する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|登録するタスクシステムのポインタ|

<b>戻り値</b>
|<span style="color: #2196f3">メモリアドレス</span>|成功|
|<span style="color: #2196f3">NULL</span>|失敗|

<b>解説</b>
タスクは<span style="color: #ff5252">デフォルトでは非アクティブ状態</span>で登録されています。
<a href="#4">IsActivateRegdTask関数</a>で有効化することができます。
Taskポインタ型のオブジェクトにこの関数を代入して呼び出します。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="3">
** <span style="color: #ff5252">SetTaskParameter関数</span>：タスクの設定<hr>
</a>

<span style="font-size: 150%"><b>宣言</b></span>　　
>|cpp|
void SetTaskParameter(
    Task* p_task,
    unsigned int priority,
    void ( *p_FuncAccessed ) ( Task* ),
    void* p_resource
) ;
||<
<b>概略</b>
<span style="color: #2196f3">タスク自体の設定をする</span>

<b>引数</b>
|<span style="color: #2196f3">p_task</span>|設定対象のタスクのポインタ|
|<span style="color: #2196f3">priority</span>|タスクの優先度|
|<span style="color: #2196f3">p_FuncAccessed</span>|タスクが呼び出す関数のポインタ|
|<span style="color: #2196f3">p_resource</span>|タスクに渡すデータへのポインタ|

<b>戻り値</b>
|なし|

<b>解説</b>
第一引数は、<a href="#2">RegisterNewTask関数</a>で登録済みのタスクを用いる。
第二引数は、unsigned intで扱える値(0 ～ 4,294,967,295)ならばなんでもよい。自由に決めてもらって構わない。
第三引数は、通常、関数自体は普通に定義して&を付けて関数を引数として渡す。<span style="color: #ff5252">関数はvoid型で引数がTask*型</span>でなければならない。
第四引数は、関数が外部のデータとやりとりするためのvoidポインタ。<span style="color: #ff5252">データを取り出すときには一度キャストしたほうが良い</span>。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="4">
** <span style="color: #ff5252">IsActivateRegdTask関数</span>：タスクを有効化<hr>
</a>

<b>宣言</b>
>|cpp|
bool IsActivateRegdTask(
    TaskSystem* p_task_sys,
    Task*       p_regd_task
) ;
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステムに登録済みのタスクをアクティブにする</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|登録しているタスクシステムのポインタ|
|<span style="color: #2196f3">p_task_sys</span>|アクティブにしたいタスクのポインタ|

<b>戻り値</b>
|<span style="color: #2196f3">true</span>|成功|
|<span style="color: #2196f3">false</span>|失敗|

<b>解説</b>
アクティブなタスクは処理対象になる。
注意すべき点としては必ず、登録後のタスクを用いること。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="5">
** <span style="color: #ff5252">IsDeactivateRegdTask関数</span>：タスクを無効化<hr>
</a>

<b>宣言</b>
>|cpp|
bool IsDeactivateRegdTask(
    TaskSystem* p_task_sys,
    Task*       p_regd_task
) ;
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステムに登録済みのタスクを非アクティブにする</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|登録しているタスクシステムのポインタ|
|<span style="color: #2196f3">p_regd_task</span>|非アクティブにしたいタスクのポインタ|

<b>戻り値</b>
|<span style="color: #2196f3">true</span>|成功|
|<span style="color: #2196f3">false</span>|失敗|

<b>解説</b>
非アクティブにしてもタスクシステムからは削除されない。
あくまで処理対象から外れるだけ。
<a href="#4">IsActivateRegdTask関数</a>と同様に登録済みのタスクを用いる。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="6">
** <span style="color: #ff5252">IsDelTask関数</span>：タスクを削除<hr>
</a>

<b>宣言</b>
>|cpp|
bool IsDelTask(
    TaskSystem* p_task_sys,
    Task*       p_regd_task
)
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステム上からタスクを完全削除する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|登録するタスクシステムのポインタ|
|<span style="color: #2196f3">p_regd_task</span>|削除したいタスクのポインタ|

<b>戻り値</b>
|<span style="color: #2196f3">true</span>|成功|
|<span style="color: #2196f3">false</span>|失敗|

<b>解説</b>
<a href="#2">RegisterNewTask関数</a>と対になる処理である。
タスクの削除を行うと、同時に非アクティブ化も行われるので処理対象から外れる。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="7">
** <span style="color: #ff5252">IsDelAllInactiveTask関数</span>：無効タスクを削除<hr>
</a>

<b>宣言</b>
>|cpp|
bool IsDelAllInactiveTask(
    TaskSystem* p_task_sys,
    Task*       p_regd_task
)
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステム上から非アクティブタスクのみ完全削除する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|登録するタスクシステムのポインタ|

<b>戻り値</b>
|<span style="color: #2196f3">true</span>|成功|
|<span style="color: #2196f3">false</span>|失敗|

<b>解説</b>
全ての非アクティブタスクに対して削除処理が行われる。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="8">
** <span style="color: #ff5252">IsCallActiveFunc関数</span>：タスクの実行<hr>
</a>

<b>宣言</b>
>|cpp|
bool IsCallActiveFunc(
    TaskSystem* p_task_sys
)
||<
<b>概略</b>
<span style="color: #2196f3">アクティブタスクの処理関数を優先度にしたがって呼び出す</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|対象のタスクシステム|

<b>戻り値</b>
|<span style="color: #2196f3">true</span>|成功|
|<span style="color: #2196f3">false</span>|失敗|

<b>解説</b>
タスクに付けた優先度に従って処理関数を呼び出す。
<span style="color: #ff5252">同じ優先度のタスクの場合、追加した順番に処理する。</span>
例えばもし同じ優先度のタスクA、タスクB、タスクCがあるとする。
TaskA = RegisterNewTask( task_sys ) ;
TaskB = RegisterNewTask( task_sys ) ;
TaskC = RegisterNewTask( task_sys ) ;
のようにした場合、A→B→Cの順に処理する。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="9">
** <span style="color: #ff5252">IsDeinitTaskSys関数</span>：タスクシステムの後始末<hr>
</a>

<b>宣言</b>
>|cpp|
bool IsDenitTaskSys(
    TaskSystem* p_task_sys
) ;
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステムが確保したメモリを開放する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|対象のタスクシステム|

<b>戻り値</b>
|<span style="color: #2196f3">true</span>|成功|
|<span style="color: #2196f3">false</span>|失敗|

<b>解説</b>
<a href="#1">InitTaskSys関数</a>と対になる関数。
<span style="color: #ff5252">メモリリークなどの問題につながる恐れがあるため、必ず呼び出すこと</span>。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="10">
** <span style="color: #ff5252">GetAllTaskSize関数</span>：全タスクの合計サイズを取得<hr>
</a>

<b>宣言</b>
>|cpp|
inline unsigned int GetAllTaskSize(
    TaskSystem* p_task_sys
) ;
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステムに登録された全てのタスクの合計サイズを取得する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|対象のタスクシステム|

<b>戻り値</b>
|<span style="color: #2196f3">全タスクの合計サイズ（バイト単位）</span>|

<b>解説</b>
inlineを用いてインライン関数化しているだけで、戻り値はunsingned int型。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="11">
** <span style="color: #ff5252">GetAllTaskNum関数</span>：全タスクの合計数を取得<hr>
</a>

<b>宣言</b>
>|cpp|
inline unsigned int GetAllTaskNum(
    TaskSystem* p_task_sys
) ;
||<
<b>概略</b>
<span style="color: #2196f3">タスクシステムに登録された全てのタスクの合計数を取得する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|対象のタスクシステム|

<b>戻り値</b>
|<span style="color: #2196f3">全タスクの合計数</span>|

<b>解説</b>
inlineを用いてインライン関数化しているだけで、戻り値はunsingned int型。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="12">
** <span style="color: #ff5252">GetActiveTaskSize関数</span> ：有効タスクの合計サイズを取得<hr>
</a>

<b>宣言</b>
>|cpp|
inline unsigned int GetActiveTaskSize(
    TaskSystem* p_task_sys
) ;
||<
<b>概略</b>
<span style="color: #2196f3">アクティブになっているタスクの合計サイズを取得する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|対象のタスクシステム|

<b>戻り値</b>
|<span style="color: #2196f3">アクティブタスクの合計サイズ（バイト単位）</span>|

<b>解説</b>
inlineを用いてインライン関数化しているだけで、戻り値はunsingned int型。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="13">
** <span style="color: #ff5252">GetActiveTaskNum関数</span>：有効タスクの合計数を取得<hr>
</a>

<b>宣言</b>
>|cpp|
inline unsigned int GetActiveTaskNum(
    TaskSystem* p_task_sys
) ;
||<
<b>概略</b>
<span style="color: #2196f3">アクティブになっているタスクの合計数を取得する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|対象のタスクシステム|

<b>戻り値</b>
|<span style="color: #2196f3">アクティブタスクの合計数</span>|

<b>解説</b>
inlineを用いてインライン関数化しているだけで、戻り値はunsingned int型。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
<a name="14">
** <span style="color: #ff5252">GetInactiveTaskSize関数</span> ：無効タスクの合計サイズを取得<hr>
</a>

<b>宣言</b>
>|cpp|
inline unsigned int GetInactiveTaskSize(
    TaskSystem* p_task_sys
) ;
||<
<b>概略</b>
<span style="color: #2196f3">非アクティブになっているタスクの合計サイズを取得する</span>

<b>引数</b>
|<span style="color: #2196f3">p_task_sys</span>|対象のタスクシステム|

<b>戻り値</b>
|<span style="color: #2196f3">非アクティブタスクの合計サイズ（バイト単位）</span>|

<b>解説</b>
inlineを用いてインライン関数化しているだけで、戻り値はunsingned int型。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>
<!--___________________________________________________________________________________________-->
** <span style="color: #ff5252">GetInactiveTaskNum関数</span>：無効タスクの合計数を取得

<span style="font-size: 150%"><b>宣言</b></span>
>|cpp|
inline unsigned int GetInactiveTaskNum(
    TaskSystem* p_task_sys
) ;
||<
<span style="font-size: 150%"><b>概略</b></span>
<span style="color: #2196f3">非アクティブになっているタスクの合計数を取得する</span>
<span style="font-size: 150%"><b>引数</b></span>
|<span style="color: #2196f3">p_task_sys</span>|対象のタスクシステム|
<span style="font-size: 150%"><b>戻り値</b></span>
|<span style="color: #2196f3">非アクティブタスクの合計数</span>|
<span style="font-size: 150%"><b>解説</b></span>
inlineを用いてインライン関数化しているだけで、戻り値はunsingned int型。

<a href="#example">実装例を見る</a>
<a href="#contents">目次に戻る</a>
<br>


<a name="example">
** 実装例<hr>
</a>
>|cpp|
#include <stdio.h>
#include "tasksyslib.h"

void PrintPlayerName( Task* player_t ) {
    char *str = ( char * ) player_t->p_resource ;
    printf( "%s\n", str ) ;
}

void PrintState( TaskSystem *p_task_sys ) {
    printf( "All Task Size      = %d byte\n", GetAllTaskSize( p_task_sys ) ) ;
    printf( "All Task Num       = %d\n", GetAllTaskNum( p_task_sys ) ) ;
    printf( "Active Task Size   = %d byte\n", GetActiveTaskSize( p_task_sys ) ) ;
    printf( "Active Task Num    = %d\n", GetActiveTaskNum( p_task_sys ) ) ;
    printf( "Inactive Task Size = %d byte\n", GetInactiveTaskSize( p_task_sys ) ) ;
    printf( "Inactive Task Num  = %d\n", GetInactiveTaskNum( p_task_sys ) ) ;
}

int main() {
    //タスクシステムの初期化
    TaskSystem* draw_ts = InitTaskSys( 100 ) ;

    //タスクの登録
    Task* playerA_t = RegisterNewTask( draw_ts ) ; 
    Task* playerB_t = RegisterNewTask( draw_ts ) ;
    Task* playerC_t = RegisterNewTask( draw_ts ) ;
    Task* playerD_t = RegisterNewTask( draw_ts ) ;

    //タスクの設定
    SetTaskParameter( playerA_t, 3, &PrintPlayerName, "John" ) ;
    SetTaskParameter( playerB_t, 2, &PrintPlayerName, "David" ) ;
    SetTaskParameter( playerC_t, 1, &PrintPlayerName, "Richard" ) ;
    SetTaskParameter( playerD_t, 1, &PrintPlayerName, "Robert" ) ;

    //タスクをアクティブにする
    if( !IsActivateRegdTask( draw_ts, playerA_t ) ) {
        return 1 ; //エラー処理
    }
    if( !IsActivateRegdTask( draw_ts, playerB_t ) ) {
        return 1 ;
    }
    if( !IsActivateRegdTask( draw_ts, playerC_t ) ) {
        return 1 ;
    }
    if( !IsActivateRegdTask( draw_ts, playerD_t ) ) {
        return 1 ;
    }

    //タスクを実行する
    if( !IsCallActiveFunc( draw_ts ) ) {
        return 1 ;
    }
    
    PrintState( draw_ts ) ;
    printf( " _ _ _ _ _ _ _ _ _ _ _\n" ) ;

//______________________________________________________________
    //タスクBを非アクティブ化
    if( !IsDeactivateRegdTask( draw_ts, playerB_t ) ) {
        return 1 ;
    }

    //タスクを実行する
    if( !IsCallActiveFunc( draw_ts ) ) {
        return 1 ;
    }
    
    PrintState( draw_ts ) ;
    printf( " _ _ _ _ _ _ _ _ _ _ _\n" ) ;

//______________________________________________________________
    //非アクティブタスク、つまりBを削除
    if( !IsDelAllInactiveTask( draw_ts ) ) {
        return 1 ;
    } 

    //タスクCを削除
    if( !IsDelTask( draw_ts, playerC_t ) ) {
        return 1 ;
    }

    //タスクを実行する
    if( !IsCallActiveFunc( draw_ts ) ) {
        return 1 ;
    }
    
    PrintState( draw_ts ) ;

//______________________________________________________________
    if( !IsDeinitTaskSys( draw_ts )  ) {
        return 1 ;
    }
    return 0 ; //正常終了
}
||<
<br>
<b><span style="font-size: 150%">実行結果</span></b>
>|cpp|
John
David
Richard
Robert
All Task Size      = 112 byte
All Task Num       = 4
Active Task Size   = 112 byte
Active Task Num    = 4
Inactive Task Size = 0 byte
Inactive Task Num  = 0
 _ _ _ _ _ _ _ _ _ _ _
John
Richard
Robert
All Task Size      = 112 byte
All Task Num       = 4
Active Task Size   = 84 byte
Active Task Num    = 3
Inactive Task Size = 28 byte
Inactive Task Num  = 1
 _ _ _ _ _ _ _ _ _ _ _
John
Robert
All Task Size      = 56 byte
All Task Num       = 2
Active Task Size   = 56 byte
Active Task Num    = 2
Inactive Task Size = 0 byte
Inactive Task Num  = 0
||<
<a href="#contents">目次に戻る</a>


上記の例ではタスクの操作をわかりやすくするために処理関数を複数呼び出していますが、ゲーム等で使うときは、whileループ等で周期的に呼ばれるようにするとよいと思います。


以上がリファレンスです。
誤字脱字、ライブラリ自体のバグ等を発見次第コメント欄等で指摘して頂けると幸いです。
他にも質問などもお気軽にお寄せください。
では。
