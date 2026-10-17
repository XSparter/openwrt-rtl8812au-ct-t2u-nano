<p align="center">
  <img src="assets/logo.svg" width="160" alt="TP-Link 2357:012e su OpenWrt">
</p>

<h1 align="center">TP-Link Archer T3U Nano (2357:012e) su OpenWrt<br>la storia rtw88 — testato AP + STA</h1>

<p align="center">
  <a href="https://buymeacoffee.com/dr_gaussx"><img src="https://img.shields.io/badge/Offrimi%20un%20caff%C3%A8-dr_gaussx-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black" alt="Buy Me a Coffee"></a>
  <img src="https://img.shields.io/badge/OpenWrt-24.10%20%C2%B7%20kernel%206.6-blue" alt="OpenWrt 24.10 / kernel 6.6">
  <img src="https://img.shields.io/badge/chipset-RTL8812BU-informational" alt="chipset RTL8812BU">
</p>

<p align="center"><i>🇬🇧 English version: <a href="README.md">README.md</a></i></p>

## TL;DR

La chiavetta USB con ID **`2357:012e`** è una **TP-Link Archer T3U Nano** (AC1300, chipset **Realtek RTL8812BU**). Su OpenWrt funziona con il **driver mainline `rtw88`** — **nessun driver esterno necessario**:

```sh
opkg update
opkg install kmod-rtw88-8822bu
```

Inseriscila, crea l'interfaccia (UCI/LuCI — esempi sotto) e funziona **sia in modalità AP che STA (client)**. Verificato su OpenWrt 24.10.8, kernel 6.6.144, ramips/mt7621 (Xiaomi Mi Router 3G). Misure qui sotto.

> ❗ I pacchetti driver `rtl8812au-ct` / `rtl8821au` in questa repo sono **legacy**: compilati durante l'indagine partendo da un presupposto sbagliato. **Non usarli per `2357:012e`** — usa il mainline `rtw88`. Restano per memoria storica, e gli `.ipk` precompilati nelle [Releases](../../releases) sono marcati **deprecati**.

## Il dispositivo

| | |
|---|---|
| USB ID | `2357:012e` (vendor TP-Link) |
| Dispositivo | **Archer T3U Nano** v1.x (AC1300, dual-band) |
| Chipset | **Realtek RTL8812BU** (famiglia RTL8822BU), 2×2 802.11ac, USB 2.0 |
| Driver Linux corretto | **mainline `rtw88`** → pacchetto OpenWrt **`kmod-rtw88-8822bu`** (tira dentro `rtw88_usb` + `rtw88_8822b` + `rtw88_core`) |
| Testato su | OpenWrt 24.10.8, kernel 6.6.144, ramips/mt7621, porta USB 2.0 |
| Nomi interfacce | `phyN-ap0` (AP), `phyN-sta0` (STA) — creati da mac80211 |
| LED | non pilotato da `rtw88` su kernel 6.6 — **resta spento**, cosmetico (vedi note) |

Verifica che sia la stessa: `lsusb` → `ID 2357:012e TP-Link Archer T3U Nano`, oppure `cat /sys/kernel/debug/usb/devices`. L'ID è confermato nel database linux-hardware.org come gestito da `rtw8822bu.c`.

## Installazione e configurazione

### 1. Pacchetti

```sh
opkg update
opkg install kmod-rtw88-8822bu
```

