---
title: "YouTube埋め込みで「エラー153」が出たときの原因と解決策"
emoji: "📺"
type: "tech"
topics: ["YouTube", "iframe", "HTML", "エラー153", "Web"]
published: false
---

## はじめに

Webサイトに埋め込んでいたYouTube動画が、ある日突然「**エラー153 動画プレーヤーの設定エラー**」と表示され、再生できなくなりました。

![エラー153が表示された状態](/images/youtube-153error-before.png)
*埋め込み動画にエラー153が表示され、再生できない状態*

同じエラーに遭遇した方に向けて、原因と解決策を共有します。

## 結論：埋め込みコードを再取得するだけで解決

YouTubeの「共有」→「埋め込む」から**最新の埋め込みコードを再取得して差し替える**ことで解決します。

![正常に再生できる状態](/images/youtube-153error-after.png)
*最新の埋め込みコードに差し替え後、正常に再生できるようになった*

動画の「共有」を開き、「埋め込む」を選択します。

![共有画面で「埋め込む」を選択](/images/youtube-share-1.png)
*「共有」→「埋め込む」を選択*

表示された埋め込みコードをコピーして差し替えます。

![埋め込みコードの取得画面](/images/youtube-share-2.png)
*最新の埋め込みコードが表示される*

以下、原因の詳細を解説します。

## 原因：YouTube側がリファラーポリシーを厳格化した

エラー153の正体は、**YouTubeの埋め込みプレーヤーがHTTPリファラー（参照元情報）を受け取れない**ことで発生するエラーです。

### 公式ドキュメントの記載

YouTube API の公式ドキュメント「[Required Minimum Functionality](https://developers.google.com/youtube/terms/required-minimum-functionality)」に、以下の記載があります。

> API Clients that use the YouTube embedded player (including the YouTube IFrame Player API) must provide identification through the `HTTP Referer` request header.
>
> YouTube recommends using `strict-origin-when-cross-origin` Referrer-Policy, which is already the default in many browsers.

つまり、YouTube側は埋め込みプレーヤーに対して**HTTPリファラーによる識別を必須**としており、推奨するリファラーポリシーは `strict-origin-when-cross-origin` です。

### なぜ突然エラーが出るようになったのか

以前はリファラーが送信されなくても再生できていましたが、YouTube側がこの要件を**厳格に適用するようになった**ため、古い埋め込みコードや、リファラーポリシーが `no-referrer` や `same-origin` に設定されている環境でエラー153が発生するようになりました。

エラー画面を右クリックして「デバッグ情報をコピー」すると、以下のようなエラーコードが確認できます。

```
"debug_error": "{\"errorCode\":\"embedder.identity.missing.referrer\""
```

`missing.referrer`（リファラーが欠落）と明示されており、リファラーポリシーが原因であることが確認できます。

## 解決策

### iframe に `referrerpolicy` 属性を追加する

最もシンプルな対処は、iframe タグに `referrerpolicy="strict-origin-when-cross-origin"` を追加することです。

```diff
  <iframe
    src="https://www.youtube.com/embed/VIDEO_ID"
    title="YouTube video player"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
+   referrerpolicy="strict-origin-when-cross-origin"
    allowfullscreen
  ></iframe>
```

### 最新の埋め込みコードに差し替える（推奨）

YouTubeの「共有」→「埋め込む」から取得できる最新のコードには、`referrerpolicy` が最初から含まれています。古い埋め込みが複数ある場合は、一括で差し替えるのが確実です。

変更前（エラー153が発生）：

```html
<iframe
  src="https://www.youtube.com/embed/VIDEO_ID"
  title="YouTube video player"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen
></iframe>
```

変更後（正常に再生可能）：

```html
<iframe
  src="https://www.youtube.com/embed/VIDEO_ID?si=XXXXXXXXXXXX"
  title="YouTube video player"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  referrerpolicy="strict-origin-when-cross-origin"
  allowfullscreen
></iframe>
```

変更点まとめ：

| 項目 | 変更前 | 変更後 |
|---|---|---|
| `referrerpolicy` | なし | `strict-origin-when-cross-origin` |
| `allow` | `web-share` なし | `web-share` 追加 |
| `src` | `/embed/VIDEO_ID` | `/embed/VIDEO_ID?si=XXXX` |
| `frameborder` | なし | `"0"` |

:::message
`si` パラメータは共有元のトラッキング用識別子で、エラー153の直接の原因ではありません。エラー解消の本質は `referrerpolicy` の追加です。
:::

## おわりに

YouTubeの埋め込みプレーヤーは、HTTPリファラーによる識別が必須となっています。エラー153が出た場合は、`referrerpolicy="strict-origin-when-cross-origin"` が設定されているか確認してみてください。

## 参考

- [Required Minimum Functionality - YouTube API Services](https://developers.google.com/youtube/terms/required-minimum-functionality) — リファラー要件の公式ドキュメント
- [Fixing YouTube Error 153 — Simon Willison's TILs](https://til.simonwillison.net/youtube/fixing-153-embed) — デバッグ情報を含む詳細な調査記事
