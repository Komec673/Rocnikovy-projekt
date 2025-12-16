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
![DIYvesta](DIYvesta.png)

Vygenerováno pomocí Google Gemini
#### Příklad kódu v MicroPythonu
![kod](kod.png) 

S vytvořením kódu mi pomáhalo Google Gemini
#### Hlavní chyby a problémy u DIY řešení:
##### Chyby spojené s měřením
1. Drift
- Problém: Největší a nejsložitější problém. Akcelerometr měří zrychlení, když toto zrychlení vypočítáme na rychlost a poté na polohu/vzdálenost, každá malá nepřesnost v datech ze senzoru se sčítá.
- Výsledek: Po pouhých několika sekundách nehybnosti nebo pohybu se vypočítaná poloha na základě zrychlení dramaticky liší od skutečné polohy. Tím je měření ujeté vzdálenosti a rychlosti nepřesné.
- Řešení: Musíme implementovat komplexní filtry (např. Kalmanův filtr), které se snaží drift matematicky potlačit.

2. Šum a vibrace
- Problém: Levné senzory generují elektronický šum. Senzor je navíc připevněn na těle hráče, což způsobuje mechanické vibrace (např. při dopadu nohy, sprintu).
- Výsledek: Senzor vnímá vibrace jako skutečné zrychlení a zkresluje data.
- Řešení: Aplikace digitálních filtrů (např. dolní propust) ve zpracování dat, které odstraní vysokofrekvenční šum a vibrace předtím, než se data použijí pro integraci.

3. Teplotní závislost
- Problém: Senzor IMU není teplotně kompenzován. Změny teploty (např. zahřátí čipu při provozu nebo změna venkovní teploty) mohou ovlivnit jeho kalibraci.
- Řešení: Nutnost provést kalibraci za běhu nebo během inicializace (např. měření průměrného offsetu os na začátku tréninku).
##### Chyby spojené s výpočtem a kalibrací
1. Statické zrychlení
   - Problém: Akcelerometr měří jak pohybové zrychlení, tak gravitaci. Pokud není senzor dokonale v rovině (což na fotbalistovi nikdy nebude), gravitační zrychlení unikne do všech tří os (X, Y, Z).
   - Výsledek: Zpracovávaná data zrychlení jsou neustále zkreslena statickou složkou gravitace.
   - Řešení: Použití gyroskopu společně s akcelerometrem (v rámci IMU) k určení aktuální orientace zařízení. To umožní odečíst gravitační složku a ponechat pouze zrychlení způsobené pohybem hráče.

2. Nesprávná kalibrace gyroskopu/akcelerometru
   - Problém: Každý čip má mírně odlišné offsety (nulové body) a škálování. Pokud tyto offsety nejsou správně odečteny na začátku, každé měření bude posunuto.
   - Řešení: Předběžná kalibrace – na začátku tréninku měření nehybného senzoru (např. 1000 vzorků) pro určení průměrné chyby, která se následně od všech budoucích hodnot odečítá.
   
##### Chyby spojené s hardwarem a zapojením
1. Problémy s I2C komunikací
   - Problém: Komunikace mezi Pico W a MPU-6050 probíhá přes I2C sběrnici. Chyby v komunikaci mohou být způsobeny nedostatečným napájením, chybějícími pull-up rezistory (které jsou ale často již na breakout modulech), nebo příliš dlouhými/nekvalitními kabely.
   - Řešení: Zajistit krátké a pevné zapojení a případně snížit rychlost I2C sběrnice, pokud by docházelo k chybám.

2. Nevhodné umístění a uchycení
   - Problém: Pokud je senzor v boxu volně, nebo je box volně uchycen na vestě, dochází k nechtěnému pohybu a vibracím.
   - Řešení: Pevné 3D tištěné pouzdro, kde je senzor pevně fixován a kde je eliminován pohyb samotného senzoru vůči tělu.

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
## Zdroje
Google Gemini. Online. Dostupné z: https://gemini.google.com/app?hl=cs. [cit. 2025-12-16].
Catapult. Online. Dostupné z: https://www.catapult.com/solutions/vector-pro. [cit. 2025-12-16].
Catapult Support. Online. Dostupné z: https://support.catapultsports.com/hc/en-us/articles/360001235615-What-is-inside-Catapult-s-wearable-technology. [cit. 2025-12-16].
Alibaba. Online. Dostupné z: https://www.alibaba.com/product-insights/statsports-vest.html. [cit. 2025-12-16].
