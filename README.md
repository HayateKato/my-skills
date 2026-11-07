# my-skills

Claude Code で使う個人用のスキルと `CLAUDE.md` を管理するリポジトリ。
スキルはプラグインとして配布し、`CLAUDE.md` はシンボリックリンクで各環境に置く。

## 構成

```
my-skills/
├── .claude-plugin/
│   └── marketplace.json      # マーケットプレイスの定義
├── plugins/
│   └── my-skills/            # プラグイン本体
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
└── instructions/
    └── CLAUDE.md             # 全環境共通の指示
```

## 前提

- Claude Code がインストールされている。

## 導入手順

### ローカル（編集・push する環境）

1. リポジトリを書き込みできる形で clone する。

   ```bash
   git clone git@github.com:HayateKato/my-skills.git ~/my-skills
   ```

2. プラグインを入れる。

   ```bash
   claude plugin marketplace add HayateKato/my-skills
   claude plugin install my-skills@my-marketplace
   ```

   Claude デスクトップアプリから SSH 先に接続して使う場合も、プラグインはローカルに入れるだけでよい。
   プラグインは接続のたびに SSH 先へ転送される。

3. `~/.claude/CLAUDE.md` から clone へのシンボリックリンクを張る。
   clone を直接参照するので、編集した内容はすぐに反映される。

   ```bash
   mkdir -p ~/.claude
   # 既に ~/.claude/CLAUDE.md がある場合は、先に退避する
   [ -e ~/.claude/CLAUDE.md ] && mv ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.bak
   ln -s ~/my-skills/instructions/CLAUDE.md ~/.claude/CLAUDE.md
   ```

4. 新しいセッションで `/my-skills:` と入力してスキルが一覧に出て、「CLAUDE.md の内容を要約して」と聞いて内容が返ってくれば完了である。

### リモート（SSH 先）

1. リポジトリを HTTPS で clone する。

   ```bash
   git clone https://github.com/HayateKato/my-skills.git ~/my-skills
   ```

2. `~/.claude/CLAUDE.md` から clone へのシンボリックリンクを張る。

   ```bash
   mkdir -p ~/.claude
   # 既に ~/.claude/CLAUDE.md がある場合は、先に退避する
   [ -e ~/.claude/CLAUDE.md ] && mv ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.bak
   ln -s ~/my-skills/instructions/CLAUDE.md ~/.claude/CLAUDE.md
   ```

3. `CLAUDE.md` を最新に保つため、ログイン時に自動で pull する。
   `~/.bashrc`（zsh なら `~/.zshrc`）に次を追記する。

   ```bash
   ( git -C "$HOME/my-skills" pull --ff-only -q >/dev/null 2>&1 & )
   ```

   自動で pull されたか確認するため、現在のセッションで git ファイルの最終更新日時を確認する。

   ```bash
   ls -l ~/my-skills/.git/FETCH_HEAD
   ```

   その後、SSHログインし直して再度上記コマンドを実行し、最終更新日時が先程の日時よりも進んでいれば完了である。

## 更新手順

スキルと `CLAUDE.md` はセッション開始時に読み込まれる。
変更は、更新したあとに始めたセッションから反映される。

### スキルを追加・変更する

1. `plugins/my-skills/skills/<スキル名>/SKILL.md` を作成・編集する。
   新しく作る場合は、先頭のフロントマターに `name` と `description` を書く。
   `description` が空だとスキルが自動で呼ばれないので、「いつ使うか」を具体的に書く。

   ```markdown
   ---
   name: <スキル名>
   description: <どんなときに使うか>
   ---
   ```

2. 試す場合は、インストールせずに直接読み込ませる。

   ```bash
   claude --plugin-dir ./plugins/my-skills
   ```

3. commit して push する。

### CLAUDE.md を変更する

`instructions/CLAUDE.md` を編集して push する。
各 SSH 先では、次にログインしたときに自動で pull される。

すぐに反映させたい場合は、SSH 先で手動で pull する。

```bash
git -C ~/my-skills pull
```
