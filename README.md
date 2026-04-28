# Network Engineering Project
**nazwa:** House#1  
**opis:** Budowa profesjonalnej sieci LAN z wdrożeniem najwyżego bezpieczeństwa (Zero Trust Architecture).  .  

### Zadania:  
1. **Podział sieci na VLANY** - kompletna izolacja środowiska Windows Server, systemu monitoringu i użytkowników domowych.
2. **Instalacja VPN (WireGuard)** - dla zdalnego, szyfrowanego dostępu do monitoringu i systemu smart home.
3. **Adresacja urządzeń**  
  3.1 **Routing statyczny** - tam gdzie to możliwe  
  3.2 **DHCP Static-Only + ARP Reply-Only\*** - w przypadku urządzeń takich jak np. kamery (połączonych z routerem przez niezarządzalny POE_switch)  
  3.3 **DHCP** - dla użytkowników domowych
>Używamy DHCP Static-Only do centralnego przypisania urządzeniom stałego adresu IP po adresie MAC, oraz ARP Reply-Only dla zabezpieczenia przez dołączeniem do sieci niechcianych urządzeń z ustawionym IP. 

---
  
### Użyte urządzenia:
* **Modem ISP:** HALNY (w trybie routera z DMZ)
* **Main Router:** MikroTik ax3 (Layer 3 & Firewall)
* **Managed Switch:** Tenda TEG2208D (Layer 2)
* **Unmanaged Switch:** 1x (Gigabit dumb-switch)
* **PoE Switch:** 2x (do zasilania kamer)
* **Access Point:** 1x (TP-Link Archer c6 - dawny główny router )

---

