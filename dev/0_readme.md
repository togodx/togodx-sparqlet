# README (TogoDX-server 用 SPARQlist 手引)

- 内訳を返すのではなく、フロントエンド表示に必要な全データを返す
  - 内訳計算などは TogoDX-server で行う
  - mode での出力の場合分けも無し
  - パラメータも基本不要
- データ形式は"連続値系"と"分類系"の２種類
- どちらも key, value オブジェクトの配列
  - 配列の順番は問わない
- SPARQList 名は前と同じ
  - 一応 Description 部分もコピペ
- データが大きくなりがちで、web UI からだと結果を表示できない場合もあるので、curl などで download して確認

## 連続値系 (策定中。bin周りがまだ変わるかもしれないけれど、id,label,valueはとりあえず必要）
- 要素（遺伝子、タンパク質、化合物など）の ID、ラベル、値などを記述
- 例（UniProt molecular mass）
```json
[
  {
    "id": "A0A087WXM7",
    "label": "A0A087WXM7_HUMAN",
    "value": 22482,
    "binId": 3,
    "binBegin": 20000,
    "binEnd": 30000,
    "binLabel": "20-30 kDa"
  },
  ...
]
```
- keys
  - id (必須): 要素の ID
  - label (必須): 要素のラベル
  - value (必須:demical): 値
  - binId (必須): フロントエンドで利用する bin の ID
    - 数値のように sort に利用できるもの
      - 独自に数値を振るなら場合 integer
      - 0 スタートは避けたい
  - binBegin (必須:demical): フロントエンドのバー表示で binning する場合の begin 値
    - binning しない場合は value と同じ
    - ○未満全部を指定する場合は false
  - binEnd (必須:demical): フロントエンドのバー表示で binning する場合の end 値
    - binning しない場合は value と同じ
    - ○以上全部を指定する場合は false
  - binLabel (必須): フロントエンドで表示するラベル
- binning
  - renge サイズは SPARQList 作成者に任せる
  - ○以上○未満を基本 (begin <= X < end)
    - ○超過○以下にしたい場合は報告 (begin < X <= end)
- 参考
  - <a href="./protein_number_of_phosphorylation_sites_uniprot">protein_number_of_phosphorylation_sites_uniprot</a>
  - <a href="./protein_molecular_mass_uniprot">protein_molecular_mass_uniprot</a>

## 分類系：基本
- 分類カテゴリの親子関係、要素のアノテーション関係を記述
- 例（UniProt の GO:celluler_component 分類)
```json
[
  { // ルートノード
    "id": "GO_0005575",
    "label": "cellular_component",
    "root": true
  },
  { // 分類カテゴリの階層のノード
    "id": "GO_0005826",
    "label": "actomyosin contractile ring",
    "parent": "GO_0070938"
  },
  { // 要素への分類カテゴリのアノテーション
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
  - parent: 要素やカテゴリの親ノードの ID
    - ルートノードはには不要、他は必須
  - leaf: boolean
    - "id" が要素（遺伝子、タンパク質、化合物など）の場合 true
    - 分類カテゴリの場合 false
      - false の場合は無くても良い
  - root: boolean
    - ルートノードのみ true
- prefix が無いと要素 ID と 分類カテゴリ ID が区別つかなくなる場合（どっちも数値だけとか）
  - leaf フラグで区別してるから問題無いはず
    - 区別付かない場合は一応報告
- 参考（備考：入れ子のバックエンド SPARQLet なのでパラメータを受け取ってる）
  - <a href="./backend_protein_go_uniprot">backend_protein_go_uniprot</a>
  - <a href="./backend_protein_uniprot_keywords_uniprot">backend_protein_uniprot_keywords_uniprot</a>
  - <a href="./backend_gene_expression_level_refex">backend_gene_expression_level_refex</a>

## 分類系：ルートノードが無い場合や、階層が無い場合
- ルートノードを作成して、階層の無い分類や親の居ない分類をルートノードにぶら下げる
- 例（Ensembl gene の chromosome による分類）
```json
[
  {  // ルートノードを作成
    "id": "root",
    "label": "root node",
    "root": true
  },
  {  // 階層の無い、または親の居ない分類のノード
    "id": "3",
    "label": "chr3",
    "leaf": false,
    "parent": "root"  // ルートノードにぶら下げる
  },
  {
    "id": "ENSG00000001617",
    "label": "SEMA3F",
    "leaf": true,
    "parent": "3"
  },
  ...  
]
```

## 分類系：アノテーションの無い要素
- 専用のノードを作成してルートノードにぶら下げる
- p-value 計算に必要なため、アノテーション無しの要素がある場合はかならず作成する
  - 計算には全体の数（母数）が必要
- 例（GOアノテーションの付かないUniProt）
```json
[
  {
    "id": "GO_0005575",
    "label": "cellular_component",
    "root": true
  },
  {  // アノテーションの無い要素カテゴリノードを作成
    "id": "wo_GO_0005575", //  データ内で unique になるよう ID をつける
    "label": "without annotation",
    "parent": "GO_0005575"  // ルートノードにぶら下げる
  },
  {  // アノテーションの無い要素
    "id": "Q5XG85",
    "label": "U633C_HUMAN",
    "leaf": true,
    "parent": "wo_GO_0005575"  // 作成したノードにぶら下げる
  },
  ...  
]
```

## その他：入れ子 SPARQLet でコード流用
- 以前と同じ
- 参考：下記の関係
  - <a href="./protein_domains_uniprot">protein_domains_uniprot</a>
  - <a href="./backend_protein_uniprot_keywords_uniprot">backend_protein_uniprot_keywords_uniprot</a>
    - 後ろはの SPARQLet 名は "backend_[category]_ [description]_ [database]"