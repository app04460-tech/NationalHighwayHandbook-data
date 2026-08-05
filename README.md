# NationalHighwayHandbook-data

iOSアプリ「国道図鑑」が使用する、日本の一般国道459路線の加工済みデータ
(`kokudo.json`)を公開するリポジトリです。

このデータは OpenStreetMap のデータを加工して作成した派生データベースであり、
**Open Database License (ODbL) 1.0** のもとで公開します(`LICENSE` 参照)。

## 出典

- **路線の形状(ポリライン)**: © OpenStreetMap contributors (ODbL)
  https://www.openstreetmap.org/copyright
  - Overpass API から一般国道のルートリレーション
    (`type=route, route=road, name=国道◯号`)を取得(取得日: 2026年8月4日〜5日)
- **延長等の数値・経過都道府県**: 国土交通省「道路統計年報2024」
  表26「一般国道の路線別、都道府県別道路現況」(令和4年度末現在)を基に作成
- **起点・終点**: 「一般国道の路線を指定する政令」(昭和40年政令第58号)別表
  (e-Gov法令検索より取得)
- **制定年**: Wikipedia 各路線記事の infobox を集計

地図上の線形はOSM由来のため、`totalKm` 等の公称値とは算出方法が異なります。

## 生成手順

生成スクリプトはアプリ本体リポジトリの `script/` にあります。

1. `python3 script/fetch_osm.py`
   — Overpass API から全459路線の生データを `rawdata/osm/{番号}.json` に取得
   (リレーションが複数ある路線等は `rawdata/osm/overrides.json` の
   リレーションIDで解決)
2. `python3 script/build_attributes.py`
   — 道路統計年報・政令・Wikipedia から `rawdata/attributes.csv`(459行)を生成
3. `python3 script/preprocess.py`
   — way連結・区間種別分類(normal / sea / steps)・重用路線の収集・
   Douglas-Peucker 間引き(許容誤差0.0002度)を行い `kokudo.json` を出力

## フォーマット概要

```json
{
  "sources": { "geometry": "...", "attributes": "...", "dataRepo": "..." },
  "dataYear": "2024",
  "dataVersion": 2,
  "highways": [
    {
      "number": 42,
      "name": "国道42号",
      "origin": "浜松市",
      "terminal": "和歌山市",
      "totalKm": 537.1,
      "realKm": 484.0,
      "gendoKm": 429.9,
      "overlapKm": 33.4,
      "seaKm": 19.6,
      "designatedYear": 1959,
      "prefectures": ["静岡県", "愛知県", "三重県", "和歌山県"],
      "regions": ["中部", "近畿"],
      "overlapRoutes": [1, 23, 167, 259, 311, 424],
      "hasSea": true,
      "hasSteps": false,
      "originCoord": [137.5891, 34.7102],
      "terminalCoord": [135.1712, 34.2261],
      "polylines": [
        { "kind": "normal", "coords": [[経度, 緯度], ...] },
        { "kind": "sea", "coords": [...] }
      ]
    }
  ]
}
```

- `kind`: `normal`(通常区間)/ `sea`(海上区間・フェリー航路)/
  `steps`(階段区間)
- 座標は `[経度, 緯度]`、小数第5位丸め
- **`prefectures` は都道府県コード順(JIS X 0401、北から)であり、
  経路順(起点→終点順)ではありません**。出典の道路統計年報 表26の
  掲載順に由来します。起点・終点の位置が必要な場合は `originCoord` /
  `terminalCoord`(経路端点の座標、向き確定済み)を使ってください
- `originCoord` / `terminalCoord`: 起点・終点のマーカー座標
  `[経度, 緯度]`。路線ポリラインの端点のうち、起点・終点名を
  ジオコーディングした座標に最も近いものを選んで向きを確定している

## ライセンス

このデータベースは Open Database License (ODbL) 1.0 で提供されます。
利用の際は「© OpenStreetMap contributors」の帰属表示と、
派生データベースを公開する場合の同ライセンスでの提供が必要です。