Il modulo si carica da solo e crea una nuova `phy` (controlla con `ls /sys/class/ieee80211/`). Per la modalità AP serve anche `hostapd`/`wpad` (praticamente ogni immagine OpenWrt ce l'ha già).

### 2. Modalità AP (esempio 5 GHz VHT80)

Modo più facile per creare la radio: LuCI → *Network → Wireless → Add* (rileva la radio USB da sola). UCI equivalente dal nostro router:

```uci
config wifi-device 'radio2'
	option type 'mac80211'
	option path 'platform/1e1c0000.xhci/usb1/1-1/1-1:1.0'   # path USB: adattalo alla tua board (vedi nota)
	option band '5g'
	option channel '36'
	option htmode 'VHT80'
	option country 'IT'
	option disabled '0'

config wifi-iface 'default_radio2'
	option device 'radio2'
	option network 'lan'          # ← bridged in LAN: DHCP e firewall "funzionano e basta"
	option mode 'ap'
	option ssid 'T2U-Nano-Test'
	option encryption 'psk2'
	option key 'testtest1234'
```

Applica con `uci commit wireless && wifi up radio2` (o `wifi reload`).

> **Il `path` USB dipende dalla macchina.** Se non lo conosci: aggiungi la radio da LuCI, oppure guardalo in `ubus call network.wireless status` dopo aver inserito la chiavetta. Path sbagliato = radio giù.

### 3. Modalità STA (client di un altro AP)

```uci
config wifi-iface 'default_radio2'
	option device 'radio2'
	option network 'wwan'
	option mode 'sta'
	option ssid 'YourSSID'
	option encryption 'psk2'
	option key 'YourKey'
```

```uci
config interface 'wwan'
	option proto 'dhcp'
	option defaultroute '0'       # lascia 0 a meno che tu non voglia usare questo link come default route
	option peerdns '0'
```

Più una regola firewall (fw4) perché le risposte DHCP verso l'interfaccia client non vengano scartate (vedi il gotcha sotto):

```uci
config rule
	option name 'Allow-WWAN-DHCP'
	option device 'phy3-sta0'     # adatta: nome della TUA interfaccia sta
	option proto 'udp'
	option dest_port '68'
	option family 'ipv4'
	option target 'ACCEPT'
```

Poi `uci commit && wifi up radio2 && ifup wwan`. Verifica: `iw dev phyN-sta0 link` e il lease `udhcpc` in `logread`.

## Prestazioni (misurate, non stimate)

Setup: Xiaomi Mi Router 3G (ramips/mt7621, **USB 2.0**), OpenWrt 24.10.8, chiavetta in AP su 5 GHz VHT80 ch36 (IT), client: Samsung Z Flip 6 (2×2) e un PC (2×2).

| Test | Risultato |
|---|---|
| PHY client ↔ AP | **866 Mbps** (VHT80, MCS9, 2SS — su entrambi i client; telefono: TX 866 / RX 585) |
| Latenza, stessa subnet via Wi-Fi | **1–2 ms**, 0% persi |
| Throughput — AP → client (`curl` 64 MB dal router) | **~156 Mbps** |
| Throughput — client → AP (upload verso il router) | **~57–59 Mbps** |
| Controllo: stessi tool via Ethernet | 578 Mbps (tool validati; i ~59 Mbps sono il vero limite RX dell'AP con questa chiavetta/host) |
| Modalità STA — associazione + DHCP + ping al gateway hotspot | OK (5–7 ms al telefono; 38–52 ms a 8.8.8.8 attraverso l'uplink del telefono) |
| AP 2.4 GHz (ch1, 20 MHz) | ~40 Mbps (ch6 da noi era congestionato: ~20 Mbps) |

Note: trasferimenti `curl` da 64 MB, con source-binding (`--interface`), un client per volta. Con USB 2.0 su mt7621 non aspettarti centinaia di Mbps — i numeri sopra sono quello che l'hardware fa davvero qui.

## Note, stranezze e il grande gotcha del firewall

**Il gotcha che ci è costato giorni:** un'interfaccia AP **non assegnata a nessuna zona firewall** si vede scartare l'input da **fw4** (`input policy drop`) — i client non riescono nemmeno a ottenere il DHCP, ed è *identico* a un driver/AP rotto (per questo siamo finiti in un intero rabbit hole di debug del driver). Fix: metti l'AP in `network 'lan'` (bridge) come sopra. Se lo isoli in una zona dedicata, ricordati le regole input per DHCP/ICMP. Idem in STA: le risposte DHCP che bussano su `phyN-sta0` vogliono una regola input ACCEPT (fornita sopra).

- **Stranezza alla prima inserzione:** al primo plug `rtw_8822bu` può loggare `failed to register 0xc4` e il dispositivo si ri-enumera da solo; al secondo tentativo va e tutto funziona. Cosmetico.
- **Messaggi dmesg innocui:** `error beacon valid` / `failed to download drv rsvd page` possono comparire mentre riconfiguri l'AP (cambi di canale/banda) — osservati, nessun impatto funzionale.
- **LED:** il LED della chiavetta è controllato dai registri del chip, e il mainline `rtw88` ha imparato a pilotarlo solo da **Linux 6.14** (upstream *"wifi: rtw88: Add support for LED blinking"*, commit `4b6652bc6d8d`; nei mac80211 più nuovi di OpenWrt come `062-v6.14-wifi-rtw88-Add-support-for-LED-blinking.patch`). Su OpenWrt 24.10 (kernel 6.6) il **LED resta spento — è previsto, è cosmetico**. Su Windows/con i driver vendor lampeggia. `ls /sys/class/leds` su 24.10 non mostra nessuna voce `rtw88-*` (verificato).
- **Country/regdomain:** imposta il `country` giusto (noi `IT`). Ch36–48 in 5 GHz = niente seccature DFS. In 2.4 GHz preferisci il canale meno affollato tra 1/6/11.
- **Nomi:** le interfacce sono `phyN-ap0` / `phyN-sta0` (nomi dinamici mac80211), NON `wlanX` — non stupirti.

## Perché la repo si chiama `...t2u-nano`? (storia — cos'abbiamo scoperto)

1. La chiavetta si enumera come `2357:012e` ma **nessun driver in-tree la rivendicava** (`Driver=(none)`): apparentemente "non supportata".
2. Abbiamo ipotizzato fosse *l'ennesima revisione della T2U Nano*: TP-Link aveva già fatto uscire le T2U Nano `2357:011e` / `2357:0122` con silicio RTL8811AU, quindi un PID nuovo sembrava una nuova revisione dello stesso prodotto → abbiamo **forzato l'ID nei driver vendor della FAMIGLIA SBAGLIATA** (`driver_info = RTL8821`): `rtl8812au-ct` di Candela (commit in questa repo) e `rtl8821au` di morrownr. Questo è stato l'errore.
3. Vicoli ciechi di quel percorso: forzare il bind del driver *stock* all'ID via `new_id` ha **riavviato all'istante il router** (`driver_info` sbagliato → panic — non fatelo); la build -ct patchata si agganciava ma l'AP era instabile; la build vendor 8821au si associava in STA ma con latenza assurda (~350 ms) — niente di tutto ciò poteva mai funzionare davvero.
4. Ricontrollando i database dei USB ID (linux-hardware.org) è emersa la verità: **`2357:012e` = Archer T3U Nano, chipset RTL8812BU** — e il mainline `rtw88` (`rtw8822bu`) lo supportava già da sempre.
5. Installato `kmod-rtw88-8822bu` → STA e AP testati e funzionanti (prove sopra). L'ultimo mistero (i client si associavano ma niente DHCP in AP) era il **problema delle zone fw4**, non il driver.

I pacchetti legacy restano nell'albero per riferimento; i loro `.ipk` precompilati sono pubblicati nelle [Releases](../../releases) e marcati **deprecati**.

## Compilalo da solo (solo pacchetti legacy)

**Per `2357:012e` non devi compilare nulla** — `kmod-rtw88-8822bu` arriva dal feed ufficiale. Le istruzioni sotto riproducono le build *legacy* dei driver vendor.

### Con l'SDK OpenWrt (il metodo esatto usato qui)

```sh
# 1. Prendi l'SDK per target e release giusti (noi: 24.10.8, ramips/mt7621)
wget https://downloads.openwrt.org/releases/24.10.8/targets/ramips/mt7621/openwrt-sdk-24.10.8-ramips-mt7621_*.tar.zst
tar --zstd -xf openwrt-sdk-*.tar.zst
cd openwrt-sdk-*/

# 2. Copia i pacchetti nell'albero
cp -r /path/to/this/repo/package/rtl8812au-ct package/
cp -r /path/to/this/repo/package/rtl8821au   package/

# 3. Compila
make package/rtl8812au-ct/compile -j$(nproc) V=s
make package/rtl8821au/compile   -j$(nproc) V=s
```

Gli `.ipk` finiscono in `bin/packages/mipsel_24kc/base/` (i moduli kernel anche in `bin/targets/ramips/mt7621/packages/`). Prerequisiti host su Debian/Ubuntu: `build-essential file gawk gettext libncurses-dev python3 rsync unzip wget zstd`.

### Perché i Makefile sembrano "patchati"

Entrambi i driver vendor hanno richiesto fix di build per Linux 6.6 + backports OpenWrt + GCC recenti: downgrade `-Wno-error` (il codice upstream scatta su `-Werror`), un `Build/Prepare` che rimuove un `wireless.h` in conflitto, allenta `-Wframe-larger-than`, disattiva un ramo di workaround API rotto e (per rtl8821au) un **fix alla finestra di download del firmware** (`patches/002-fwdl-window.patch`). Tutti fix di build — nulla di tutto ciò serve per `rtw88`.

## Binari precompilati (legacy, deprecati) + MD5

Vedi le [Releases](../../releases). Compilati con l'**SDK OpenWrt 24.10.8, ramips/mt7621, kernel 6.6.144** e solo per quello — i moduli kernel sono legati al kernel: `opkg` rifiuterà di installarli su altri kernel/hash (`6.6.144~f31f6f85…`).

| File | MD5 | Cos'è |
|---|---|---|
| `kmod-rtl8812au-ct_6.6.144.2022.10.26~9b2b203a-r3_mipsel_24kc.ipk` | `486656d19726ffcfed75c9f8ff938176` | Candela rtl8812au-ct + ID `012e` forzato — legacy |
| `kmod-rtl8821au_6.6.144.5.12.5.2-r5_mipsel_24kc.ipk` | `f14239a83b8415319f349036d52d7cf9` | morrownr 8821AU/8811AU + ID `012e` + fix fwdl (r5, ultimo tentativo vendor) — legacy |

Installazione (solo kernel corrispondente): `opkg install ./kmod-....ipk`. **Entrambi deprecati: quasi certamente vuoi `kmod-rtw88-8822bu` invece.**

## Crediti

- [OpenWrt](https://openwrt.org/) e il mainline **rtw88** (Realtek; supporto USB di Sascha Hauer et al.)
- [Ben Greear / Candela](https://github.com/greearb/rtl8812AU_8821AU_linux) — `rtl8812au-ct`
- [morrownr](https://github.com/morrownr/8821au-20210708) — `rtl8821au`
- Database USB-ID di linux-hardware.org (identificazione del dispositivo)

## Liberatoria

Nessuna garanzia di alcun tipo — usalo a tuo rischio, può invalidare la garanzia / rompere le cose. **Non** forzare il bind del driver stock a USB ID sconosciuti via `new_id`: sul nostro router di test ha causato un riavvio istantaneo (`driver_info` sbagliato → panic).

<p align="center">
  <a href="https://buymeacoffee.com/dr_gaussx"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" width="200" alt="Buy Me a Coffee"></a>
</p>
