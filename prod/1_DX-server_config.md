# DX-server config

- SPARQList をここ (https://togodx.integbio.jp/sparqlist/) にコピー
- github の togodx-config-human の dx-server_dev ブランチの config 2つに追記する

## properties.json
- <a href="https://github.com/togodx/togodx-config-human/blob/dx-server_dev/config/properties.json">https://github.com/togodx/togodx-config-human/blob/dx-server_dev/config/properties.json</a>
  - develop ブランチの config をテスト用に削ったものなので、SPARQList の準備ができたら消されたた部分を下からコピペして復帰
  - コピペ元 (develop ブランチ)：<a href="https://github.com/togodx/togodx-config-human/blob/develop/config/properties.json">https://github.com/togodx/togodx-config-human/blob/develop/config/properties.json</a>

## yaml
- <a href="https://github.com/togodx/togodx-config-human/blob/dx-server_dev/config/dx-server.yaml">https://github.com/togodx/togodx-config-human/blob/dx-server_dev/config/dx-server.yaml</a>
- 順番は問わないので、一番下に追記
  - api: API名
  - dataset: DB key 名
  - datamodel: Classification（分類） / Distribution（連続値）