# FFL7

FFL7の文法などを記述します。

## ディレクトリ構成

```
my-project/
├── .gitignore           # Git 管理に含めたくないファイルを記述
├── README.md            # 概要
└── docs/                # GitHub Pages の公開対象フォルダ
    ├── index.md         # サイトのトップページ
    ├── assets/          # サイトに使用するスタイルシート、スクリプト、画像など
    │  ├── css/
    │  └── images/
    ├── pages
    └── _posts           # 残骸
```

## build

githubpagesを利用する場合は、`_config.yml`の`theme`をコメントアウトし、`remote_theme`のコメントアウトを外す。
