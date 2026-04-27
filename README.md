# Network Engineering Project
**nazwa:** House#1  
**opis:** Budowa profesjonalnej sieci LAN z wdrożeniem najwyżego bezpieczeństwa (Zero Trust Architecture).  .  

### Zadania:  
1. **Podział sieci na VLANY** - kompletna izolacja środowiska Windows Server, systemu monitoringu i użytkowników domowych.
2. **Instalacja VPN (WireGuard)** - dla zdalnego, szyfrowanego dostępu do monitoringu i systemu smart home.
3. **Adresacja urządzeń**  
  3.1 **Routing statyczny** - tam gdzie to możliwe  
  3.2 **DHCP "Static-Only" + ARP Reply-Only\*** - w przypadku urządzeń takich jak np. kamery (połączonych z routerem przez niezarządzalny POE_switch)
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
| Interfejs | Funkcja / VLAN | Status Portu | Adresacja | Urządzenie docelowe | 
| :--- | :--- | :--- | :--- | :--- | 
| **ether1** | WAN | Domyślny | `192.168.0.2` (DMZ na ISP) | ISP Modem (`192.168.0.1`) | 
| **ether2** | VLAN 200 | Access (Untagged) | Brama: `.20.1` | Access Point | 
| **ether3** | VLAN 200 | Access (Untagged) | Brama: `.20.1` | dumb_switch (Użytkownicy) | 
| **ether4** | VLAN 300 | Access (Untagged) | Brama: `.30.1` | POE_switch (strych - Kamery) |
| **ether5** | VLAN 100, 200, 300 | **Trunk (Tagged)** | - | Tenda Switch (mały pokój) |
| **wlan1** | VLAN 200 | Wirtualny Access | Brama: `.20.1` | Wi-Fi 2.4GHz Users |
| **wlan2** | VLAN 200 | Wirtualny Access | Brama: `.20.1` | Wi-Fi 5GHz Users |

> "dumb_switch" służy za rozszerzenie liczby portów routera, podłączymy tam urządzenia niewymagające konfiguracji IP.  
POE_switch rónież należy do urządzeń "dumb", jedynie zasili kamery i połączy je z siecią.

**Switch Zarządzalny L2 (Tenda TEG2208D - Mały pokój)**
| Port | Funkcja / VLAN | Status Portu | Urządzenie docelowe | Uwagi |
| :--- | :--- | :--- | :--- | :--- |
| **Port 1** | VLAN 100, 200, 300 | **Trunk (Tagged)** | MikroTik `ether5` | Uplink z routera |
| **Port 2** | VLAN 100 | Access (Untagged) | Windows Server | Static IP |
| **Port 3** | VLAN 200 | Access (Untagged) | Drukarka sieciowa | Static z poza puli DHCP |
| **Port 4** | VLAN 300 | Access (Untagged) | Home Assistant Server | Static IP |
| **Port 5** | VLAN 300 | Access (Untagged) | Old NVR | Static IP |
| **Port 6** | VLAN 300 | Access (Untagged) | POE_switch | Zasilanie kamer (Mały pokój) |
| **Port 7** | - | - | Pusty | - |
| **Port 8** | - | - | Pusty | - |

