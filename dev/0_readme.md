# README (TogoDX-server 用 SPARQList 手引)

- 内訳を返すのではなく、フロントエンド表示に必要な全データを返す
  - 内訳計算などは TogoDX-server で行う
  - mode での出力の場合分けも無し
  - パラメータも基本不要
- データ形式は"連続値系"と"分類系"の２種類
  - どちらも key, value オブジェクトの配列
  - 配列の順番は問わない
- SPARQList 名は前と同じ
  - 作業完了したら公開版 <a href="https://togodx.integbio.jp/sparqlist/">https://togodx.integbio.jp/sparqlist/</a> にコピーして、DX-serverのテストデータとして使用
    - しばらくは公開版も Web UI からも保存できるようにしておく
    - 落ち着いたら公開版は github 経由に以降
  - Description も一応コピーしておく
  - gthub の <a href="https://togodx.integbio.jp/sparqlist/1_DX-server_config">config</a> 追記してプルリク
- データが大きくなりがちで、web UI からだと結果を表示できない場合もあるので、curl などで download して確認

## 連続値系
エキソンの数、タンパク質の分子量、翻訳後修飾の数など、フロントエンドでヒストグラム表示するもの
- 要素（遺伝子、タンパク質、化合物など）の ID、ラベル、値などを記述
- 例（UniProt molecular mass）
```json
[
  {
    "id": "A0A087WXM7",
    "label": "A0A087WXM7_HUMAN",
    "value": 22482,
    "binId": 3,
    "binLabel": "20-30 kDa"
  },
  ...
]
```
- keys
  - id (必須): 要素の ID
  - label (必須): 要素のラベル
    - view results テーブルで表示される
  - value (必須:demical): 値
  - binId (必須): フロントエンドで利用する bin の ID
    - フロントエンドでバーの上での並びを反映した ASCII で sort 可能なものにしておく
      - 独自に数値を振るなら positive integer
      - 0 スタートは避ける
  - binLabel (必須): フロントエンドで表示するラベル
  - (binBegin, binEnd: 廃止（作った人は消してください）)
- binning
  - binning しない場合も binId, binLabel が必要
    - value と同じでも良いが binId は 0 を避けるて +1 したりする
  - renge サイズは SPARQList 作成者に任せる
    - 今の '# of exons' のように等分じゃない bin はつくらない
  - ○以上○未満を基本 (begin <= X < end)
    - ラベルはわかりやすい感じで
      - カウントとかの離散値の場合 [0〜9, 10〜19, 20〜29] のような以上以下ラベルのほうがわかりやすいかもしれないが、どうか
    - ○超過○以下にしたい場合は報告 (begin < X <= end)
- 参考
  - <a href="./protein_number_of_phosphorylation_sites_uniprot">protein_number_of_phosphorylation_sites_uniprot</a>
  - <a href="./protein_molecular_mass_uniprot">protein_molecular_mass_uniprot</a>

## 連続値系：カウント0
- 何かの数をカウントする系は、p-value 計算に必要なため 0 個であるという情報も取得する
  - 例外もあるかとは思う
  - データ取得例は下記参照
- カウント以外で値を持つ要素と持たない要素が出てくる場合とかあるかな？

## 分類系：基本
- ２種類の関係（ツリーやグラフのエッジ情報）を記述
  - 分類カテゴリの親子関係
  - 要素のアノテーション関係を記述
- 例（UniProt の GO:celluler_component 分類)
```json
[
  {                                         // ルートノード
    "id": "GO_0005575",
    "label": "cellular_component",
    "root": true
  },
  {                                         // 分類カテゴリの階層のノード
    "id": "GO_0005826",
    "label": "actomyosin contractile ring",
    "parent": "GO_0070938"
  },
  {                                         // 要素への分類カテゴリのアノテーション
    "id": "A0A0U1RQR1",
    "label": "A0A0U1RQR1_HUMAN",
    "leaf": true,
    "parent": "GO_0000407"
  },
  ...
]
```
- keys
  - id (必須): 要素やカテゴリの ID
  - label (必須): 要素やカテゴリのラベル
    - view results テーブルに表示される
  - parent: 要素やカテゴリの親ノードの ID
    - ルートノードはには不要、他は必須
  - leaf: boolean
    - "id" が要素（遺伝子、タンパク質、化合物など、内訳のカウント対象）の場合 true
      - disease の例のようにオントロジーの病気自体を数えてる場合は保留
        - 全てのオントロジーに同じ情報を持ったLeafを作る（三橋さんの案）
    - 分類カテゴリの場合 false
      - false の場合は無くても良い
  - root: boolean
    - ルートノードのみ true
- prefix が無いと、要素 ID と 分類カテゴリ ID が区別つかなくなる場合（どっちも数値だけとか）
  - leaf フラグで区別してるから問題無いはず
  - 分類カテゴリ ID 側に適当な Prefix をつけても良い（フロントエンドとDX-serverの通信に使われるだけなので）
    - 区別付かない場合は一応報告
- オントロジーに要素のぶら下がっていないノードがある場合
  - そのノードをわざわざ取り除く必要はない（取り除いても良い）
