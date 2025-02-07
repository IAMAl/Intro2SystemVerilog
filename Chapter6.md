## 6. Case文とEnum列挙

### Case文

例えば有限状態機械を記述したい場合、特に特定の条件に対して場合分けをして、それぞれの場合について代入やassignを行いたい時があります。
例えばneru、aho、majimeの3状態を持つ有限状態機械は

```verilog
localparam NERU = 0;
localparam AHO = 1;
localparam MAJIME = 2;
logic   [1:0] FSM;
always_ff @( posedge clock ) begin
    if ( reset ) begin
        FSM <= NERU;
    end
    else case (FSM)
        NERU: begin
            if ( aho ) begin
                FSM <= AHO;
            end
            else if ( majime ) begin
                FSM <= MAJIME;
            end
            else begin
                FSM <= NERU;
            end
        end
        AHO: begin
            if ( majime ) begin
                FSM <= MAJIME;
            end
            else if ( neru ) begin
                FSM <= NERU;
            end
        end
        MAJIME: begin
            if ( aho ) begin
                FSM <= AHO;
            end
            else if ( neru ) begin
                FSM <= NERU;
            end
            else begin
                FSM <= MAJIME;
            end
        end
        default: begin
                FSM <= NERU;
        end
    endcase
end
```

と記述できます。
これは**case**文の使用例です。
caseの後のカッコ内に場合分け条件を持つオブジェクト名を記述し、**case**と**endcase**の間に記述します。
勿論場合を評価される式は一つ(この例の場合FSM)なのでif文と違い優先順位はありません。
注意したいのは```case (...) begin```のように**beginを使用せず、endcaseで閉じます**。

この例ではローカルパラメータを利用して状態を保持するFSMへ代入しています。代入条件は```logic```や```wire```といったオブジェクトとして宣言されている```aho```、```majime```、```neru```がif文の条件を満たす時、clockの立ち上がりエッジをイベントとして代入します。

FSMは合計４つの状態を保持できますが、"FSM == 2'h3"の状態が未定義となっています。
なんらかの動作不良で未定義の状態へ遷移した場合の動作保証ができなくなるので、論理回路を合成し検証する際、未定義の状態を可能な限り排除する必要があります。
これを支援するため、**default**が用意されています。
```default```にcase文内で記述していない場合の代入式を記述しておきます。

またこの例では```AHO```の状態で```MAJIME```でも```NERU```状態でもない暗黙の状態を含んでいます。
if文が全て成立しない場合、代入されないのでこの時暗黙に```AHO```の状態となります。
この暗黙の状態も検証する際にワーニングとしてレポートされるでしょう。
さらに**全てが自明な場合**、caseでは暗黙の値を含まない場合に```unique```という識別子を用いて```unique case```として論理合成系を支援することができます。


### Enum列挙

この例では状態を保持するlogic FSMに対してローカルパラメータで整数値をラベル(```NERU```、```AHO```、```MAJIME```)へ指定して使用しました。
開発をしているとたまに同じ名前でパラメータを使用している状態が発生し、想定と異なった値を代入や割り当てをしてしまうことがあります。
例えばシミュレーション上なぜか違った値が表示され検証に時間を浪費する、といったことが発生しえます。

これを避けるために固有の名前を定義することができます。
固有の名前を特定のオブジェクト(型)に紐づけることで誤った名前の使用を避けることができます。
例えば、

```verilog
enum [1:0] {
	NERU	= 2'b00,
	AHO     = 2'b01,
    MAJIME	= 2'b10,
    KYOMU	= 2'b11
	} ningen_e;
```

では**enum**を使用して４つの状態を列挙した変数```ningen_e```としています。
文法は

```verilog
enum 幅 {
	名前1   = 定数,
    名前2   = 定数,
    ...
	} 列挙名;
```

で、列挙する名前を”,"で区切って列挙します。
ユーザ定義型に使用すると例えば

```verilog
typedef enum logic [1:0] {
	NERU	= 2'b00,
	AHO     = 2'b01,
    MAJIME	= 2'b10,
    KYOMU	= 2'b11
	} ningen_t;
```

とすることでlogic型に紐づいた```ningen_t```型を用意でき、下記のように宣言して使用します。

```verilog
ningen_t    FSM;
```

これによりFSMに固有の名前(ラベル)を付与できます。
使用例は先のローカルパラメータの例と同じですが、この型以外のオブジェクトで名前を使用するとエラーになります。

この型で宣言されたオブジェクトは通常名前で代入や割り当てを行いますが、直接数値で指定もでき、大小比較を用いて組合せ回路を作ることを可能にしています。
例えば順番が定義されている時その範囲内を条件として、あるいは範囲外を条件としたいときに有用です。
また、パラメータと同様に```enum```で列挙した部分の修正や更新を行えばよく、冗長な書き換え作業を省くことができ、さらにコード本体に触れることが基本ないため、書き換えに伴う誤ったコーディングを防ぐことができます。