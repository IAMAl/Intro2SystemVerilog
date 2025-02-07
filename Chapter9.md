## 9. インターフェイス

前までの章で構造体を導入して記述することでその構造体を更新するだけでその構造体を使用している記述を修正する必要がなく煩雑な作業をなくしてかつ手作業による間違いを避ける利点を紹介しました。モジュールのポートにそのような構造体を使用することもできますし、モジュールの間を接続する際にも使用できます。

ネット型のような使い方についてそのアイデアを発展させたものがあります。
**インターフェイス**というアイデアをこの章で取り上げます。
インターフェイスは名前の通り入出力部分に該当します。
インターフェイス部分が明示的な場合これをインスタンスとして再利用したい時があります。
インターファイスは**modport**関数を併用してインターフェイスを定義できます。
インターフェイスは**interface**と**endinterface**の間にインターフェイスで使用するオブジェクトを宣言しmodport関数で使用します。
modport関数はポート宣言とともに用意したオブジェクトを宣言します。
modportは関数なので列挙して引数を示します。

```verilog
interface my_if;

    logic   [WIDTH-1:0] data1, data2;
    logic               valid;
    logic               ready;

    modport master (
        output  data1,
        output  data2, valid,
        input   ready
    );

    modport slave (
        input  data1,
        input  data2, valid,
        output   ready
    );
endinterface
```

例えば上述のようなインターフェイスを作った場合、master側のインターフェイスとslave側のインターフェイスの対応が明示的でわかりやすいでしょう。
この```interface```は```module```と同等で同様にしてパラメータ宣言やパッケージなどのインポートができます。
このような記述をしたインターフェイスを用いることでこの構造に従ったインターフェイスを用意することができ、モジュールのポートや結線ロジックなどとして利用できます。

```modport```関数は
```verilog
modport ポート名 (
    入出力ポート宣言子  予め用意したオブジェクト
);
```
の形式で引数を','使用して列挙します。
入出力ポート宣言子に対して複数のオブジェクト引数を列挙することもできます。

例えば、

```verilog
module hoge (
    input               clock,
    input               reset,
    my_if.master        aho,
    ...
);

    my_if           nemui;

    nandakana ehehe(
        .clock(     cloxk       ),
        .reset(     reset       ),
        .nani(      aho         ),
        .nandato(   nemui.slave )
    );

    assign nemui.master.data1   = data_nankana;
    .... 

    if (aho.ready) begin
        ...

endmodule
```

のような使い方をします。
この例ではモジュールのポートについてインターフェイスを使用してmaster側のインターフェイスを使用することを宣言しています。
モジュール内では**nemui**というオブジェクトを用意して結線に使用しています。
ここでポート```nandato```は```modport master```、ポート```nandato```は```modport slave```と同じ構造の出力とします。
この記述によりmasterからmasterへの方向へ信号が伝播することを明示できていて何らかのロジックで使用されslave側のインターフェイスへ割り当てslave側で伝播していきます。
