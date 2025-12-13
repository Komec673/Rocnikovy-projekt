# Ročníkový projekt
## Cíl projektu
Cílem tohoto projektu je porovnat, jak se v profesionálním fotbale používají drahé monitorovací systémy (vesty s GPS) a jak bychom mohli podobnou funkci levně napodobit pomocí mikrokontrolerů (jako je např. Raspberry Pi Pico, Arduino, ESP32) a levných senzorů.

Zaměřím se na to, co umí drahé systémy přesně změřit (polohu, rychlost míče nebo hráče), navrhnu vlastní prototyp založený na mikrokontroleru, který nabídne cenově dostupnou alternativu a zjistím jaké problémy můžou nastávat s levnými součástkami (např. chyba v měření vzdálenosti).
## Průzkum
### Komerční systémy
Komerční sportovní analytické systémy se zaměřují na přesné a komplexní monitorování výkonu celého týmu v reálném čase.
#### Příklad: Systémy STATSports a Catapult
- Typ zařízení: Vesta/Podprsenka s malým boxem na zádech.
- Primární senzory: GNSS (GPS + GLONASS), nebo LPS (ClearSky, vlastní lokální polohový systém společnosti Catapult) pro venkovní sledování polohy. Akcelerometry, Gyroskopy a Magnetometry (IMU – Inertial Measurement Unit) pro sledování pohybu, zrychlení, zpomalení a směru.
- Sledované metriky: Rychlost (maximální, průměrná), uběhlá vzdálenost (celková, v různých rychlostních zónách), zrychlení/zpomalení (počet a intenzita), srdeční tep (pomocí externího hrudního pásu nebo integrovaného optického senzoru).
- Zpracování dat: Zařízení obsahuje výkonný mikrokontroler (často na bázi ARM Cortex architektury) a specializované čipy pro zpracování senzorických dat a fúzi senzorů. Data jsou logována do interní paměti a/nebo přenášena v reálném čase.
- Komunikace: Vysokofrekvenční rádiové přenosy (pro data v reálném čase na krátkou vzdálenost k přijímači u hřiště) nebo Bluetooth/Wi-Fi pro stažení logovaných dat po tréninku.
- Přesnost: Vysoká přesnost (poloha s přesností +- 1 metru, zrychlení s vysokou vzorkovací frekvencí až 100 Hz).
- Cena: Vysoká cena (25 000 kč+ za jednotku + licence).
![Vesta](vesta.png)

### DIY projekty - jednoduché monitotování
Tyto projekty se snaží replikovat základní funkce komerčních systémů s minimálními náklady, často zaměřené na jednu konkrétní metriku.
#### Příklad: Měření rychlosti hráče pomocí Raspberry Pi Pico W
Tento typ projektu je ideální pro demonstraci základních principů zpracování senzorických dat včetně bezdrátového přenosu dat v reálném čase.

- Typ zařízení: Malá krabička nebo modul připevněný na botě nebo opasku.
- Primární senzory: IMU modul (Inertial Measurement Unit) – např. MPU-6050 (Akcelerometr a Gyroskop). Levné moduly nemají GPS.
- Sledované metriky: Zrychlení (měřeno akcelerometrem), rychlost (vypočítána integrací zrychlení – v(t) = v0 + integral od t0 do t z a(tau) d(tau)), směr/rotace (měřeno gyroskopem).
- Mikrokontroler: Raspberry Pi Pico W – má dvě jádra pro rychlé zpracování a integrované Wi-Fi/Bluetooth. To řeší problém komunikace a umožňuje posílat data o zrychlení rovnou do počítače nebo telefonu.
- Zpracování dat: Mikrokontroler čte data ze senzorů (např. 100x za sekundu). Dvě jádra (Core 0 a Core 1) umožňují dělit práci: např. Core 0 čte a filtruje data ze senzoru, Core 1 se stará o bezdrátový přenos.
- Komunikace: Integrované Wi-Fi pro přenos dat do lokální sítě nebo na server (např. MQTT, HTTP) nebo Bluetooth pro připojení k aplikaci v telefonu. Toto je klíčová výhoda oproti standardnímu Pico.
- Přesnost: Přesnost měření je stejná – přesnost polohy je stále limitována driftem, ale přenos dat je spolehlivý.
- Cena: Velmi nízká cena (lehce vyšší než Pico bez Wi-Fi, 300 - 700 Kč za komplet).
![DIYvesta](DIYvesta.png) Vygenerováno Google Gemini
#### Příklad kódu v MicroPythonu
![kod](kod.png) S vytvořením kódu mi pomohlo Google Gemini

### Srovnání: Komerční vs. DIY řešení
| Aspekt | Komerční Systém (STATSports) | DIY Řešení (Pico W) |
| :--- | :--- | :--- |
| **Primární mikrokontroler/Čip** | Speciálně optimalizované ARM Cortex-M čipy, ASIC (Application-Specific Integrated Circuit) pro přesnost | RP2040 (Dvoujádrový) čip, navržený pro rychlé zpracování I/O |
| **Senzor Polohy** | Multi-band GNSS (GPS, GLONASS, Galileo) s korekcemi | Žádný (lze připojit levný GPS modul, ale přesnost pro rychlý fotbal je nízká) |
| **Inerciální senzor (IMU)** | Vysoká třída, nízký šum, vysoká vzorkovací frekvence => 100 Hz | Spotřebitelská třída (např. MPU-6050, MPU-9250), vyšší šum |
| **Sledované metriky** | Přesná poloha (heatmapy), maximální rychlost, počet sprintů, akcelerační zátěž. | Zrychlení v reálném čase, Relativní rotační změny. Rychlost/Vzdálenost pouze jako odhad s chybou (driftem). |
| **Řešení chyby (drift)** | Pokročilá fúze senzorů (GNSS + IMU) + proprietární Kalmanovy filtry. | Nutnost použít Kalmanův filtr nebo Doplněný filtr (Complementary Filter) vlastními silami. |
| **Bezdrátová komunikace** | Vysokofrekvenční RF modul (široký dosah, spolehlivost) nebo Bluetooth LE. | Integrované Wi-Fi/Bluetooth (vhodné pro lokální síť, dosah limitován standardem). |
| **Napájení a výdrž** | Optimalizované obvody, baterie s výdrží 6-10 hodin. | Běžná Li-Pol baterie. Výdrž silně závislá na použití Wi-Fi a kódu. |
| **Účel použití** | Monitorování fyziologické zátěže, taktická analýza (kde se hráč pohyboval). | Demonstrace principů, testování algoritmů (filtrace dat), měření relativní zátěže (akcelerace). |
| **Požadované znalosti** | Žádné (jen uživatelská úroveň). | Hardware (I2C, pájení), programování (MicroPython/C++), matematika (filtrace dat). |
| **Přibližná cena** | 25 000 Kč+ za jednotku + roční licenční poplatky. | 300 - 700 Kč za komplet |
