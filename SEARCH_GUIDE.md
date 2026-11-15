# 仕入れ検索 手順書（AIチャット用）

このファイルは、BOND Vintage の仕入れアプリ用の「取込コード」を作るときの手順書です。
**安いモデル（例：Sonnet）のチャットでも、この手順だけで同じ作業ができる**ように書いています。

## 依頼のしかた（ユーザー側）
チャットに商品画像を貼り、次のように送ります。

> `C:\Users\advke\OneDrive\BondVintage_Shiire_App\SEARCH_GUIDE.md` の手順で取込コードを作って

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
   - **上限は100件**。サイトごとに偏りすぎないようにし、近い順に並べる。
5. **取込コードを出す**：下の形式のJSONを1つだけ、コードブロックで返す。そのあとに、注目の3件と注意点を短く添える。

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
