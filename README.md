# pi-lock
pi-coding-sgent インストールスクリプトでの hash 確認用 lock ファイル。

## 推奨方法

信頼できる環境でロックファイルを作り、VM 内で `npm ci`

1. ホスト側で一度だけ実行します。

```bash
mkdir pi-lock && cd pi-lock && npm init -y >/dev/null
npm install --package-lock-only --ignore-scripts --save-exact \
  @earendil-works/pi-coding-agent@<VERSION>
sha256sum package.json package-lock.json
```

- `<VERSION>` は `npm view @earendil-works/pi-coding-agent version` で確認できます。公開直後のバージョンは避けるのが無難です。
- パッケージ名は `@earendil-works/pi-coding-agent` です。
- `--ignore-scripts` を付けているのは、通常の npm インストールでは、ライフサイクルスクリプトが不要とされているためです。

2. 2. 2 つのファイルを、どこか VM から取得できる場所に置きます。 自分の GitHub リポジトリで構いません。次の手順でハッシュを検証するので、置き場所自体は信頼しなくて済みます。

3. 以下のスクリプトを作成。

```
PI_LOCK=https://raw.githubusercontent.com/<you>/<repo>/<commit-sha>/pi-lock
install -d /opt/pi && cd /opt/pi
curl -fsSLO "$PI_LOCK/package.json" -O "$PI_LOCK/package-lock.json"
sha256sum -c <<'SUMS'
<package.json の sha256>  package.json
<package-lock.json の sha256>  package-lock.json
SUMS
npm ci --ignore-scripts
ln -sf /opt/pi/node_modules/.bin/pi /usr/local/bin/pi
```

`npm ci` はロックファイルの integrity(sha512)と一致しないパッケージを拒否します。2 つのファイルの sha256 を cloud-init に埋め込めば、依存ツリー全体が固定されます。更新するときは、ホストでロックファイルを作り直し、ハッシュも差し替えます。