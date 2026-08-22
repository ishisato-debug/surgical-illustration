# surgical-illustration

大学講義「デジタルイラストで手術記録を描く」の配布資料サイト。
講義スライドの最後に QR コードを出し、学生がスマホでその場から開く用途。

- 公開URL: https://ishisato-debug.github.io/surgical-illustration/
- ホスティング: GitHub Pages（`main` ブランチの `/ (root)`）
- 受講者は学内外の学生・研修医。**外部大学からの参加があるため、認証は掛けない**

## 構成

```
.
├── index.html   ページ本体。CSS・JS を内包した1ファイル完結
├── handout.pdf  ダウンロードボタンの実体（全13ページ）
└── images/
    ├── slide-01.jpg 〜 slide-13.jpg           表示用（長辺1600px, JPEG q82）
    └── slide-01-full.jpg 〜 slide-13-full.jpg 拡大用（長辺2600px, JPEG q88）
```

`index.html` のセクションと元スライドの対応：

| セクション | スライド | 見出し |
| --- | --- | --- |
| `#basics` | 1〜10 | デジタルイラストの基本 |
| `#example` | 11〜12 | Procreateで描いたオペレコの例 |
| `#refs` | 13 | おすすめ資料 |

## 設計上の判断（変更する前に読む）

- **画像2種持ちは意図的**。表示用だけ先に読み込み、拡大用はライトボックスを開いた時に初めて取得する。講義室で100人が同時アクセスしても詰まらないようにするため。初回読み込みは約0.5MB
- **`loading="lazy"`** で画面外の画像は遅延読み込み。これも同じ理由
- **拡大は2段階**（全画面 → タップで2.5倍）。スライド11・12は血管名などの細字があり、等倍表示では読めないため。`.lb.zoom img { width:250% }` の数値がその2.5倍
- **PDF はページ内表示せずダウンロードのみ**。スマホで PDF を開くと別アプリに遷移して講義に戻ってこられなくなるため
- **ダークモード対応済み**だが、スライド画像は白いまま加工していない。暗くすると細字が読めなくなるため
- フォントは Google Fonts の Zen Kaku Gothic New。読み込み失敗時も `font-display: swap` でフォールバックが即表示される

## PDF を差し替えたときの画像再生成

`handout.pdf` を新しいものに置き換えたら、`images/` も必ず作り直す。
ページに出ているのは PDF から書き出した画像なので、PDF だけ差し替えても表示は古いまま。

```bash
# 必要: poppler-utils (pdftoppm), ImageMagick (convert)
mkdir -p images && cd images
pdftoppm -r 200 -png -aa yes -aaVector yes ../handout.pdf raw

for f in raw-*.png; do
  n="${f#raw-}"; n="${n%.png}"
  convert "$f" -resize 1600x -background white -alpha remove -alpha off \
    -quality 82 -sampling-factor 4:2:0 -strip -interlace Plane "slide-${n}.jpg"
  convert "$f" -resize 2600x -background white -alpha remove -alpha off \
    -quality 88 -strip "slide-${n}-full.jpg"
done
rm raw-*.png
```

ページ数が13から変わる場合は `index.html` の `<figure class="slide">` ブロックも増減させ、
各カードの `data-full` / `data-cap` / `alt` を合わせること。

## 未確定・要確認

- **フッターの講義名・日付・氏名が未記入**。`index.html` 末尾の `<!-- ▼ここを編集 -->` の箇所
- **`<h1>` の「手術イラストの描き方」は仮タイトル**。正式な講義名が決まったら差し替える

## 注意

- リポジトリ名を変えると公開URLが変わり、**配布済みのQRコードが無効になる**。改名しない
- `handout.pdf` のファイル名は変えない。`index.html` の2箇所からリンクしている
- 資料を差し替えてもURLは変わらないので、QRコードは翌年以降も使い回せる
