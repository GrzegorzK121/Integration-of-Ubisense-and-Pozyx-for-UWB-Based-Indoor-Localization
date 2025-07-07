# UWB-RTLS-Corridor-Demo

Implementacja korytarzowego pola testowego RTLS opartego na UWB (system Ubisense).
Repo służy do wersjonowania konfiguracji sieci, skryptów kalibracyjnych
i narzędzi analitycznych (Python/Jupyter).

<div align="center">
  <img src="docs/img/corridor_overview.jpg" width="650">
</div>

## 📐 Topologia

| Warstwa | Sprzęt / Oprogramowanie | Status |
|---------|-------------------------|--------|
| Kotwice | 10 × **Ubisense Series 7000** (AoA + ToF) | ✅ zamontowane |
| Switch  | **Planet GSD-804P** (8×GbE, 65 W PoE)        | ✅ w suficie |
| Core    | Laptop z Ubuntu 22.04 + `rtlsd` (Ubisense) | ✅ on-line |
| LAN     | CAT6, własnoręcznie konfekcjonowane         | ✅ przetestowane |

## ✔️ Co już działa

* **Zawieszenie** kotwic na niestandardowych wysięgnikach
  (profile AL + drewniana baza, izolacja drgań sufitu podwieszanego).  
* **Okablowanie PoE** – patch-panele zaślepione; switch zasilany z listwy UPS.  
* **Adresacja** – VLAN 100 (kotwice), VLAN 200 (backend), rezerwacje DHCP.  
* **Backend** – `rtlsd` + InfluxDB + MQTT (publikacja 10 Hz).  
* Python `/scripts/stream2csv.py` → zapisuje raw XYZ + timestamp.  
* Wstępna kalibracja offsetów AoA (błąd ~12 cm @ 14 m).  

## 🖼️ Galeria

| # | Podgląd | Opis |
|---|---------|------|
| 1 | ![image](https://github.com/user-attachments/assets/fb79224c-5b5f-412c-af84-f2fe2497fd9e)

| 2 | ![](docs/img/02_switch_laptop.jpg) | **Stolik serwisowy** – laptop z `ssh`, switch PoE z identyfikacją portów (żółte tagi). |
| 3 | ![](docs/img/03_switch_close.jpg) | Zbliżenie na **Planet GSD-804P**, LED status PoE OK. |
| 4 | ![](docs/img/04_ceiling_selfie.jpg) | Selfie podczas przeciągania skrętki nad sufitem podwieszanym. |
| 5 | ![](docs/img/05_cable_test.jpg) | Testery RJ-45 + zaciskarka – konfekcja patchy 15 m. |
| 6 | ![](docs/img/06_switch_ceiling.jpg) | Switch zainstalowany w korycie kablowym (nad klatką schodową). |
| 7 | ![](docs/img/07_ubisense_pair.jpg) | Para kotwic okienna – symetryczne rozstawienie do triangulacji 2D. |
| 8 | ![](docs/img/08_anchor_farm.jpg) | „Farma” – komplet 10 kotwic na podstawach, gotowe do montażu. |

> **Uwaga**  
> Pliki source znajdują się w `docs/img/`; do README dodaj miniaturki
> lub linkuj pełne JPG.

## 🛠️ TODO / Roadmap

- [ ] Dodać kalibrację z tachimetrem (Ground Truth).  
- [ ] Skrypt `aoa_heatmap.py` – wizualizacja błędu kątowego.  
- [ ] Integracja tagów **Pozyx** → porównanie z Ubisense.  
- [ ] Dashboard Grafana (live xy/σ).  
- [ ] Dockerfile + docker-compose dla całego stosu (rtlsd + MQTT + Influx).  
- [ ] Instrukcja „deployment from scratch”.

## ⚙️ Wymagania

| Komponent | Wersja |
|-----------|--------|
| Python    | 3.9+   |
| Ubisense SDK | 3.7 |
| InfluxDB  | ≥2.0 |
| Grafana   | ≥9    |

---

MIT License © 2023 YourName
