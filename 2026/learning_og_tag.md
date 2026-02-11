今回はリンクカードの実装に必要な**OGタグ**について学習しました。

## 概要
### リンクカード
URLをSNSやチャットに貼った時に自動で表示される画像つきプレビューのこと。  
それ専用の実装は必要なく、`<head>`のmeta情報を読み込みSNSが自動で作成するものです。  
リンクカードを正しく表示するために必要なものが**OGタグ**です。

### OGタグ
#### OGP
OGとは、正式にはOGP（Open Graph Protocol）のことでwebページをSNSでも理解できる共通のフォーマットで表現するためのしくみのことです。  
2010年にFacebookが提唱し、webページをオブジェクトとして扱えるようにしたルール（Protocol）になります。  

#### 設定できるもの
主に主に設定できるものは以下の3つです。
- タイトル
- 説明文
- サムネイル画像
他にも細かいものでいうとページ種別や正規URLなども設定することができます。

#### 基本構造
OGタグはHTMLのhead内にmetaタグとして書きます。  
property属性に種類を指定して、content属性に内容を記述します。
```
<head>
  <meta property="og:..." content="...">
</head>
```

## 種類
### 基本OGタグ
#### `og:title`
ページのタイトル。リンクカードの見出しになります。

#### `og:description`
ページの説明文。サブテキストとして表示させることができます。

#### `og:type`
ページの種類。article / websiteなどを指定できます。

#### `og:url`
正規URL。どのURLが本物かを示します。

#### `og:site_name`
サイト名。カードの補助情報として指定できます。

### 画像関連OGタグ
#### `og:image`
画像URL。サムネイル画像などはここで設定できます。

#### `og:iamge_secure_url`
HTTPS画像URL。安全なURLを指定できます。

#### `og:image:type`
MIMEタイプ。image/pngなどを指定できます。

#### `og:image:width`
画像幅を指定できます。

#### `og:image:height`
画像高さを指定できます。

#### `og:image:alt`
代替テキストを指定できます。

### タイプの拡張タグ
UIには直接関係しませんがメタデータとして設定できる拡張タグがあります。  
今回は詳細は省略しますので参考程度にまとめておきます。

#### `article:...`
ブログやニュース記事で使用します。

#### `profile:...`
人物ページで使用します。

#### `og:video:...`
動画で使用します。

### SNS用の特有タグ
#### X（旧Twitter）
- `twitter:card`（カードタイプ）
- `twitter:title`（タイトル）
- `twitter:description`（説明文）
- `twitter:iamge`（画像URL）

など

#### Facebook
- `fb:app_id`（FacebookアプリID）
- `fb:admins`（管理者ID）
など

## Next.jsでの書き方
App Routerを前提としてまとめます。

Next.jsでは、`<meta />`を直接書く必要はなく`export const metadata`または`generalMetadata()`を使用して設定します。  
Next.jsが自動で`<head>`にOGタグを生成します。

### 静的ページ
トップの`page.tsx`、または`layout.tsx`に書きます。  
JSON形式で`openGraph`の値として設定します。

```
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "トップページ",
  description: "サイトの説明文です",

  openGraph: {
    title: "トップページ",
    description: "サイトの説明文です",
    url: "https://example.com",
    siteName: "My Site",
    type: "website",
    images: [
      {
        url: "https://example.com/ogp.png",
        width: 1200,
        height: 630,
        alt: "サイトのOG画像",
      },
    ],
  },

  twitter: {
    card: "summary_large_image",
    title: "トップページ",
    description: "サイトの説明文です",
    images: ["https://example.com/ogp.png"],
  },
};

```

### 動的ページ
ブログなどのパスによって内容が変わる場合は、`app/[slug]/page.tsx`などで以下のように書きます。
```
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug);

  const url = `https://example.com/articles/${post.slug}`;

  return {
    title: post.title,
    description: post.excerpt,

    openGraph: {
      title: post.title,
      description: post.excerpt,
      url,
      type: "article",
      images: [
        {
          url: post.ogImage,
          width: 1200,
          height: 630,
        },
      ],
    },

    twitter: {
      card: "summary_large_image",
      title: post.title,
      description: post.excerpt,
      images: [post.ogImage],
    },
  };
}

```

### metadataBase
```
metadataBase: new URL("https://example.com")
```
これを設定すると画像指定の際に相対パスを使えるようになります。

### OGP画像の自動生成
Next.jsは`opengraph-image.tsx`を作るだけで自動でOG画像を生成することができます。  
このファイルではHTMLを返すのではなく`ImageResponse` を返します。
```
// app/[slug]/opengraph-image.tsx

import { ImageResponse } from "next/og";

export const size = {
  width: 1200,
  height: 630,
};

export const contentType = "image/png";

export default async function Image({ params }) {
  const post = await getPost(params.slug);

  return new ImageResponse(
    (
      <div
        style={{
          width: "100%",
          height: "100%",
          display: "flex",
          backgroundColor: "#111",
          color: "white",
          padding: "80px",
          fontSize: 60,
          fontWeight: "bold",
        }}
      >
        {post.title}
      </div>
    ),
    {
      ...size,
    }
  );
}

```

## まとめ
webサイトやブログを共有する際にはリンクカードがあると、より興味関心をひくことができるでしょう。  
OGタグをしっかりと設定することで適切なリンクカードを作成できるようになりましょう！
