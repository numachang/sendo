# この fork の運用

`numachang/sendo` は、`sendo-feed/sendo` を Cloudflare にデプロイして使うための fork。
上流の開発用の PR・マージ手順とは別に、更新を自動で取り込む。

## 上流への自動追従

[sendo-updater](https://github.com/sendo-feed/sendo-updater)（fork は `numachang/sendo-updater`、
Cloudflare の Worker は `sendo-updater`）が、毎分この fork と上流の差分を調べ、遅れていれば
上流の `main` をこの fork の `main` に取り込む（「Sync fork」と同じ API）。独自コミットは残り、
競合すれば上書きせずに止まる。取り込んだ push で、Workers Builds がマイグレーションとデプロイを行う。

reader の fork（`numachang/sendo-reader`）は、この fork が上流に追いつき、`main` の先頭で
`Workers Builds: sendo` が成功してから取り込まれる。この fork のデプロイが失敗している間は、
reader の取り込みも止まる。

- 仕組み、トークン、困ったときの直し方は sendo-updater の README を見る。決定は
  `docs/decisions/deploy.md` の 7。
- 手で取り込むときは、fork の画面で「Sync fork」→「Update branch」を押す。
- 自動の取り込みを止めるには、Cloudflare で `sendo-updater` の Worker を消す（reader の fork の取り込みも止まる）。
- 競合したら、`sendo-updater` のログに `failed (409)` が出る。fork で手で直す。
- 上流の更新によっては、Cloudflare の画面での準備（R2 を使えるようにする、など）が要る。
  取り込んだあとのデプロイが止まったら、Workers Builds のログと README の「初めてのデプロイ」を見る。

参照：[sendo-updater](https://github.com/sendo-feed/sendo-updater)、
[reader 側の運用](https://github.com/numachang/sendo-reader/blob/main/FORK.md)。
