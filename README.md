# kasanegi-site

kasanegi.com の 1 ページのサイト。`index.html`(インライン CSS・外部依存なし)と `assets/`(図・OG 画像)・`favicon.svg`・`robots.txt`・`sitemap.xml`、GitHub Pages 用の `CNAME`(`kasanegi.com`)・`.nojekyll`。MDAF の節は `index.html` の中で HTML コメントにしてあり、表示しない。`demo/` は別の作業。

## 公開の前に: kasanegi.com はいま MDAF のサイトとして稼働している(2026-10-07 確認)

- レジストラは Cloudflare, Inc.(RDAP)。DNS も Cloudflare(NS `crystal.ns.cloudflare.com` / `viddy.ns.cloudflare.com`)。DNSSEC なし・CAA なし・MX なし。
- `https://kasanegi.com` と `https://www.kasanegi.com` は Cloudflare のプロキシ越しに 200。title は「Japan public-data APIs for AI agents — kasanegi.com」。
- 配信元は Cloudflare Workers の `micro-data-api-factory`(`C:\dev\x402\micro-data-api-factory`)と推定。その `wrangler.toml` に kasanegi.com のルートは書かれておらず、ドメインの割り当てはダッシュボード側と推定(確かめていない)。

手順 4 で DNS を GitHub Pages に向けると、MDAF のサイトは kasanegi.com から外れる(API が同じホスト名で動いていれば API も)。MDAF をどこへ移すかを先に決めてから進める。

Cloudflare と GitHub の画面の名前は 2026-10-07 に開いて確かめたものではない。`gh api` のコマンドもまだ実行していない。

## 手順

### 1. GitHub にリポジトリを作って push する

```
cd C:\dev\kasanegi-site
gh repo create MatsushitaTokitsugu/kasanegi-site --public --source . --remote origin --push
```

### 2. GitHub Pages を有効にする(ブランチ main・ルート)

- 画面: https://github.com/MatsushitaTokitsugu/kasanegi-site/settings/pages → Build and deployment → Source: Deploy from a branch → Branch: `main` / `(root)` → Save
- または: `gh api -X POST repos/MatsushitaTokitsugu/kasanegi-site/pages -f "source[branch]=main" -f "source[path]=/"`

### 3. カスタムドメインを kasanegi.com にする

- 同じ画面の Custom domain に `kasanegi.com` → Save(リポジトリの `CNAME` と同じ値)
- または: `gh api -X PUT repos/MatsushitaTokitsugu/kasanegi-site/pages -f cname=kasanegi.com`
- 推奨: アカウントの Settings → Pages → Add a domain で kasanegi.com を検証する。表示される TXT(`_github-pages-challenge-MatsushitaTokitsugu`)を手順 4 と同じ Cloudflare の DNS に足す。他のアカウントにこのドメインを Pages で使われるのを防ぐ。

### 4. DNS を GitHub Pages に向ける(Cloudflare ダッシュボード → kasanegi.com)

1. いま kasanegi.com と www を受けているものを外す。
   - Workers & Pages → `micro-data-api-factory` → Settings → Domains & Routes に kasanegi.com / www.kasanegi.com があれば外す。Worker のカスタムドメインの DNS レコードは Worker 側の管理で、DNS の画面からは消せない。
   - ルート(`kasanegi.com/*` など)の場合は Routes から外し、DNS → Records に残るプロキシ付きの A / AAAA / CNAME を消す。
   - どちらの方式かは確かめていない。
2. DNS → Records に足す。Proxy status はすべて DNS only(灰色の雲)。プロキシ(オレンジ)のままだと、GitHub の証明書発行と Enforce HTTPS が通らないことがある。

| Type | Name | Content |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA(任意) | `@` | `2606:50c0:8000::153` |
| AAAA(任意) | `@` | `2606:50c0:8001::153` |
| AAAA(任意) | `@` | `2606:50c0:8002::153` |
| AAAA(任意) | `@` | `2606:50c0:8003::153` |
| CNAME | `www` | `matsushitatokitsugu.github.io` |

3. 確認: `nslookup kasanegi.com 1.1.1.1` が 185.199.108〜111.153 を返す。GitHub の Pages の画面で DNS check successful。

### 5. HTTPS を強制する

- 証明書が出たら(数分〜24 時間)、Pages の画面の Enforce HTTPS にチェック。
- または: `gh api -X PUT repos/MatsushitaTokitsugu/kasanegi-site/pages -F https_enforced=true`
- 状態: `gh api repos/MatsushitaTokitsugu/kasanegi-site/pages --jq "{status, cname, https_enforced, cert: .https_certificate.state}"`

### 6. contact@kasanegi.com を受ける(Cloudflare Email Routing)

いま MX は無い。DNS が Cloudflare なので、Email Routing で受けて手元の受信箱へ転送する。

1. Cloudflare ダッシュボード → kasanegi.com → Email → Email Routing → 有効にする。MX 3 本(`route1/2/3.mx.cloudflare.net`)と SPF の TXT(`v=spf1 include:_spf.mx.cloudflare.net ~all`)が足される。
2. Destination addresses に転送先を足し、届いた確認メールのリンクを押す。
3. Routing rules → Custom address `contact` → Send to an email → 転送先。
4. 確認: `nslookup -type=MX kasanegi.com 1.1.1.1` で route1〜3。別のアドレスから送って届くか。

- Email Routing は受けて転送するだけ。contact@kasanegi.com から送る・返信するには、別の送信の仕組みが要る。
- 手順 4 の A / CNAME と、Email Routing の MX / TXT は干渉しない。

## 更新

`index.html` を直して commit → `git push`。数分で反映される。
