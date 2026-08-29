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

---

# Additional platform patches (RR-N 7.1.2 tree)

These apply outside `frameworks/base`. Each file is a `git apply`-style diff
against the pristine RR-N 7.1.2 source of the named project.

## rr-n-frameworks-native-graphicbuffer-compat.patch

Apply in `frameworks/native`.

`libs/ui/GraphicBuffer.cpp`: re-exports the Nougat-7.0 4-argument
`GraphicBuffer(w, h, format, usage)` constructor symbol
(`_ZN7android13GraphicBufferC1Ejjij`) that N-MR1 dropped, as an `extern "C"`
alias placement-constructing via the current defaulted ctor. Required because
the MTK 7.0 vendor blobs (`libcam_utils.so`, `libcam.client.so`,
`libMtkOmxVenc.so`, `libmtk_mmutils.so`) fail to dlopen without it — camera
HAL module open returned -22.

## rr-n-libhardware_legacy-mtk-wifi.patch

Apply in `hardware/libhardware_legacy`.

- `wifi.c`: power the MTK combo WLAN function on/off by writing `1`/`0` to
  `/dev/wmtWifi` inside `wifi_load_driver()`/`wifi_unload_driver()`, and wait
  for `wlan0` to register. Without this the netdevs never appear and
  WifiStateMachine cannot start the supplicant. Gated on
  `MTK_WMT_WIFI_POWERON`.
- `Android.mk`: sets `-DMTK_WMT_WIFI_POWERON` when the board defines
  `BOARD_HAS_MTK_WMT_WIFI := true` (set in BoardConfig.mk).

## rr-n-packages-apps-settings-cmparts.patch

Apply in `packages/apps`.

- `Settings/AndroidManifest.xml`: move the `org.apache.http.legacy`
  uses-library under `<application>` (PackageParser ignores manifest-level
  entries) and add a required `org.cyanogenmod.platform` uses-library.
  Without it Settings crash-loops inflating `CMPartsPreference` in
  Display/Battery.
- `CMParts/AndroidManifest.xml`: required `org.cyanogenmod.platform`
  uses-library.
- `CMParts/.../TouchscreenGestureSettings.java`: null-guard the gesture list
  (BootReceiver NPE on devices whose HAL reports gesture support but returns
  no gestures).

## License

GPL-3.0 (matches frameworks/base). Patch contains Apache-2.0 AOSP code
(SQLiteTokenizer) — license headers preserved.
