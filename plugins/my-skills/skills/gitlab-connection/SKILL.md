---
name: gitlab-connection
description: GitLab API へ接続する方法を示す。glab や GitLab API で Issue 作成・更新・push・マージリクエスト作成などを行う前に、接続先ホストと対象プロジェクトを確認するときに使う。
---

# GitLab 接続

GitLab API アクセスに適用する。接続方法のみを扱い、Issue 作成・push・マージリクエスト作成の承認は、呼び出し元のスキルの条件に従う。

## 接続先の確認

- 対象リポジトリの CLAUDE.md・AGENTS.md、Git リモート、利用可能な接続設定から、GitLab のホスト、Web/API のベースパス、対象プロジェクトを確認する。
- SSH リモートと Web/API のホスト・ポート・パスが一致するとは限らない。SSH URL から Web/API の接続先を決めつけない。
- 接続先とプロジェクト ID をスキルに固定せず、対象リポジトリごとに確認する。他のリポジトリの設定を流用しない。
- 接続先や対象プロジェクトを一意に特定できない場合は、書き込みを行う前にユーザーに確認する。

## API の呼び出し方

- リモートの自動判定が適切に働かない構成では、確認した接続先を `GITLAB_HOST` に指定し、`glab api` で `projects/<project-id>` を明示する。
  - `glab mr create` などの高レベルコマンドは、自動判定に失敗しても目立つエラーを出さずに終わることがある。その場合は `glab api` で作成する。
- Markdown の説明文など長い本文は、ファイルに書いてから JSON にして `--input` で送る。`-f` で渡すと改行や記号の扱いで崩れることがある。

  ```bash
  python3 -c 'import json,sys;print(json.dumps({"title":sys.argv[1],"description":open(sys.argv[2]).read()}))' "<タイトル>" body.md > body.json
  glab api -X POST "projects/<project-id>/issues" -H "Content-Type: application/json" --input body.json
  ```

- 書き込んだ後は、返ってきた `iid`・`web_url` と、本文が送ったものと一致するかを確認する。

よく使うエンドポイント（すべて `projects/<project-id>/` の下）:

| 操作 | メソッドとパス |
| --- | --- |
| Issue の一覧・取得 | `GET issues?state=opened`、`GET issues/<iid>` |
| Issue の作成・更新 | `POST issues`、`PUT issues/<iid>`（閉じるときは `state_event=close`） |
| Issue へのコメント | `POST issues/<iid>/notes` |
| Issue の関連付け | `POST issues/<iid>/links`（`target_project_id`、`target_issue_iid`、`link_type`: `relates_to` / `blocks` / `is_blocked_by`） |
| ファイルの添付 | `POST uploads`（返ってきた `markdown` をコメント本文に貼る） |
| MR の作成・取得 | `POST merge_requests`、`GET merge_requests/<iid>` |
| MR のパイプライン | `GET merge_requests/<iid>/pipelines` |
