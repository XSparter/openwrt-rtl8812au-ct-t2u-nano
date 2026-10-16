<p align="center">
  <img src="assets/logo.svg" width="160" alt="T2U Nano driver logo">
</p>

<h1 align="center">rtl8812au-ct for OpenWrt + TP-Link Archer T2U Nano (2357:012e)</h1>

<p align="center">
  <a href="https://buymeacoffee.com/dr_gaussx"><img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-dr_gaussx-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black" alt="Buy Me a Coffee"></a>
  <img src="https://img.shields.io/badge/OpenWrt-24.10-blue" alt="OpenWrt 24.10">
</p>

<p align="center"><i>🇮🇹 Anche in italiano: <a href="README.it.md">README.it.md</a></i></p>

## What

Rebuild of Candela's [rtl8812au-ct](https://github.com/greearb/rtl8812AU_8821AU_linux) driver for OpenWrt 24.10 (`ramips/mt7621`, kernel 6.6) with **one added USB ID**:

```
{USB_DEVICE(0x2357, 0x012e), .driver_info = RTL8821}  /* TP-Link Archer T2U Nano (newer rev, RTL8811AU) */
```

The stock `kmod-rtl8812au-ct` from OpenWrt feeds already supports the older T2U Nano revisions (`2357:011e`, `2357:0122`) but **not** `2357:012e`, so the dongle stays unbound (`Driver=(none)`). This build fixes exactly that — same driver, same mac80211 AP support, one line more.

Built automatically by [GitHub Actions](.github/workflows/build-driver.yml) with the official OpenWrt 24.10.8 SDK. Every push to `package/` rebuilds the `.ipk`.

> ⚠️ **Disclaimer.** I assume **no responsibility**: use at your own risk, may void warranty / break your device. Do **not** force-bind the stock driver to this ID via `new_id` — on our test router that instantly rebooted the box (wrong `driver_info` → panic). Use this properly patched build instead.

## Install

1. Download the `.ipk` from [**Releases**](../../releases).
2. Copy to the router and install (it upgrades/replaces the feed's `kmod-rtl8812au-ct`):
   ```sh
   opkg install kmod-rtl8812au-ct-*.ipk
   reboot
   ```
3. Plug the T2U Nano in, then check:
   ```sh
   ls /sys/class/ieee80211/   # a new phy should appear
   iw phy phyX info | grep -A6 'Supported interface modes'   # AP mode listed
   ```
4. Add a wireless interface on the new radio from LuCI (`Network → Wireless`) or `/etc/config/wireless`.

## Build it yourself

Any push to `package/` (or the **Run workflow** button) triggers the CI build. Locally, with the OpenWrt SDK:

```sh
cp -r package/rtl8812au-ct <sdk>/package/
cd <sdk> && make package/rtl8812au-ct/compile -j$(nproc)
```

## Credits

- [Ben Greear / Candela](https://github.com/greearb/rtl8812AU_8821AU_linux) for the -ct driver
- [OpenWrt](https://openwrt.org/) project

<p align="center">
  <a href="https://buymeacoffee.com/dr_gaussx"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" width="200" alt="Buy Me a Coffee"></a>
</p>
