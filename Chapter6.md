## Case文とEnum列挙

### Case文

例えば有限状態機械を記述したい場合特に、特定の条件に対して場合分けをしてそれぞれの場合について代入やassignを行いたい時があります。
例えばneru、aho、majimeの3状態を持つ有限状態機械は 

```systemverilog
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
これは**case**文の使用例です。caseの後のカッコ内に場合分け条件を持つオブジェクト名を記述し、caseとendcaseの間に記述します。勿論場合を評価される式は一つ(この例の場合FSM)なのでif文と違い優先順位はありません。注意したいのは```case (...) begin```と**beginを使用せず、endcaseで閉じます**。

この例ではローカルパラメータを利用して状態を保持するFSMへ代入しています。代入条件はlogicやwireといったオブジェクトとして宣言されているaho、majime、neruがif文の条件を満たす時、clockの立ち上がりエッジをイベントとして代入します。

FSMは合計４つの状態を保持できますが、"FSM == 2'h3"の状態が未定義となっています。なんらかの動作不良で未定義の状態へ遷移した場合の動作保証ができなくなるので、論理回路を合成し検証する際、未定義の状態を可能な限り排除する必要があります。これを支援するため、**default**が用意されています。

またこの例ではAHOの状態でMAJIMEでもNERU状態でもない暗黙の状態を含んでいます。if文が全て成立しない場合、代入されないのでこの時暗黙にAHOの状態となります。この暗黙の状態も検証する際にワーニングとしてレポートされるでしょう。さらに**全てが自明な場合**、caseでは暗黙の値を含まない場合にuniqueという識別子を用いて```unique case```として論理合成系を支援することができます。


### Enum列挙

この例では状態を保持するlogic FSMに対してローカルパラメータで整数値をラベル(NERU、AHO、MAJIME)へ指定して使用しました。開発をしているとたまに同じ名前でパラメータを使用している状態が発生し、想定と異なった値を代入や割り当てをしてしまうことがあります。例えばシミュレーション上なぜか違った値が表示され検証に時間を浪費する、といったことが発生しえます。

これを避けるために固有の名前を定義することができます。固有の名前を特定のオブジェクト(型)に紐づけることで誤った名前の使用を避けることができます。
例えば、

```systemverilog
enum [1:0] {
	NERU	= 2'b00,
	AHO     = 2'b01,
    MAJIME	= 2'b10,
    KYOMU	= 2'b11
	} ningen_f;
```

では**enum**を使用して４つの状態を列挙した変数ningen_fとしています。
文法は

```systemverilog
enum 幅 {
	名前1   = 3,
    名前2   = 0,
    ...
	} オブジェクト名;
```

で、名前を”,"で区切って列挙します。
ユーザ定義型に使用すると例えば

```systemverilog
typedef enum logic [1:0] {
	NERU	= 2'b00,
	AHO     = 2'b01,
    MAJIME	= 2'b10,
    KYOMU	= 2'b11
	} ningen_t;
```

とすることでlogic型に紐づいたningen_t型を用意でき、下記のように宣言して使用します。

```systemverilog
ningen_t    FSM;
```

これによりFSMに固有の名前(ラベル)を付与できます。使用例は先のローカルパラメータの例と同じですが、この型以外のオブジェクトで名前を使用するとエラーになります。

この型で宣言されたオブジェクトは通常名前で代入や割り当てを行いますが、直接数値で指定もでき大小比較を用いて組合せ回路を作ることを可能にしています。例えば順番が定義されている時その範囲内を条件として、あるいは範囲外を条件としたいときに有用です。また、パラメータと同様にenumで列挙した部分の修正や更新を行えばよく、冗長な書き換え作業を省くことができ、さらにコード本体に触れることが基本ないため、書き換えに伴う誤ったコーディングを防ぐことができます。
