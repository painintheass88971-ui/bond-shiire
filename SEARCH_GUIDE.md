# 仕入れ検索 手順書（AIチャット用）

このファイルは、BOND Vintage の仕入れアプリ用の「取込コード」を作るときの手順書です。
推奨モデルは **GPT-6.1 Sol** です（チャットのモデル選択で切り替えてください）。

## いつ実行するか（重要）
次のどちらかのときだけ、この手順の検索（各サイトを巡回する作業）を行います。
- 文の先頭が **「$仕入れ先検索」**、**「仕入れ検索」**、**「仕入れ先検索」** のどれかで始まる命令が届いたとき
- このファイルを名指しして「取込コードを作って」「検索して」と頼まれたとき

服の手がかりは、**画像と文字情報のどちらでも、両方でも**かまいません（例：ブランド、品名、色、サイズ、タグの文字、出品ページのURL）。
命令だけが届いて手がかりがまだない場合は、「画像か服の情報を送ってください」とだけ返して待ちます。届いたら開始します。

それ以外で画像が送られた場合（投稿準備や商品登録の相談など）は、サイトの巡回をしません。依頼された内容にだけ答えます。
判別だけを頼まれた場合は、検索語までの取込コード（`results` なし）を返します。

## 依頼のしかた（ユーザー側）
| やりたいこと | 送る文 |
|---|---|
| 1回目（25件） | `仕入れ検索` ＋ 画像や服の情報 |
| 次の25件 | `仕入れ検索 続き` |
| 判別と検索語だけ | `仕入れ検索 ワードだけ` ＋ 画像や服の情報 |

「仕入れ検索」の代わりに「仕入れ先検索」と送っても同じです。

最初の依頼では、このファイルの場所も伝えます：
- PCのチャット：`C:\Users\advke\OneDrive\BondVintage_Shiire_App\SEARCH_GUIDE.md`
- スマホなどほかの端末：`https://github.com/painintheass88971-ui/bond-shiire/blob/main/SEARCH_GUIDE.md`

**ブラウザを操作できないチャット**（スマホのチャットなど）では、各サイトの巡回ができません。その場合は、`results` を含まない判別と検索語だけの取込コードを返します。そのうえで「出品の一覧が必要なら、PCのチャットで同じ画像を貼って『仕入れ検索』と送ってください」と伝えます。
（アプリの「AIへの依頼文をコピー」で、この文をコピーできます）

## 段階的な検索（25件ずつ）
- 1回の検索で追加する結果は **25件まで** です。「続き」と言われるたびに、次の25件を足します。**合計100件まで**です。
- `"batch": 回数` を入れます（1回目は `1`、続きは `2`、`3`、`4`）。
- `results/latest.json` には、前の結果も含めた**合計**を保存します（アプリはファイルの中身をそのまま表示します）。
- 取込コードをチャットに直接出す場合（保存できないとき）は、`batch` が2以上なら新しい分だけを入れます。アプリが前の結果に追加します。
- すでに出したURLは、もう一度入れません。似ているものの残りが少なくなったら、ほかの検索語を使います。
- 2回目以降は、新しく検索した語を `resultsMeta` に書きます。

## 守ること（AI側）
- 購入、入札、いいね、フォロー、メッセージ送信、ログインはしない。検索結果を見るだけにする。
- 仕入れアプリ（このフォルダ）以外のサイトやファイルは変更しない。
- eBay は自動ブラウザがブロックされるので対象外にする（アプリの検索ボタンからは開ける）。
- 返答は簡潔な日本語で書く。

## 手順
1. **画像を判別する**：ブランド、アイテムの種類、年代の目安、素材、色、柄、ディテール（襟、ジップ、ポケットなど）、サイズ、生産国、タグを読み取る。
2. **検索語を作る**（それぞれ日本語と英語を用意する）
   - `keywords`（3つ前後）：同じアイテムを探す語。ブランド名を入れる。
   - `design`（3つ前後）：**ブランド名を使わず**、デザインの特徴だけで探す語（例：ボア襟 チェック ウール ジップ）。
   - `similar`（3つ前後）：似ていて狙い目になる、ほかのブランドやモデル。
3. **各サイトを検索する**：下の表のURLに検索語を入れて開き、下の「抽出スクリプト」で出品一覧を取り出す。
   - 国内サイトは日本語の語、海外サイトは英語の語を使う。
   - `keywords` の1つ目と `design` の1つ目を、全サイトで検索する。件数が足りなければ、ほかの語も使う。
4. **画像で選ぶ**：取り出した出品のサムネイルを見て、**見た目が本当に近いものだけ**を選ぶ。タイトルだけで判断しない。
   - 選ぶ基準：形、柄、配色、ディテールが元の画像と共通していること。ブランドが違ってもデザインが近ければ選んでよい。
   - **1回で25件まで**（続きは上の「段階的な検索」に従う）。サイトごとに偏りすぎないようにし、近い順に並べる。
