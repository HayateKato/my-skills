# my-skills

Claude Code と Codex で使う個人用のスキルと共通指示を管理するリポジトリ。
Claude Code ではスキルをプラグインとして導入し、Codex ではスキルへのシンボリックリンクを置く。
`CLAUDE.md` と `AGENTS.md` はシンボリックリンクで各環境に置く。

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
    ├── CLAUDE.md             # Claude Code 用の共通指示
    └── AGENTS.md             # Codex 用の共通指示
```

## 前提

以下がローカルにインストールされている。

- Claude Code
- Claude デスクトップアプリ
- Codex CLI または Codex アプリ

## 導入手順

### Claude Code：ローカル（編集・push する環境）

1. リポジトリを clone する。

   ```bash
   git clone https://github.com/HayateKato/my-skills.git ~/my-skills
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

### Claude Code：リモート（SSH 先）

1. リポジトリを clone する。

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

### Codex：ローカル（編集・push する環境）

以下は `~/work/my-skills` に clone した場合の手順である。
別の場所に clone している場合は、パスを読み替える。

1. リポジトリを clone する。
   既に clone 済みの場合は、この手順を省略する。

   ```bash
   mkdir -p ~/work
   git clone https://github.com/HayateKato/my-skills.git ~/work/my-skills
   ```

2. `~/.codex/AGENTS.md` から clone へのシンボリックリンクを張る。
   clone を直接参照するので、編集した内容はすぐに反映される。

   ```bash
   mkdir -p ~/.codex
   # 既に ~/.codex/AGENTS.md がある場合は、先に退避する
   [ -e ~/.codex/AGENTS.md ] && mv ~/.codex/AGENTS.md ~/.codex/AGENTS.md.bak
   ln -s ~/work/my-skills/instructions/AGENTS.md ~/.codex/AGENTS.md
   ```

3. 各スキルへのシンボリックリンクを `~/.codex/skills/` に置く。
   同名のファイル・ディレクトリ・リンクがある場合は、未使用の名前に退避してから実行する。

   ```bash
   mkdir -p ~/.codex/skills
   for skill in "$HOME/work/my-skills/plugins/my-skills/skills/"*; do
     skill_name="${skill##*/}"
     skill_link="$HOME/.codex/skills/$skill_name"
     if [ -e "$skill_link" ] || [ -L "$skill_link" ]; then
       printf '既存のためスキップ: %s\n' "$skill_link"
       continue
     fi
     ln -s "$skill" "$skill_link"
   done
   ```

4. 新しい Codex セッションで「共通の AGENTS.md の内容を要約して」と聞いて内容を確認する。
   Codex CLI では `/skills` または `$` から `python-rules` などが一覧に出ることも確認する。

## 更新手順

共通指示の変更は、更新したあとに始めたセッションから反映される。
スキルを更新した場合も、新しいセッションで動作を確認する。

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

   Codex では導入手順のシンボリックリンクから clone 内のスキルを直接参照する。
   新しいスキルを追加した場合は、導入手順3を再実行してリンクを追加する。

3. commit して push する。

### CLAUDE.md・AGENTS.md を変更する

Claude Code 用は `instructions/CLAUDE.md`、Codex 用は `instructions/AGENTS.md` を編集して push する。
Codex はローカルの clone を直接参照するため、新しいセッションから変更が反映される。
Claude Code 用の `CLAUDE.md` は、各 SSH 先で次にログインしたときに自動で pull される。

`CLAUDE.md` をすぐに反映させたい場合は、SSH 先で手動で pull する。

```bash
git -C ~/my-skills pull
```
