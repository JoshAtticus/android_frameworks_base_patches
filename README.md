# RR-N frameworks/base patches for the Alcatel 1C (5009A / u5a)

Required `frameworks/base` changes to build Resurrection Remix Nougat for the
Alcatel 1C (MT6580). The RR nougat branch tip was left mid-rebase and its
backported apps depend on these framework additions.

## Apply

```bash
cd frameworks/base
git am path/to/rr-n-frameworks-base.patch
# or: git apply rr-n-frameworks-base.patch
```

## Contents

- `core/java/android/database/sqlite/SQLiteTokenizer.java` (new, from AOSP P)
- `SQLiteQueryBuilder`: `update()`, `delete()`, `setStrictColumns()`, `setStrictGrammar()`
- `DownloadManager`: `COLUMN_DESTINATION`, `COLUMN_FILE_NAME_HINT`, `Query.setFilterByString()`
- `Settings.Secure.MANAGED_PROVISIONING_DPC_DOWNLOADED`

## Also required (not a git patch)

Jack server fix — RR's `prebuilts/sdk/tools/jack-admin` never exports the TLS
certs it later uses. After the first `setup-jack-server` run creates
`~/.jack-server/{server,client}.jks`, export them:

```bash
keytool -exportcert -alias server -keystore ~/.jack-server/server.jks -storepass Jack-Server -file ~/.jack-server/server.pem
keytool -exportcert -alias client -keystore ~/.jack-server/client.jks -storepass Jack-Server -file ~/.jack-server/client.pem
```

## License

GPL-3.0 (matches frameworks/base). Patch contains Apache-2.0 AOSP code
(SQLiteTokenizer) — license headers preserved.