5. **結果を保存して共有する**（ユーザーが選んだ共有方式。スマホでも自動で見られるようにする）
   - `C:\Users\advke\OneDrive\BondVintage_Shiire_App\results\latest.json` に、下の形式のJSONを UTF-8 で**上書き保存**する。
   - 中身は**その商品についての全部**にする。1回目は判別、ワード、25件の結果。続きのときは、前の `results` に新しい25件を足した合計を入れる。
   - `"savedAt"` に保存した日時（ISO形式。例：`2026-11-15T23:30:00+09:00`）を必ず入れる。アプリは、この値が変わったことで新しい結果だと判断する。
   - 保存したら、このファイルだけをコミットしてプッシュする：
     ```powershell
     cd C:\Users\advke\OneDrive\BondVintage_Shiire_App
     git add results/latest.json
     git commit -m "results: <商品名> batch <回数>"
     git push
     ```
   - ほかのファイルはコミットしない。プッシュに失敗したら、取込コードをチャットに出して、アプリに貼ってもらう。
6. **チャットに報告する**：「保存しました。1〜2分後にアプリを開くか『最新の検索結果を読み込む』を押してください」と伝える。注目の3件と注意点を短く添える。取込コードの全文は出さなくてよい。
   - アプリ：https://painintheass88971-ui.github.io/bond-shiire/
   - VS Codeの中のブラウザに直接取り込む必要はない（そこにだけ保存されて、ほかの端末に反映されないため）。

## 検索URL（`{q}` に検索語をURLエンコードして入れる）
| サイト | URL | 使う語 | 抽出パターン |
|---|---|---|---|
| メルカリ | `https://jp.mercari.com/search?keyword={q}&status=on_sale` | 日本語 | `jp\.mercari\.com/item/m` |
| ヤフオク | `https://auctions.yahoo.co.jp/search/search?p={q}` | 日本語 | `auctions\.yahoo\.co\.jp/jp/auction/` |
| Yahoo!フリマ | `https://paypayfleamarket.yahoo.co.jp/search/{q}?open=1` | 日本語 | `paypayfleamarket\.yahoo\.co\.jp/item/` |
| ラクマ | `https://fril.jp/s?query={q}` | 日本語 | `item\.fril\.jp/` |
| Depop | `https://www.depop.com/search/?q={q}` | 英語 | `depop\.com/products/` |
| Vinted | `https://www.vinted.com/catalog?search_text={q}` | 英語 | `vinted\.com/items/` |
| Grailed | `https://www.grailed.com/shop?query={q}` | 英語 | `grailed\.com/listings/` |

## 抽出スクリプト（Playwright）
ページを開いたら、3秒ほど待ってから下にスクロールし（`window.scrollBy` を数回）、`page.evaluate(ex, パターン)` を実行します。
`mouse.wheel` は使わないでください。ページが表示されていないと止まることがあります。

```js
const ex = (pat) => {
  const seen = new Set(), out = [];
  for (const a of document.querySelectorAll('a[href]')) {
    const url = a.href.split('?')[0];
    if (!new RegExp(pat).test(url) || seen.has(url)) continue;
    let box = a, img = a.querySelector('img');
    for (let i = 0; i < 4 && (!img || !/[¥$円]/.test(box.innerText || '')); i++) { box = box.parentElement || box; img = img || box.querySelector('img'); }
    if (!img) continue;
    const txt = (box.innerText || '').replace(/\s+/g, ' ').trim();
    const price = (txt.match(/[¥$€£]\s?[\d,]+(?:\.\d+)?|[\d,]+\s?円/) || [''])[0];
    const title = (img.alt || a.getAttribute('aria-label') || txt).slice(0, 80);
    const image = img.currentSrc || img.src || img.dataset.src || '';
    if (!/^https?:/.test(image)) continue;
    seen.add(url); out.push({ title, price, image, url });
  }
  return out;
};
```

- 出力が大きいときは、結果を `BONDJSON` と `BONDEND` で挟んで返し、保存されたファイルから取り出す。
- サムネイルを目で確認するときは、候補の画像を番号付きで並べた一覧をページに表示して、スクリーンショットを撮ると速い。
- Grailed の価格は、値下げ前と値下げ後の2つが並ぶことがある。その場合は価格の後ろに「（目安）」を付ける。
- Vinted はクッキーのバナーが出るが、閉じなくても抽出できる。

## 取込コードの形式
```json
{
  "savedAt": "2026-11-15T23:30:00+09:00",
  "batch": 1,
  "item": { "name": "", "brand": "", "category": "", "era": "", "details": "", "checkpoints": "" },
  "keywords": [ { "ja": "", "en": "", "note": "" } ],
  "design":   [ { "ja": "", "en": "", "note": "" } ],
  "similar":  [ { "ja": "", "en": "", "note": "" } ],
  "resultsMeta": "YYYY/MM/DD 調べ／元の商品：…／注意点",
  "results":  [ { "site": "メルカリ", "title": "", "price": "¥13,000", "image": "https://…", "url": "https://…", "match": "どこが似ているか（短く）" } ]
}
```
- `site` の値は、上の表のサイト名と同じ表記にする（アプリの絞り込みボタンで使う）。
- `match` には、見た目のどこが共通しているかを1行で書く（例：「ボア襟×チェック×ジップ。配色が近い」）。
- `checkpoints` には、仕入れ前に確認すべき点を書く（ダメージ、サイズ、素材、送料、関税など）。
