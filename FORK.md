# この fork の運用

`numachang/sendo` は、`sendo-feed/sendo` を Cloudflare にデプロイして使うための fork。
上流の開発用の PR・マージ手順とは別に、更新を自動で取り込む。

## 上流への自動追従

[Sync upstream](https://github.com/numachang/sendo/actions/workflows/sync-upstream.yml) が毎時17分に、
上流の `main` をこの fork の `main` に取り込む。GitHub の「Sync fork」と同じ API を使い、
独自コミットを残す。競合した場合は失敗し、上書きしない。
取り込みで `main` が更新されると、接続済みの Workers Builds がマイグレーションとデプロイを行う。

- 手動で動かすには、Actions の「Sync upstream」から「Run workflow」を選ぶ。
- 認証には自動発行される `GITHUB_TOKEN`（`contents: write`）を使う。追加のシークレットは不要。
- このワークフローは `numachang/sendo` でだけ動く。
- 停止するには、Actions の「Sync upstream」から「Disable workflow」を選ぶ。
- 失敗したら Actions のログを確認する。競合は手動で解消してから、もう一度動かす。

GitHub の定期実行には遅延があり、公開リポジトリで60日間活動がないと自動で無効になる。
その場合は Actions から有効に戻す。

参照：[fork の同期 API](https://docs.github.com/en/rest/branches/branches#sync-a-fork-branch-with-the-upstream-repository)、
[定期実行の制約](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule)。