### Spis treści:  
1. [Schemat administracyjny](#1-schemat-administracyjny-sieci) - grafika  
  1.1. [Konfiguracja sieci LAN](#11-konfiguracja-sieci-lan) - rozpiska adresacji  
2. [Schemat urządzeń i kabli](#2-schemat-rozłożenia-sieci-i-urządzeń) - grafika  
  2.1. [Połączenia fizyczne](#21-połączenia-fizyczne) - konfiguracja interfejsów
3. [Polityka Bezpieczeństwa (Firewall)](#3-polityka-bezpieczeństwa-firewall) - reguły ruchu
4. [Konfiguracja VPN (WireGuard)](#4-konfiguracja-vpn-wireguard) - zdalny dostęp

---

## 1. Schemat administracyjny sieci
![Schamat administracyjny sieci](schemat_administracyjny.png)

## 1.1 Konfiguracja sieci LAN
| VLAN | Network Address | Maska / CIDR | Host IP Range | Default Gateway | IP Assign | DHCP Pool |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | 
| **100** | `192.168.10.0` | `255.255.255.248` (**/29**) | `.10.1` - `.10.6` | `192.168.10.1` | Static | - | 
| **200** | `192.168.20.0` | `255.255.255.0` (**/24**) | `.20.1` - `.20.254` | `192.168.20.1` | Static + DHCP | `.100` - `.254` |
| **300** | `192.168.30.0` | `255.255.255.224` (**/27**) | `.30.1` - `.30.30` | `192.168.30.1` | Static-Only | - | 

 
## 2. Schemat rozłożenia sieci i urządzeń
![Schamat rozmieszczenia sieci](schemat_rozmieszczenia.png)

## 2.1. Połączenia fizyczne

**Modem ISP (HALNY router)**
| Funkcja | Ustawienie | Wartość | Uwagi |
| :--- | :--- | :--- | :--- |
| **LAN IP** | Adres bramy ISP | `192.168.0.1` | Domyślny adres podsieci dostawcy |
| **DMZ** | Adres docelowy | `192.168.0.2` | Przekierowanie całego ruchu z zewnątrz bezpośrednio na port WAN MikroTika (niezbędne dla VPN) |
| **DHCP Server** | Rezerwacja IP | `192.168.0.2` | Przypisane na stałe do adresu MAC portu ether1 w MikroTiku |

**Router główny (MikroTik ax3)**
| Interfejs | Funkcja / VLAN | Status Portu | Adresacja | Brama | Urządzenie docelowe | 
| :--- | :--- | :--- | :--- | :--- |  :--- |
| **ether1** | WAN | Domyślny | `192.168.0.2` (DMZ na ISP) | | ISP Modem (`192.168.0.1`) | 
| **ether2** | VLAN 200 | Access (Untagged) | `192.168.20.2` | `192.168.20.1` | Access Point | 
| **ether3** | VLAN 200 | Access (Untagged) | DHCP | `192.168.20.1` | dumb_switch (Użytkownicy) | 
| **ether4** | VLAN 300 | Access (Untagged) | DHCP Static-Only | `192.168.30.1` | POE_switch (strych - Kamery) |
| **ether5** | VLAN 100, 200, 300 | **Trunk (Tagged)** | **192.168.20.3** | - | Tenda Switch (mały pokój) |
| **wlan1** | VLAN 200 | AP | DHCP | `192.168.20.1` | Wi-Fi 2.4GHz Users |
| **wlan2** | VLAN 200 | AP | DHCP | `192.168.20.1` | Wi-Fi 5GHz Users |

> "dumb_switch" służy za rozszerzenie liczby portów routera, podłączymy tam urządzenia niewymagające konfiguracji IP.  
POE_switch rónież należy do urządzeń "dumb", jedynie zasili kamery i połączy je z siecią.

**Switch Zarządzalny L2 (Tenda TEG2208D - Mały pokój)**
| Port | Funkcja / VLAN | Status Portu | Adresacja | Urządzenie docelowe | Uwagi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **CPU/Internal**| Management | **VLAN 200** | `192.168.20.3` | - | IP do zarządzania switchem |
| **Port 1** | Trunk | **Tagged** | - | MikroTik `ether5` | Uplink do routera |
| **Port 2** | VLAN 100 | Access | `192.168.10.2` | Windows Server | Static |
| **Port 3** | VLAN 200 | Access | `192.168.20.90` | Drukarka | Static |
| **Port 4** | VLAN 300 | Access | `192.168.30.2` | HA Server | Static |
| **Port 5** | VLAN 300 | Access | `192.168.30.3` | Old NVR | Static |
| **Port 6** | VLAN 300 | Access | - | POE_switch (pokój) | Dumb POE switch |
| **Port 7** | - | - | - | - |
| **Port 8** | - | - | - | - |

## 3. Polityka Bezpieczeństwa (Firewall)

Zasady filtrowania ruchu oparte na modelu **Zero Trust**. Każdy ruch między VLANami jest domyślnie blokowany (`Drop`), z wyjątkiem jawnie zdefiniowanych reguł.

### 3.1. Reguły Przekierowań (NAT / Serwer "na świat")
* **Destination NAT (WAN -> 192.168.10.2):** Przekierowanie wybranych portów usługowych na Windows Server.

### 3.2. Reguły Filtrowania (Łańcuch: Forward)
Wszystkie poniższe reguły dotyczą ruchu *przechodzącego* przez router (z jednej sieci do drugiej). Kolejność w tabeli odpowiada dokładnej kolejności ustawienia w panelu MikroTik.

| Nr | Akcja | Source | Destination| Opis |
| :--- | :--- | :--- | :--- | :--- |
| **1** | `Accept` | **Dowolne** | **Dowolne** <br>*(Stan: Established / Related)* | **Podtrzymanie rozmowy**  |
| **2** | `Accept` | **Internet (WAN)** | **Windows Server** <br>`192.168.10.2` <br>*(Stan: DST-NAT)* | **Otwarcie serwera na świat** Wpuszcza z internetu tylko ten ruch, dla którego utworzono przekierowania portów |
| **3** | `Accept` | **Użytkownicy (VLAN 200)** <br>`192.168.20.0/24` | **Internet (WAN)** | **Wyjście w świat** |
| **4** | `Accept` | **Użytkownicy (VLAN 200)** <br>`192.168.20.0/24` | **Windows Server** <br>`192.168.10.2` | **Dostęp LAN -> Serwer:** Pozwala domownikom korzystać z udostępnionych plików, DNS czy innych lokalnych usług na serwerze Windows. |
| **5** | `Accept` | **Użytkownicy (VLAN 200)** <br>`192.168.20.0/24` | **HA i NVR (VLAN 300)** <br>`192.168.30.2`, `.30.3` | **Dostęp LAN -> Monitoring:** Domownicy mogą połączyć się z Home Assistantem oraz rejestratorem. (nie dajemy dostępu do całej podsieci!) `.30.x`, a jedynie do konkretnych urządzeń. |
| **6** | **DROP** | **Monitoring (VLAN 300)** <br>`192.168.30.0/27` | **Internet (WAN)** | **Kwarantanna (Izolacja IoT):** Bezwzględny zakaz wyjścia do internetu dla kamer i rejestratora. |
| **7** | **DROP** | **Dowolne** | **Dowolne** | **Zero Trust (Final Drop):** Inny ruch (np. kamera próbująca połączyć się z serwerem, albo serwer próbujący połączyć się z komputerem domowym). Cały taki ruch jest  blokowany. |

---

## 4. Konfiguracja VPN (WireGuard)
>VPN funkcjonuje jako niezależny, wirtualny interfejs sieciowy w routerze z własną podsiecią. Każde zaufane urządzenie (Peer) otrzymuje unikalny stały adres IP wewnątrz tunelu.

### 4.1 Adresacja Wirtualnej Sieci VPN
| Sieć VPN | Adres Serwera (MikroTik) | Maska Sieci |
| :--- | :--- | :--- |
| `10.0.0.0/24` | `10.0.0.1` | `255.255.255.0` |

**Klienci VPN (Peers):**
| Urządzenie | Adres IP (wewnątrz tunelu) |
| :--- | :--- |
| **telefon#1** | `10.0.0.2/32` | 
| **laptop#1** | `10.0.0.3/32` |

### 4.2 Reguły Sieciowe i Zapory dla VPN
Aby tunel działał bezpiecznie, wdrożono następujące reguły na routerze MikroTik:

* **Otwarcie portu nasłuchowego (Input):**
  * Zezwolenie na ruch UDP na porcie `51820` z interfejsu WAN. Zapewnia możliwość nawiązania szyfrowanego połączenia.
* **Prawa klientów VPN (Forward - dodane przed regułą Final Drop):**
  * **Zezwól:** Klient VPN (`10.0.0.0/24`) `->` **HA i NVR** (`192.168.30.2`, `.30.3`). Umożliwia podgląd kamer i sterowanie automatyką domową.
  * **Zezwól:** Klient VPN (`10.0.0.0/24`) `->` **Windows Server** (`192.168.10.2`). Umożliwia zdalne zarządzanie serwerem, RDP itp.
  * *Komputery i drukarki w sieci domowej (VLAN 200) pozostają całkowicie niewidoczne dla klientów podłączonych przez VPN.

