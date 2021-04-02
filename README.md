# BeautifulSoup チートシート

Python で HTML を扱うためのライブラリ BeautifulSoup （ `beautifulsoup4` ）のチートシートです。

- [`beautifulsoup4` · PyPI](https://pypi.org/project/beautifulsoup4/)
- [Beautiful Soup: We called him Tortoise because he taught us.](https://www.crummy.com/software/BeautifulSoup/)

公式のドキュメントは充実していますが内容が盛りだくさんなので、必要な情報にたどり着くのに時間がかかりがちです。
BeautifulSoup をときどき使う人（私）が使い方をサッと確認したいときに便利なチートシートです。

Python 3 を前提としています。

- BeautifulSoup をインストールする
- BeautifulSoup を読み込む
- ページタイトルなどのメタ情報を取得する
- 要素を検索する
- 要素の情報を取得する

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

`BeautifulSoup` オブジェクトを直接 callable として実行すると `find_all()` と同じ挙動をします。

特定のタグ（複数）の要素をすべて取得する:

```python
soup.find_all(["a", "link"])
# または
soup(["a", "link"])
```
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

## 要素の情報を取得する

タグ名を取得する:

```python
tag.name
```

属性を取得する:

```python
# id
tag["id"]
# => "id-abc"

# class
tag["class"]
# => ["class-a", "class-b"]

# その他属性
tag["href"]

# すべての属性
tag.attrs
# =>
# {
#   "aria-current": "page",
#   "class": ["styles-module--site-title-link--9ES5k"],
#   "href": "/",
# }
```

デフォルトでは `class` などの一部の属性は自動的に `list` を返します。
文字列を返すようにしたい場合は `BeautifulSoup` オブジェクトの生成時に `multi_valued_attributes=None` を指定します。

```python
soup = BeautifulSoup(html, "html.parser", multi_valued_attributes=None)

tag = soup.find("a")
tag["class"]
# => "class-a class-b"
```

内部のテキストを取得する:

```python
# 内部の要素が 1 件のみの場合
tag.string
# => " こちら "

# 内部の要素が複数の場合
[x for x in tag.strings]
# => [" こちら ", " あちら "]

# 内部の要素が複数の場合・先頭・末尾のスペースを削除済み
[x for x in tag.stripped_strings]
# => ["こちら", "あちら"]
```

属性を持つかどうかをチェックする:

```python
tag.has_attr("class")
```

## 関連要素を取得する

子要素を取得する:

`tag.contents` または `tag.children` を使います。
`tag.children` はジェネレーターです。

```python
tag.contents
# => [...]

[child for child in tag.children]
# => [...]

tag.contents[0]
# => ...

tag.children[0]
# TypeError: 'list_iterator' object is not subscriptable
```

親要素を取得する:

`tag.parent` または `tag.parents` を使います。
`tag.parents` はジェネレーターです。

```python
# 直接の親を取得する
tag.parent

# すべての親を下から上にさかのぼる
[x for x in tag.parents]

# 例:
[x.name for x in soup.title.parents]
# => ["head", "html", "[document]"]

# 最上位の要素は `BeautifulSoup` オブジェクトそのもの
type([x for x in soup.title.parents][-1])
# => <class 'bs4.BeautifulSoup'>
```

兄弟・姉妹要素を取得する:

`tag.next_sibling` `tag.previous_sibling` `tag.next_siblings` `tag.previous_siblings` を使います。

```python
# 次の要素を取得する
tag.next_sibling

# 前の要素を取得する
tag.previous_sibling

# 後の要素を順に取得する
[x for x in tag.next_siblings]

# 前の要素を順に（後ろから前に）取得する
[x for x in tag.previous_siblings]
```

これらと似て非なる機能を持つものとして次のアトリビュートもあります。

- `tag.next_element`
- `tag.previous_element`
- `tag.next_elements`
- `tag.previous_elements`