- 参考（備考：入れ子のバックエンド SPARQLet なのでパラメータを受け取ってる）
  - <a href="./backend_protein_go_uniprot">backend_protein_go_uniprot</a>
    - カテゴリの親子関係とアノテーション関係を別の SPARQL クエリで取る例
  - <a href="./backend_protein_uniprot_keywords_uniprot">backend_protein_uniprot_keywords_uniprot</a>
    - カテゴリの親子関係とアノテーション関係を UNION 句を使って１つの SPARQL クエリで取る例
  - <a href="./backend_gene_expression_level_refex">backend_gene_expression_level_refex</a>

## 分類系：ルートノードが無い場合や、階層が無い場合
- ルートノードを作成して、階層の無い分類や親の居ない分類をルートノードにぶら下げる
- 内訳をカウント以外でソートする場合、id は ASCII で sort 可能なものにしておく
  - 何でソートするかは、config json に記述予定（内訳数、ID）
  - Chromosome の場合、[1〜22 ,X, Y, MT] -> [1〜22, 23, 24, 25] のように
  - 0 は避ける
- 例（Ensembl gene の chromosome による分類）
```json
[
  {                                      // ルートノードを作成
    "id": "root",
    "label": "root node",
    "root": true
  },
  {                                      // 階層の無い、または親の居ない分類のノード
    "id": 3,
    "label": "chr3",
    "leaf": false,
    "parent": "root"                     // ルートノードにぶら下げる
  },
  {
    "id": 25,                            // id を sortable に
    "label": "MT",
    "leaf": false,
    "parent": "root"
  },
  {
    "id": "ENSG00000001617",
    "label": "SEMA3F",
    "leaf": true,
    "parent": 3
  },
  ...  
]
```
- 
- 参考
  - <a href="./gene_chromosome_ensembl">gene_chromosome_ensembl</a>

## 分類系：アノテーションの無い要素
- 専用のノード "id: unclassified" を作成してルートノードにぶら下げる
  - DX-server側でのソート制御のため、 "unclassified" 固定
- p-value 計算に必要なため、アノテーション無しの要素がある場合は作成する
  - 計算には全体の数（母数）が必要
  - 遺伝子、タンパク質以外は難しいか
    - "パスウェイに乗っていない化合物" などは、考慮することは難しい
- 例（GOアノテーションの付かないUniProt）
```json
[
  {
    "id": "GO_0005575",
    "label": "cellular_component",
    "root": true
  },
  {                                         // アノテーションの無い要素カテゴリノードを作成
    "id": "unclassified",                   // 専用 unclassified ノード
    "label": "without annotation",          // ラベルは自由に適切なものを
    "parent": "GO_0005575"                  // ルートノードにぶら下げる
  },
  {                                         // アノテーションの無い要素の unclassified への分類
    "id": "Q5XG85",
    "label": "U633C_HUMAN",
    "leaf": true,
    "parent": unclassified"                 // unclassified ノードにぶら下げる
  },
  ...  
]
```
  - データ取得例は下記参照

## 分類系：イメージ

<img src="https://sparql-support.dbcls.jp/tmp/file/tree.jpg" height="400">

- 全てのエッジ情報 + ルートノードが必要
  - A .. Z はオントロジーなどの分類ノード
    - 共通ルートが無い場合には root node を作成
    - ルートの下の第一階層となる A, X の２つが、最初に attribute バーの内訳に出る分類
  - g1 .. g9 は geneなどの要素 = leaf
    - tree 状になっていない DAG (node D) の場合も親毎に
    - multi annotation (leaf g6) も分類毎に
    - I は分類の終端だけど leaf ではない

```pre
# id(self), parent, leaf
root, ,
A, root,
B, A,
C, A,
D, A,
E, B,
F, B,
G, C,
G, D,
H, D,
I, D,  // leaf につながらないのでこのエッジは有っても無くても結果に影響しない
X, root,
Y, X,
Z, X,
g1, E, true
g2, E, true
g3, E, true
g4, F, true
g5, C, true
g6, G, true
g6, H, true
g7, H, true
g8, Y, true
g9, Z, true
```

- Attribute バー表示イメージ

<img src="https://sparql-support.dbcls.jp/tmp/file/tree2.png" height="140">


## その他：カウント0要素, アノテーションの無い要素の取得方法
- 要素全体との差分を取る
  - 例：<a href="./backend_protein_uniprot_keywords_uniprot">backend_protein_uniprot_keywords_uniprot</a>
  - 全体が巨大だと遅くなることもある
- MINUS 句でアノテーションの無いものを取得
  - 例：<a href="./protein_number_of_glycosylation_sites_uniprot">protein_number_of_glycosylation_sites_uniprot</a>
  - 場合によっては著しく遅くなるので注意
- 他にもあったら追記してください

## その他：入れ子 SPARQLet でコード流用
- 以前と同じ
- 参考：下記の関係
  - <a href="./protein_domains_uniprot">protein_domains_uniprot</a>
  - <a href="./backend_protein_uniprot_keywords_uniprot">backend_protein_uniprot_keywords_uniprot</a>
    - 後ろはの SPARQLet 名は "backend_[category]_ [description]_ [database]"