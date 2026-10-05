# Homelab

Homelab infrastruktūra

Savarankiškai administruojama namų laboratorija, pastatyta ant Proxmox VE hipervizoriaus, su tinklo segmentacija, automatizuota infrastruktūros valdymu ir pilnu observability stack'u. Visos paslaugos diegiamos ir valdomos per docker-compose, infrastruktūra kuriama ir palaikoma Infrastructure as Code principais.

Architektūra

<img width="2325" height="1478" alt="homelab-scheme" src="https://github.com/user-attachments/assets/62636d06-af95-4ec6-b288-68aaf247867e" />



Host'inamų paslaugų apžvalga
| Kategorija             | Technologija           | Paskirtis                                                                            |
| ---------------------- | ---------------------- | ------------------------------------------------------------------------------------ |
| Hipervizorius          | Proxmox VE             | VM ir LXC konteinerių valdymas, virtualūs tinklo tiltai                              |
| Firewall / Router      | OPNsense               | VLAN segmentacija, firewall taisyklės, NAT, DNS resolveris                           |
| VPN                    | Netbird                | Saugus nuotolinis prisijungimas prie namų tinklo be atviro RDP/SSH į WAN             |
| Reverse Proxy          | Nginx Proxy Manager    | HTTPS terminavimas, sertifikatų valdymas, srauto maršrutizavimas į vidines paslaugas |
| DNS / Ad-blocking      | Pi-hole                | Tinklo lygmens reklamų blokavimas, vidinis DNS resolveris                            |
| Konteinerių valdymas   | Portainer              | Visų docker konteinerių centralizuota priežiūra ir monitoringas                      |
| Monitoringas           | Grafana + Loki + Alloy | Metrikų ir logų agregavimas, vizualizacija, distribucinis observability              |
| Automatizacija         | Ansible                | Konfigūracijos valdymas, kartotinis serverių paruošimas                              |
| Infrastructure as Code | Terraform              | VM/LXC resursų deklaratyvus valdymas Proxmox aplinkoje                               |
| VM provisioning        | Cloud-init             | Automatinis naujų virtualių mašinų paruošimas be rankinio įsikišimo                  |

Tinklo segmentacija (OPNsense)

Tinklas suskirstytas į izoliuotus VLAN segmentus pagal pasitikėjimo lygį t.y. nuo administratoriaus valdymo tinklo iki viešai pasiekiamų paslaugų DMZ zonoje. Kiekvienas segmentas turi savo firewall taisyklių rinkinį, paremtą principu "default deny", kas reiškia, jog leidžiama tik tai, kas aiškiai reikalinga, viskas kita blokuojama pagal nutylėjimą.

    TRUSTED — administratoriaus valdymo tinklas su prieiga prie firewall ir kritinių resursų
    SERVERS — Docker host'ai, monitoringo stack'as, konteinerių infrastruktūra
    DMZ — viešai pasiekiamos paslaugos (reverse proxy, VPN control plane), laikomos potencialiai kompromituotomis
    GUEST — izoliuotas svečių tinklas, tik su interneto prieiga

Nuotolinis valdymas

Nuotolinis prisijungimas prie namų infrastruktūros vyksta per Netbird VPN, nėra jokių tiesiogiai atvertų valdymo portų (SSH, RDP) į viešą internetą. Vieninteliai iš išorės pasiekiami portai — 80, 443 ir 3478 yra atverti į DMZ segmente esančią Nginx Proxy Manager virtualią mašiną, kuri veikia kaip vienintelis įėjimo taškas iš viešo interneto. Iš ten NPM atlieka port forward į Netbird konteinerį (control panel), o routing peer'as užtikrina saugų maršrutą į vidinius VLAN segmentus.

Monitoring'as

Pilnas monitoring'o stack'as, veikiantis per docker-compose:

    Grafana — vizualizacija ir dashboard'ai
    Loki — logų agregavimas iš visų konteinerių ir VM
    Alloy — distribucinis telemetrijos agentas, renkantis metrikas ir logus iš skirtingų mazgų

Automatizacija ir IaC

    Terraform — VM/LXC resursų aprašymas ir valdymas Proxmox aplinkoje
    Ansible — konfigūracijos (docker) automatizavimas, paketų diegimas, servisų valdymas naujai sukurtose mašinose
    Cloud-init — automatinis pradinis VM paruošimas (SSH raktai, tinklo nustatymai, hostname) diegimo metu, be rankinio įsikišimo

Konteinerizacija

Visos paslaugos homelab'e diegiamos ir valdomos per docker-compose YAML failus. Portainer naudojamas kaip centralizuota valdymo sąsaja visiems konteineriams, veikiantiems skirtingose VM/LXC konteineriuose visame tinkle.

