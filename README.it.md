<p align="center">
  <img src="assets/logo.svg" width="160" alt="Logo driver T2U Nano">
</p>

<h1 align="center">rtl8812au-ct per OpenWrt + TP-Link Archer T2U Nano (2357:012e)</h1>

<p align="center">
  <a href="https://buymeacoffee.com/dr_gaussx"><img src="https://img.shields.io/badge/Offrimi%20un%20caff%C3%A8-dr_gaussx-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black" alt="Buy Me a Coffee"></a>
  <img src="https://img.shields.io/badge/OpenWrt-24.10-blue" alt="OpenWrt 24.10">
</p>

<p align="center"><i>🇬🇧 Also in English: <a href="README.md">README.md</a></i></p>

## Cos'è

Ricompilazione del driver [rtl8812au-ct](https://github.com/greearb/rtl8812AU_8821AU_linux) di Candela per OpenWrt 24.10 (`ramips/mt7621`, kernel 6.6) con **un USB ID in più**:

```
{USB_DEVICE(0x2357, 0x012e), .driver_info = RTL8821}  /* TP-Link Archer T2U Nano (nuova revisione, RTL8811AU) */
```

Il `kmod-rtl8812au-ct` stock dei feed OpenWrt supporta già le vecchie revisioni T2U Nano (`2357:011e`, `2357:0122`) ma **non** la `2357:012e`, quindi la chiavetta resta scollegata (`Driver=(none)`). Questa build sistema esattamente questo — stesso driver, stesso supporto AP mac80211, una riga in più.

Compilato automaticamente dalle [GitHub Actions](.github/workflows/build-driver.yml) con l'SDK ufficiale OpenWrt 24.10.8. Ogni push su `package/` ricompila l'`.ipk`.

> ⚠️ **Liberatoria.** **Non mi assumo alcuna responsabilità**: usalo a tuo rischio e pericolo, può invalidare la garanzia / rompere il dispositivo. **Non** forzare il bind del driver stock a questo ID via `new_id` — sul nostro router di test ha riavviato di colpo la macchina (panic). Usa questa build patchata come si deve.

## Installazione

1. Scarica l'`.ipk` dai [**Releases**](../../releases).
2. Copialo sul router e installa (sostituisce/aggiorna il `kmod-rtl8812au-ct` dei feed):
   ```sh
   opkg install kmod-rtl8812au-ct-*.ipk
   reboot
   ```
3. Collega la T2U Nano e verifica:
   ```sh
   ls /sys/class/ieee80211/   # deve comparire una nuova phy
   iw phy phyX info | grep -A6 'Supported interface modes'   # AP mode in elenco
   ```
4. Aggiungi un'interfaccia wireless sulla nuova radio da LuCI (`Network → Wireless`) o da `/etc/config/wireless`.

## Compilalo da solo

Ogni push su `package/` (o il pulsante **Run workflow**) avvia la build CI. In locale, con l'SDK OpenWrt:

```sh
cp -r package/rtl8812au-ct <sdk>/package/
cd <sdk> && make package/rtl8812au-ct/compile -j$(nproc)
```

## Crediti

- [Ben Greear / Candela](https://github.com/greearb/rtl8812AU_8821AU_linux) per il driver -ct
- Progetto [OpenWrt](https://openwrt.org/)

<p align="center">
  <a href="https://buymeacoffee.com/dr_gaussx"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" width="200" alt="Buy Me a Coffee"></a>
</p>
