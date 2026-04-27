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

***
  
### Użyte urządzenia:
- main router - Miktorik ax3 (Layer 3 & Firewall)
- switch zarządzalny - Tenda TEG2208D (Layer 2)
- switch niezarządzalny x1
- switch POE x2 (do zasilania kamer)
- Access Point (wcześniej pracujący w sieci jako router)

***

### Spis treści:  
1. [Schemat administracyjny](https://github.com/Matix732/network_engineering-house1#1-schemat-administracyjny-sieci) - grafika  
1.1. [Konfiguracja sieci](https://github.com/Matix732/network_engineering-house1#11-konfiguracja-sieci-lan) - rozpiska  
2. [Schemat urządzeń i kabli](https://github.com/Matix732/network_engineering-house1#2-schamat-roz%C5%82o%C5%BCenia-sieci-i-urz%C4%85dze%C5%84) - grafika
3. [21 test](#21-połączenia-fizyczne)
2.1. [Połączenia fizyczne](https://github.com/Matix732/network_engineering-house1/blob/main/README.md#21-po%C5%82%C4%85czenia-fizyczne) - konfiguracja urządzeń   

***

## 1. Schemat administracyjny sieci
![Schamat administracyjny sieci](schemat_administracyjny.png)

## 1.1 Konfiguracja sieci LAN
| VLAN | Network Address | Maska / CIDR | Host IP Range | Default Gateway | IP Assign | DHCP Pool |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | 
| **100** | `192.168.10.0` | `255.255.255.248` (**/29**) | `.1` - `.6` | `192.168.100.1` | Static | - | 
| **200** | `192.168.20.0` | `255.255.255.0` (**/24**) | `.1` - `.254` | `192.168.200.1` | Static + DHCP | `.100` - `.254` |
| **300** | `192.168.30.0` | `255.255.255.224` (**/27**) | `.1` - `.30` | `192.168.30.1` | Static-Only | - | 

## 2. Schamat rozłożenia sieci i urządzeń
![Schamat rozmieszczenia sieci](schemat_rozmieszczenia.png)

## 2.1. Połączenia fizyczne

**ISP router-modem**


**router Mikrotik**
| interfejs | VLAN | adres | urządzenie | 
| --- | --- | --- | --- | 
| ether1 | -   | 192.168.0.1 | ISP modem | 
| ether2 | 200 | 192.168.200.2 | Access Point | 
| ether3 | 200 | DHCP | dumb_switch* | 
| ether4 | 300 | 192.168.300.2 - .31 | POE_switch* |
| ether5.1 | 100 | 192.168.100.2 | Tenda switch |
| ether5.2 | 200 | 192.168.200.2 | Tenda switch |
| ether5.3 | 300 | 192.168.300.2 | Tenda switch |
| wlan1 | 200 | DHCP | WIFI users |
| wlan2 | 200 | DHCP | WIFI users |

> "dumb_switch" służy za rozszerzenie liczby portów routera, podłączymy tam urządzenia niewymagające konfiguracji IP.  
POE_switch rónież należy do urządzeń "dumb", jedynie zasili kamery i połączy je z siecią.

**switch Tenda**

