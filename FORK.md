# この fork の運用

`numachang/sendo` は、`sendo-feed/sendo` を Cloudflare にデプロイして使うための fork。
上流の開発用の PR・マージ手順とは別に、更新を自動で取り込む。

## 上流への自動追従

上流の `main` に変更が入ると、数分のうちに
[Sync upstream](https://github.com/numachang/sendo/actions/workflows/sync-upstream.yml) が、
上流の `main` をこの fork の `main` に取り込む。GitHub の「Sync fork」と同じ API を使い、
独自コミットを残す。競合した場合は失敗し、上書きしない。
取り込みで `main` が更新されると、接続済みの Workers Builds がマイグレーションとデプロイを行う。

Sync upstream を起動するのは、Cloudflare の Worker
[fork-sync-cron](https://github.com/numachang/fork-sync-cron)。毎分、上流との差分を調べ、
遅れていれば workflow_dispatch で起動する。GitHub Actions の `schedule` は使わない
（定刻の実行が作られないことがあったため。経緯は fork-sync-cron の README）。

reader の fork（`numachang/sendo-reader`）は、この fork が上流に追いつき、`main` の先頭で
`Workers Builds: sendo` が成功してから取り込む。この fork のデプロイが失敗している間は、
reader の自動の同期も止まる。

- 手動で動かすには、Actions の「Sync upstream」から「Run workflow」を選ぶ。
- 取り込みの認証には、自動発行される `GITHUB_TOKEN`（`contents: write`）を使う。
  Worker が起動に使うトークンは、fork-sync-cron の README を見る。
- このワークフローは `numachang/sendo` でだけ動く。
- 自動の同期を止めるには、Actions の「Sync upstream」から「Disable workflow」を選ぶ。
  Worker は無効なワークフローを起動しない。
- 失敗したら Actions のログを確認する。競合は手動で解消してから、もう一度動かす。
  Worker は、上流に新しいコミットが来るか1時間たつまで、失敗した同期を起動し直さない。
- 上流が進んでいるのに同期が起動されなければ、fork-sync-cron の README の「失敗したら」を見る。

参照：[fork の同期 API](https://docs.github.com/en/rest/branches/branches#sync-a-fork-branch-with-the-upstream-repository)、
[起動する Worker](https://github.com/numachang/fork-sync-cron)、
[reader 側の運用](https://github.com/numachang/sendo-reader/blob/main/FORK.md)。
