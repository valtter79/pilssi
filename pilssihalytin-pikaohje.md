# Pilssihälytin — pikaohje

## Tekstiviestikomennot

Lähetä numeroon **Veneen Pilssipumppu**. Komennot toimivat vain omasta numerostasi.

| Komento | Mitä tapahtuu |
|---|---|
| `STATUS` tai `TILANNE` | Akkujännitteet, pumppukäynnit, kenttä, versio |
| `GPS` tai `SIJAINTI` | Tilanne + karttalinkki |
| `OFF` tai `VAIMENNA` | Hälytykset pois (ajon ajaksi). Käynnit kirjataan silti. |
| `ON` tai `PAALLE` | Hälytykset takaisin päälle |
| `UPDATE` tai `PAIVITA` | Päivitys 4G:n yli GitHubista — ensisijainen tapa |
| `OTA` tai `WIFI` | Laite avaa WiFi-verkon 10 min. Salasana tulee tekstiviestissä. |

Laite lähettää itse: käynnistysviestin, hälytykset ja päivittäisen "Vene OK" -viestin klo 9.

---

## Päivitys: 6 askelta

```bash
# 1. Kopioi uusi versio oikeaan paikkaan
cp ~/Downloads/pilssihalytin.ino ~/Desktop/pilssi/src/pilssihalytin.ino

# 2. Avaa ja tarkista asetukset (rivit ~57-63)
open -e ~/Desktop/pilssi/src/pilssihalytin.ino
```

Tarkista nämä kaksi riviä:
```cpp
const char* ALARM_PHONE  = "+358406413441";
const char* FIRMWARE_URL = "https://raw.githubusercontent.com/valtter79/pilssi/main/firmware.bin";
```

```bash
# 3. Käännä
cd ~/Desktop/pilssi
pio run

# 4. Tarkista että muutokset ovat mukana
strings -a .pio/build/t-a7670/firmware.bin | grep githubusercontent
ls -l .pio/build/t-a7670/firmware.bin      # aikaleiman pitää olla tuore

# 5. Kopioi työpöydälle
cp .pio/build/t-a7670/firmware.bin ~/Desktop/
```

**6.** GitHub → repo `pilssi` → **Add file ▾** → **Upload files** → raahaa `firmware.bin` → **Commit changes**

Sitten **odota 15 min** (raw-osoitteen välimuisti) ja lähetä tekstiviesti `UPDATE`.
Onnistumisen näet statusviestin versionumerosta.

---

## Missä mikäkin on

| Mikä | Missä |
|---|---|
| Muokattava lähdekoodi | `~/Desktop/pilssi/src/pilssihalytin.ino` |
| Käännetty binääri | `~/Desktop/pilssi/.pio/build/t-a7670/firmware.bin` |
| Projektiasetukset | `~/Desktop/pilssi/platformio.ini` |
| Firmware netissä | github.com/valtter79/pilssi |

**Vain `src`-kansion tiedosto kääntyy.** Työpöydällä tai muualla olevat kopiot ovat näkymättömiä PlatformIO:lle — tämä on se ansa johon on helppo pudota.

`.pio`-kansio on piilotettu. Finderissa saat piilotetut näkyviin näppäimillä **⌘ ⇧ .** tai avaat kansion suoraan:
```bash
open ~/Desktop/pilssi/.pio/build/t-a7670/
```

---

## Jos jokin ei toimi

**Käännös kaatuu** → lue virheen ensimmäinen rivi, siinä on tiedosto ja rivinumero.

**`grep githubusercontent` ei tulosta mitään** → muokkasit väärää tiedostoa. Kopioi oikea `src`-kansioon ja käännä uudelleen.

**UPDATE ei tuota mitään** → laite oli ehkä juuri nukahtanut, tai vanha viesti odottaa SIMillä. Kokeile uudelleen; komennot luetaan joka heräämisellä. Jos ei toimi, käytä `OTA`-komentoa ja WiFi-päivitystä.

**Laite ei vastaa mihinkään** → tarkista sulake ja päävirta. Hälytin on kytketty suoraan akulle päävirtakytkimen ohi.

---

## USB-flashaus (vain jos OTA ei toimi)

```bash
cd ~/Desktop/pilssi
pio run -t upload
pio device monitor          # avaa ENNEN kuin painat RST
```

Huom: irrota buckin 5 V (VBUS-johto) USB-session ajaksi — muuten Mac ei aina tunnista porttia.

---

## Kytkennät muistin virkistykseksi

| Mistä | Mihin |
|---|---|
| MT-301 `o2` | LilyGO **IO32** |
| MT-301 `VCC` | LilyGO **V3V3** (ei 5 V!) |
| MT-301 `GND` | LilyGO **GND** + buckin VOUT− |
| MT-301 `2+` / `2−` | Uimurikytkimen lähtö / veneen miinus |
| Jakomoduuli `S` / `−` | LilyGO **IO36** / **GND** |
| Jakomoduuli ruuvit | 12 V + / − |
| Buck `VOUT+` | LilyGO **VBUS** |
| Buck `VIN+` / `VIN−` | 12 V sulakkeen kautta |

MT-301: tulopuolen jännitevalinta **12V**, lähtöpuolen **3.3V**. Lähtö on aktiivinen HIGH.
