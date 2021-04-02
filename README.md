# BeautifulSoup チートシート

Python で HTML を扱うためのライブラリ BeautifulSoup （ `beautifulsoup4` ）のチートシートです。

公式のドキュメントは充実していますが内容が盛りだくさんなので、必要な情報にたどり着くのに時間がかかりがちです。
BeautifulSoup をときどき使う人（私）が使い方をサッと確認したいときに便利なチートシートです。

Python 3 を前提としています。

## BeautifulSoup をインストールする

```bash
python -m pip install beautifulsoup4
```

## BeautifulSoup を読み込む

```python
from bs4 import BeautifulSoup
```

以下のコードではこの `import` 文が事前に実行されているものとします。

## HTML を読み込む

文字列（ `str` ）から読み込む:

```python
html = "<!DOCTYPE html><html><body><p>Hello</p></body></html>"
soup = BeautifulSoup(html, "html.parser")
```

ファイルから読み込む:

```python
with open("./index.html") as f:
    html = f.read()

soup = BeautifulSoup(html, "html.parser")
```

ウェブから読み込む（ `requests` を使う場合）:

```python
import requests

res = requests.get("https://example.com")
assert res.status_code == 200
assert res.encoding.upper() == "UTF-8"
html = res.text

soup = BeautifulSoup(html, "html.parser")
```

標準入力から読み込む:

```python
import sys

html = sys.stdin.read()
soup = BeautifulSoup(html, "html.parser")
```

## ページタイトルなどのメタ情報を取得する

```python
soup = BeautifulSoup(html, "html.parser")

# 名前
soup.name
# => [document]

# ページタイトル
soup.title
# または
soup.head.title

# meta charset
soup.head.find("meta", attrs={"charset": True})["charset"]

# meta viewport
soup.head.find("meta", attrs={"name": "viewport"})["content"]

# meta description
# 存在しない場合は空文字列とする
description = ""
if tag := soup.head.find("meta", attrs={"name": "description"}):
    description = tag["content"]

# meta keywords
# 存在しない場合は空文字列とする
keywords = ""
if tag := soup.head.find("meta", attrs={"name": "keywords"}):
    keywords = tag["content"]

# og:title
# 存在しない場合は空文字列とする
og_title = ""
if tag := soup.head.find("meta", attrs={"property": "og:title"}):
    og_title = tag["content"]
```

## 要素を検索する

特定のタグの要素のうち最初に見つかったものを取得する:

アトリビュート `soup.タグ名` で検索できます。

```python
# <head> を取得する
soup.head

# <title> を取得する
soup.title

# <head> 内の <title> を取得する
soup.head.title

# <body> を取得する
soup.body

# <a> を取得する
soup.a
```

特定の属性を持つ最初の要素を取得する:

`soup.find()` を使います。

```python
soup.find("a", target="_blank")
```

特定のタグの要素をすべて取得する:

`soup.find_all()` または `soup()` を使います。

```python
soup.find_all("a")
# または
soup("a")
```

`soup` を直接 callable として実行すると `find_all()` と同じ挙動になります。

特定のタグの要素を最大 3 件取得する:

キーワード引数 `limit` を使います。

```python
soup.find_all("a", limit=3)
```

特定のテキストを中身に持つ要素をすべて取得する:

キーワード引数 `string` を使います。

```python
soup.find_all("a", string="サイトについて")
```

子要素を再帰的に探すのをやめて直接の子のみを探す:

キーワード引数 `recursive` を使います。

```python
soup.find_all("a", recursive=False)
```

`recursive` を指定しなかった場合のデフォルトは `recursive=True` です。

特定の属性を持つ要素をすべて取得する:

```python
soup.find_all("a", href=True)
# または
soup.find_all("a", attrs={"href": True})
```

特定の属性と値を持つ要素をすべて取得する:

```python
soup.find_all("a", target="_blank")
# または
soup.find_all("a", attrs={"target": "_blank"})
```

特定のクラスを持つ要素をすべて取得する:

`class` は予約語なので、キーワード引数で指定する場合は代わりに `class_` を使います。

```python
soup.find_all("a", class_="external")
# または
soup.find_all("a", attrs={"class": "external"})
```

特定の正規表現パターンにマッチするものを取得する:

```python
import re

soup.find_all("a", href=re.compile(r"^/"))
```

任意の条件にマッチするものをすべて取得する:

```python
soup.find_all(lambda x: x.name in ["img", "iframe"])
```

CSS セレクタ形式で探す:

`soup.select()` を使います。

```python
soup.select("img#feature")
soup.select("img.attachment")
soup.select("img[alt]")
```

## タグの情報を取得する

```python
tag.name

tag["id"]
# => "id-abc"

tag["class"]
# => ["class-a", "class-b"]

tag.attrs
# =>
# {
#   "aria-current": "page",
#   "class": ["styles-module--site-title-link--9ES5k"],
#   "href": "/",
# }

tag.text
# => ' こちら '
```

デフォルトでは `class` などの一部の属性は自動的に `list` を返します。
文字列を返すようにしたい場合は `multi_valued_attributes` で指定します。

```python
soup = BeautifulSoup(html, "html.parser", multi_valued_attributes=None)

tag = soup.find("a")
tag["class"]
# => "class-a class-b"
```
