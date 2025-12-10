# Rocnikovy-projekt
## Cíl projektu
Cílem tohoto projektu je porovnat, jak se v profesionálním fotbale používají drahé monitorovací systémy (vesty s GPS) a jak bychom mohli podobnou funkci levně napodobit pomocí mikrokontrolerů (jako je např. Raspberry Pi Pico, Arduino, ESP32) a levných senzorů.

Zaměřím se na to, co umí drahé systémy přesně změřit (polohu, rychlost míče nebo hráče), navrhnu vlastní prototyp založený na mikrokontroleru, který nabídne cenově dostupnou alternativu a zjistím jaké problémy můžou nastávat s levnými součástkami (např. chyba v měření vzdálenosti).
## Průzkum
### Komerční systémy
Komerční sportovní analytické systémy se zaměřují na přesné a komplexní monitorování výkonu celého týmu v reálném čase.
#### Příklad: Systémy Catapult a STATSports
Typ zařízení: Vesta/Podprsenka s malým boxem na zádech.

Primární senzory: GNSS (GPS + GLONASS), nebo LPS (ClearSky, vlastní lokální polohový systém společnosti Catapult) pro venkovní sledování polohy. Akcelerometry, Gyroskopy a Magnetometry (IMU – Inertial Measurement Unit) pro sledování pohybu, zrychlení, zpomalení a směru.

Sledované metriky: Rychlost (maximální, průměrná), uběhlá vzdálenost (celková, v různých rychlostních zónách), zrychlení/zpomalení (počet a intenzita), srdeční tep (pomocí externího hrudního pásu nebo integrovaného optického senzoru).

Zpracování dat: Zařízení obsahuje výkonný mikrokontroler (často na bázi ARM Cortex architektury) a specializované čipy pro zpracování senzorických dat a fúzi senzorů. Data jsou logována do interní paměti a/nebo přenášena v reálném čase.

Komunikace: Vysokofrekvenční rádiové přenosy (pro data v reálném čase na krátkou vzdálenost k přijímači u hřiště) nebo Bluetooth/Wi-Fi pro stažení logovaných dat po tréninku.

Přesnost: Vysoká přesnost (poloha s přesností +- 1 metru, zrychlení s vysokou vzorkovací frekvencí až 100 Hz).

Cena:Vysoká cena (2 500 kč+ za jednotku + licence).
