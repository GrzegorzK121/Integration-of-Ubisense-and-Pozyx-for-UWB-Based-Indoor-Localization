# UWB-RTLS-Corridor-Demo

Implementacja korytarzowego pola testowego RTLS opartego na UWB (system Ubisense).
Repo służy do wersjonowania konfiguracji sieci, skryptów kalibracyjnych
i narzędzi analitycznych (Python/Jupyter).

<div align="center">
  <img src="docs/img/corridor_overview.jpg" width="650">
</div>

## Topologia sieci RTLS (Ubisense + Pozyx)

Poniższy schemat przedstawia fizyczną topologię sieci, w której równolegle działają dwa systemy lokalizacji czasu rzeczywistego: Ubisense oraz Pozyx. Oba systemy pracują we wspólnej infrastrukturze Ethernet zasilanej przez przełączniki PoE.

![Topologia RTLS](./Topologia2.png)

### Opis struktury

Router z dostępem do Internetu połączony jest z głównym switchem PoE. Z tego switcha wychodzą trzy połączenia:

- do kontrolera RTLS Pozyx
- do dwóch anchorów Ubisense
- do drugiego switcha PoE (kaskadowo)

Z drugiego switcha PoE wychodzą połączenia:

- do kontrolera RTLS Ubisense
- do dwóch kolejnych anchorów Ubisense

Kontroler Pozyx jest bezpośrednio połączony z czterema anchorami Pozyx, które są zasilane i komunikują się przez Ethernet.

### Lista połączeń

Router → Switch PoE #1  
Switch PoE #1 → RTLS Pozyx Controller  
Switch PoE #1 → Anchor Ubisense (x2)  
Switch PoE #1 → Switch PoE #2  
Switch PoE #2 → RTLS Ubisense Controller  
Switch PoE #2 → Anchor Ubisense (x2)  
RTLS Pozyx Controller → Anchor Pozyx (x4)

### Uwagi

Topologia umożliwia niezależną konfigurację każdego z systemów, przy jednoczesnym współdzieleniu infrastruktury zasilająco-transmisyjnej. Układ został dobrany tak, aby możliwa była szybka rekonfiguracja lub odseparowanie systemów w warstwie logicznej (np. VLAN, overlay, bridge).

## Możliwe rozwiązania aplikacji

Dane z systemów RTLS (Ubisense oraz Pozyx) mogą być odbierane i przetwarzane przez wspólną aplikację zbierającą. Głównym celem jest unifikacja danych lokalizacyjnych przesyłanych przez UDP i udostępnienie ich w czasie rzeczywistym do dalszych modułów (np. API, wizualizacja, system MES).

Poniżej przedstawiono dwa niezależne sposoby realizacji aplikacji – w oparciu o wirtualne maszyny oraz o kontenery z interpreterem Pythona.

### Rozwiązanie 1: UDP na wirtualnych maszynach

Każdy komponent aplikacji działa na oddzielnej maszynie wirtualnej (np. VirtualBox, Proxmox, KVM). Maszyny są połączone wewnętrzną siecią bridge, a połączenia UDP realizowane są między adresami prywatnymi.

(./topologiaWirtualki.png)

Struktura:
- VM1: odbiornik UDP z Sewio (port 5300)
- VM2: odbiornik UDP z Ubisense (port 47474)
- VM3: normalizer – aplikacja Pythona (FastAPI + asyncio) łącząca dane z obu źródeł
- VM4: API Gateway – udostępniający dane przez REST oraz WebSocket

Każdy odbiornik zapisuje dane w postaci surowej i przekazuje je do normalizatora przez asyncio Queue (socket lokalny lub TCP). Dane są ujednolicane do formatu {id, x, y, z, t} i udostępniane przez API.

### Rozwiązanie 2: Przetwarzanie UDP w kontenerach z frontendem poza serwerem

Poniższy diagram przedstawia sposób odbioru danych lokalizacyjnych z sieci UDP w kontenerach Docker/Podman oraz ich ekspozycję do zewnętrznych aplikacji. Frontend działa poza hostem kontenerowym i łączy się z API udostępnianym przez backend.

Schemat zakłada:
- dwa źródła danych UDP (wcześniej opisane),
- kontenery: ingest, normalizer, API,
- frontend działający niezależnie (np. przeglądarka, MES, dashboard).

(./topologiaKontener.png)

Architektura jest w pełni modularna, gotowa do wdrożenia w Docker Compose lub Podman Pod. Komunikacja wewnętrzna kontenerów może opierać się o kolejki asynchroniczne (np. asyncio), sockety lub lokalny TCP. API udostępnia dane lokalizacyjne w formacie ujednoliconym `{id, x, y, z, t}`.





