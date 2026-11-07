---
name: python-rules
description: Python コードを新規作成・変更するときや、Python を実行・パッケージをインストールするときに使う。.venv を使った実行環境の扱いと、日本語の Google-style docstring の書き方を定める。
---

## Python 実行環境

- Python を実行する前に、カレントディレクトリに `.venv/` が存在するか確認する
- `.venv/` があれば `.venv/bin/python` を使う
- `.venv/` がなければ `python3 -m venv .venv` で作成してから `.venv/bin/python` を使う
- `pip install` も必ず `.venv/bin/pip` で行い、システムの `python3` / `pip` は使わない

## Python docstring

Python コードの新規作成・変更に適用する。

- 新規作成または変更するモジュール、クラス、関数には、日本語の Google-style docstring を記述する。
- モジュールには、その責務を示す docstring を記述する。
- クラスには概要と `Attributes:` を記述する。
- 関数には概要を記述し、引数があれば `Args:`、戻り値があれば `Returns:` を記述する。
- 型情報は docstring 内にも明記し、実装の型ヒントと一致させる。