# Homelab

Home Lab Infrastructure
Savarankiškai administruojama namų laboratorija, pastatyta ant Proxmox VE hipervizoriaus, su tinklo segmentacija, automatizuota infrastruktūros valdymu ir pilnu observability stack'u. Visos paslaugos diegiamos ir valdomos per Docker Compose, infrastruktūra kuriama ir palaikoma Infrastructure as Code principais.

Architektūra
(Nuotrauka)

Stack apžvalga
| Kategorija             | Technologija           | Paskirtis                                                                            |
| ---------------------- | ---------------------- | ------------------------------------------------------------------------------------ |
| Hipervizorius          | Proxmox VE             | VM ir LXC konteinerių valdymas, virtualūs tinklo tiltai                              |
| Firewall / Router      | OPNsense               | VLAN segmentacija, firewall taisyklės, NAT, DNS resolveris                           |
| VPN                    | Netbird                | Saugus nuotolinis prisijungimas prie namų tinklo be atviro RDP/SSH į WAN             |
| Reverse Proxy          | Nginx Proxy Manager    | HTTPS terminavimas, sertifikatų valdymas, srauto maršrutizavimas į vidines paslaugas |
| DNS / Ad-blocking      | Pi-hole                | Tinklo lygmens reklamų blokavimas, vidinis DNS resolveris                            |
| Konteinerių valdymas   | Portainer              | Visų Docker konteinerių centralizuota priežiūra ir monitoringas                      |
| Monitoringas           | Grafana + Loki + Alloy | Metrikų ir logų agregavimas, vizualizacija, distribucinis observability              |
| Automatizacija         | Ansible                | Konfigūracijos valdymas, kartotinis serverių paruošimas                              |
| Infrastructure as Code | Terraform              | VM/LXC resursų deklaratyvus valdymas Proxmox aplinkoje                               |
| VM provisioning        | Cloud-init             | Automatinis naujų virtualių mašinų paruošimas be rankinio įsikišimo                  |

Tinklo segmentacija (OPNsense)
Tinklas suskirstytas į izoliuotus VLAN segmentus pagal pasitikėjimo lygį — nuo administratoriaus valdymo tinklo iki viešai pasiekiamų paslaugų DMZ zonoje. Kiekvienas segmentas turi savo firewall taisyklių rinkinį, paremtą principu "default deny" — leidžiama tik tai, kas aiškiai reikalinga, viskas kita blokuojama pagal nutylėjimą.

    TRUSTED — administratoriaus valdymo tinklas su prieiga prie firewall ir kritinių resursų
    SERVERS — Docker host'ai, monitoringo stack'as, konteinerių infrastruktūra
    DMZ — viešai pasiekiamos paslaugos (reverse proxy, VPN control plane), laikomos potencialiai kompromituotomis
    GUEST — izoliuotas svečių tinklas, tik su interneto prieiga

Nuotolinis valdymas
Visas nuotolinis prisijungimas prie namų infrastruktūros vyksta per Netbird VPN — nėra jokių tiesiogiai atvertų valdymo portų (SSH, RDP) į viešą internetą. Netbird control plane veikia DMZ segmente už reverse proxy, o routing peer'as užtikrina saugų maršrutą į vidinius VLAN segmentus, nepažeidžiant esamos tinklo segmentacijos.

Observability
Pilnas monitoringo stack'as, veikiantis per Docker Compose:

    Grafana — vizualizacija ir dashboard'ai
    Loki — logų agregavimas iš visų konteinerių ir VM
    Alloy — distribucinis telemetrijos agentas, renkantis metrikas ir logus iš skirtingų mazgų

Automatizacija ir IaC
    Terraform — deklaratyvus VM/LXC resursų aprašymas ir valdymas Proxmox aplinkoje
    Ansible — konfigūracijos automatizavimas, paketų diegimas, servisų valdymas naujai sukurtose mašinose
    Cloud-init — automatinis pradinis VM paruošimas (SSH raktai, tinklo nustatymai, hostname) diegimo metu, be rankinio įsikišimo

Konteinerizacija
Visos paslaugos šiame home lab'e diegiamos ir valdomos per Docker Compose YAML failus, laikomus versijų kontrolėje. Portainer naudojamas kaip centralizuota valdymo sąsaja visiems konteineriams, veikiantiems skirtingose VM/LXC visame tinkle.
📁 Repo struktūra

(Papildyti pagal realią repo struktūrą — pvz. /docker-compose, /terraform, /ansible, /docs)
