# FortiGate baseline konfiguracija prilikom inicijalizacije uređaja

***[DRAFT VERZIJA!!!]***

Baseline konfiguracija FortiGate firewall-a prilikom inicijalizacije uređaja pre implementacije u produkciono okruženje.

- [Početak dokumenta](#fortigate-baseline-konfiguracija-prilikom-inicijalizacije-uređaja)
	- [Sistemska podešavanja](#sistemska-podešavanja)
		- [Podešavanje imena uređaja](#podešavanja-imena-uređaja)
		- [Upgrade firewall uređaja](#upgrade-firewall-uređaja)
		- [Gašenje opcije automatskog upgrade-a](#gašenje-opcije-automatskog-upgrade-a)
		- [Konfiguracija DNS servera](#konfiguracija-dns-servera)
		- [Konfiguracija NTP servera](#konfiguracija-ntp-servera)
		- [Konfiguracija SNMP servera](#konfiguracija-snmp-servera)
		- [FortiGate HA menadžment interfejs](#fortigate-ha-menadžment-interfejs)
	- [Konfiguracija High-Availability(HA)](#konfiguracija-high-availabilityha)
		- [Inicijalna konfiguracija HA](#inicijalna-konfiguracija-ha)
		- [Replikacija sesija](#replikacija-sesija)
		- [Failover kriterijumi](#failover-kriterijumi)
		- [Failover opcije](#failover-opcije)
		- [Konfiguracija VDOM particija](#konfiguracija-vdom-particija)
	- [Podešavanja interfejsa](#podešavanja-interfejsa)
		- [Blokiranje intra-zone saobraćaja](#blokiranje-intra-zone-saobraćaja)
		- [Gašenje nekorišćenih interfejsa](#gašenje-nekorišćenih-interfejsa)
		- [Brisanje nekorišćenih DHCP servera](#brisanje-nekorišćenih-dhcp-servera)
		- [Gašenje menadžment servisa na svim interfejsima koji nisu za menadžment](#gašenje-menadžment-servisa-na-svim-interfejsima-koji-nisu-za-menadžment)
		- [Definisanje protoka na WAN interfejsima](#definisanje-protoka-na-wan-interfejsima)
		- [Konfiguracija detekcije uređaja](#konfiguracija-detekcije-uređaja)
	- [Administratorski pristup](#administratorski-pristup)
		- [Konfiguracija password polise](#konfiguracija-password-polise)
		- [Konfiguracija administratora](#konfiguracija-administratora)
		- [Konfiguracija Multi-Factor Authentication(MFA) za administratora](#konfiguracija-multi-factor-authenticationMFA-za-administratora)
		- [Konfiguracija break-glass administratora](#konfiguracija-break-glass-administratora)
		- [Konfiguracija administratorskog profila](#konfiguracija-administratorskog-profila)
		- [Modifikovanje podrazumevanih menadžment portova](#modifikovanje-podrazumevanih-menadžment-portova)
		- [Povećavanja timeout-a za administratorski pristup](#povećavanje-timeout-a-za-administratorski-pristup)
		- [Povećavanja timeout-a za idle stanje administratora](#povećavanje-timeout-a-za-idle-stanje-administratora)
		- [Pre-login banner](#pre-login-banner)
		- [Post-login banner](#post-login-banner)
		- [Gašenje USB auto install opcije](#gašenje-usb-auto-install-opcije)
		- [Gašenje FortiCloud SSO pristupa](#gašenje-forticloud-sso-pristupa)
	- [Logovanje i performanse uređaja](#logovanje-i-performanse-uređaja)
		- [Kreiranje revizije nakon logout](#kreiranje-revizije-nakon-logout)
		- [Uključivanje korišćenja CDN-a](#uključivanje-korišćenja-cdn-a)
		- [Uključivanje korišćenja lokalnog ISDB keša](#uključivanje-korišćenja-lokalnog-isdb-keša)
		- [Uključivanje automatske provere diska](#uključivanje-automatske-provere-diska)
		- [Logovanje CLI komandi](#logovanje-cli-komandi)
		- [Proširenje logovanja i prikaza logova](#proširenje-logovanja-i-prikaza-logova)
		- [Uključivanje logovanja na disk](#uključivanje-logovanja-na-disk)
	- [Security profili](#security-profili)


## Sistemska podešavanja


### Podešavanja imena uređaja
Podrazumevana konfiguracija je da je ime uređaja serijski broj. Preporučuje se podešavanje imena bez razmaka sa donjom crtom(_) i crticom(-).
```
config system global
	set hostname <IME-UREĐAJA>
end
```


### Upgrade firewall uređaja
Potrebno je uvek pratiti novosti PSIRT-a vezane za slabosti firmware verzija uređaja. Kada se pronađe slabost, potrebno je da se zakrpi u što kraćem vremenskom roku.

[Technical Tip: Recommended release for FortiOS](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Recommended-release-for-FortiOS/ta-p/227178)


### Gašenje opcije automatskog upgrade-a
Za uređaje koji su vezani na FortiGate Cloud, podrazumevano podešavanje je automatski upgrade na najnoviju verziju firmware-a na istoj major verziji. Preporučuje se gašenje te opcije. 
``` 
config system fortiguard
    set auto-firmware-upgrade disable
end
```

[Technical Tip: Understanding automatic patch upgrade](https://community.fortinet.com/t5/FortiGate-Cloud/Technical-Tip-Understanding-automatic-patch-upgrade-FortiGate/ta-p/316549)


### Konfiguracija DNS servera
Podrazumevana vrednost su FortiGuard DNS serveri. Preporuka je da se promene na interne DNS servere.

U slučaju da ne postoji interni DNS server, preporuka je korišćenje javnog Cisco Umbrella DNS servera:
``` 
208.67.222.222
208.67.220.220
``` 

Pored toga, preporučuje se promena DNS protokola, najčešće je u pitanju DNS preko UDP/TCP porta 53, odnosno cleartext DNS.
``` 
config system dns
    set primary <IP-DNS-PRIMARNI>  
    set secondary <IP-DNS-SEKUNDARNI>  
    set protocol cleartext 
end
```

U slučaju da je to potrebno, FortiGate može biti DNS server:
[Administration Guide: FortiGate DNS server](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/960561/fortigate-dns-server)


### Konfiguracija NTP servera
Podrazumevana vrednost su FortiGuard NTP serveri. Preporuka je da se promene na interne servere sa definisanim NTP servisom.

U slučaju da ne postoji interni NTP server, preporuka je korišćenje javnih popularnih NTP servera:
```
0.pool.ntp.org
time.nist.gov
```

``` 
config system ntp
    set ntpsync enable
    set type custom
    config ntpserver
        edit 0
            set server <FQDN-NTP-PRIMARNI>  
        next
        edit 0
            set server <FQDN-NTP-SEKUNDARNI>
        next
    end
end
```

Pored NTP servera, potrebno je definisati pravilnu vremensku zonu:
``` 
config system global
    set timezone "<IME-TIMEZONE>" 
end
```

Za Srbiju, ispod možete naći našu vremensku zonu:
```
config system global
    set timezone "Europe/Belgrade"
end
```

U slučaju da je okruženje air-gapped, bez dostupnosti internog NTP servera, moguće je ručno definisati NTP server.
``` 
execute date <YYYY-MM-DD>  
execute time <HH:MM:SS>  
```

U slučaju da je to potrebno, FortiGate može biti NTP server:
[Administration Guide: FortiGate DNS server](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Setting-up-an-NTP-server-and-an-NTP-client-using/ta-p/302556)


### Konfiguracija SNMP servera
Najsigurniji SNMP protokol u ovom trenutku je SNMP verzija 3. Međutim, zbog kompleksnosti implementacije polling-a razumljivo je korišćenje SNMP verzije 2.

Svakako, lakši segment konfiguracije je SNMP traps u okviru verzije 3 na koji se fokusiramo ispod.
```
config system snmp sysinfo
	set status enable
	set description "<IME-UREĐAJA>"
	set contact-info "<MEJL-ADMINISTRATORA>"
	set location "<IME-LOKACIJE>"
config system snmp user
    edit "<KORISNIČKO-IME>"
        set notify-hosts <IP-SNMP-SERVERA>
        set security-level auth-priv
        set auth-proto sha256
        set auth-pwd xxxx
        set priv-proto aes256
        set priv-pwd xxxx
    next
end
```

U slučaju da se koristi samo SNMP verzija 3, preporučuje se brisanje svih community-ja.
```
config system community
	purge
end
```

Preporuka je da se smanji limit prijave prilikom zauzeća memorije na uređajima.
```
config system sysinfo
	set trap-free-memory-threshold 25
	set trap-freeable-memory-threshold 50
end
```

Kada su FortiGate u HA, moguće je da uređaji šalju SNMP trap pakete kroz ```ha-mgmt-interface```. To se ne preporučuje, zbog problema sa local-out saobraćajem FortiGate-a.


### FortiGate HA menadžment interfejs
Umesto ha-mgmt-interface komande, preporučuje se korišćenje seta komandi na menadžment interfejsu.
```
config system interface
	edit <IME-INTERFEJSA>
		set dedicated-to management
		set management-ip <MENADŽMENT-IP/MASK>
	next
end
```

Komanda ```dedicated-to``` dedicira menadžment interfejs, posle čega se on ne može referencirati u pravilima.

[Technical Tip: FortiGate dedicated-mgmt feature, or Out-of-band Management](https://community.fortinet.com/t5/FortiGate/Technical-Tip-FortiGate-dedicated-mgmt-feature-or-Out-of-band/ta-p/193699)

Komandom ```management-ip``` definišemo jedinstvenu IP adresu za obe jedinice, koja se može koristiti i na menadžment, i na servisnom interfejsu(ne preporučuje se).

[Technical Tip: Implement independent Management IP for HA Cluster](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Implement-independent-Management-IP-for-HA-Cluster/ta-p/224671)





## Konfiguracija High-Availability(HA)
Konfiguracija HA je u većini implementacija ista, ili slična, i postoje određene preporuke koje se retko primenjuju, a značajni su za rad klastera.

Obradićemo jedino rešenje koje ima smisla u implementaciji FortiGate HA, a to je **FortiGate Clustering Protocol(FGCP) Active-Passive(A-P)** mod rada.


### Inicijalna konfiguracija HA
Inicijalna konfiguracija za rad klastera može uvek biti ista.

Konfiguracija primarnog HA uređaja:
```
config system ha
	set group-id <HA-ID>
	set group-name <HA-IME>
	set mode a-p
	set password <HA-ŠIFRA>
	set hbdev <HA-HB-IME-INTERFEJSA1> 150 <HA-HB-IME-INTERFEJSA2> 100
	set override disable
	set monitor <IME-INTERFEJSA>
	set priority 150
end
```

Konfiguracija sekundarnog HA uređaja:
```
config system ha
	set group-id <HA-ID>
	set group-name <HA-IME>
	set mode a-p
	set password <HA-ŠIFRA>
	set hbdev <HA-HB-IME-INTERFEJSA1> 150 <HA-HB-IME-INTERFEJSA2> 100
	set override disable
	set priority 100
end
```

Preporučuje se da se **group ID definiše eksplicitno**. Kada je group ID isti za klaster u istoj mreži, može doći do problema u dupliranim MAC address tabelama na svičevima preko kojih su vezani, čime izazivamo prekide produkcije.

Preporučuje se da su **svi produkcioni interfejsi** monitor interfejsi.

Preporučuje se da je prioritet primarnog uređaja veći od 128, što je podrazumevana vrednost prioriteta na FortiGate uređaju.

Preporučena konfiguracija klastera je da se override opcija onemogući, gde se kontrola vrši pomoću uptime-a, gde ne želimo da prekidom uređaja dođe do duplog failover-a nakon što se povrati stanje primarnog uređaja.


### Replikacija sesija
Podrazumevana vrednost ne uključuje repliciranje sesija na sekundarni uređaj. Preporučuje se repliciranje svih TCP, UDP, SCTP i ICMP sesija.
```
config system ha
    set session-pickup enable
    set session-pickup-connectionless enable
    set session-pickup-expectation enable
end
```

[Technical Tip: HA session failover (session pickup) ](https://community.fortinet.com/t5/FortiGate/Technical-Tip-HA-session-failover-session-pickup/ta-p/191165)

### Failover kriterijumi
Podrazumevani parametri failover-a su:

- Pad heartbeat(HB) linka -- Podrazumevano podešavanje

- Pad napajanja primarnog uređaja -- Podrazumevano podešavanje

- Prestanak rada SSD diska(opciono)
	Kako bi se desio failover u klasteru nakon prestanka rada SSD diska, potrebno je upaliti monitoring diska u HA procesu.
	``` 
	config system ha
		set ssd-failover enable
	end
	```

- Visoka iskorišćenost memorije uređaja(opciono)
	Kako bi se desio failover u klasteru nakon previsoke memorije uređaja, potrebno je upaliti monitoring memorije u HA procesu. Preporuka je da se i kod manjih uređaja poveća limit sa conserve mod, dokle god je preporučena verzija za uređaje 7.4.x.
	```
	config system ha
		set memory-based-failover enable
		set memory-failover-threshold 92
		set memory-failover-flip-timeout 60
	end
	config system global
		set memory-use-threshold-red 94
		set memory-use-threshold-green 90
		set memory-use-threshold-extreme 97
	end
	```

[Technical Tip: FortiGate HA failover due to memory utilization](https://community.fortinet.com/t5/FortiGate/Technical-Tip-FortiGate-HA-failover-due-to-memory-utilization/ta-p/195019)

- Pad interfejsa(opciono)
	U slučaju pada produkcionih interfejsa na primarnoj jedinici, preporučuje se odrađivanje failover-a na sekundarni uređaj, u slučaju da je na tom uređaju interfejs dostupan.
	```
	config system ha
		set monitor <IME-INTERFEJSA1> <IME-INTERFEJSA2>
	end
	```

	Monitor interfejs može biti i fizički interfejs u agregaciji, pored toga se može i definisati minimalni broj monitoring interfejsa nakon čega dolazi do failover-a.

- Monitor server(opciono)
	Kada monitoring interfejsa nije dovoljan, potrebno je testirati konekciju sa udaljenom IP adresom pomoću FortiGate link-monitor procesa. Potrebno je ugasiti opcije link monitora koje utiču na rutiranje.
	``` 
	config system link-monitor
		edit "<IME-LINK-MONITORA>"  
			set srcintf <IME-IZLAZNOG-INTERFEJSA>  
			set server <IP-ADRESA-SERVERA1> <IP-ADRESA-SERVERA1>
			set protocol <PORT-SERVERA>  
			set ha-priority <PRIORITET-LINKA>  
			set update-cascade-interface disable
			set update-static-route disable
			set update-policy-route disable
		next
	end
	config system ha
		set pingserver-monitor-interface <IME-IZLAZNOG-INTERFEJSA>  
		set pingserver-failover-threshold <FAILOVER-PRIORITET>  
		set pingserver-flip-timeout <VREME-FAILOVER>  
		set pingserver-secondary-force-reset disable
	end
	```

	Podrazumevana vrednost za protocol je 1(ICMP).
	
	Podrazumevano podešavanje za ```ping-server-flip-timeout``` je 0, failover se dešava kada se izgubi konekcija sa jednim monitor serverom. Uz pomoć ```ha-priority``` i ```ping-server-flip-timeout``` možemo kontrolisati razlog failover-a.

[Technical Tip: Combining remote link monitoring with a high availability FGCP cluster](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Combining-remote-link-monitoring-with-a-high/ta-p/191330)


### Failover opcije
U velikim okruženjima gde FortiGate razmenjuje velike količine ruta kroz dinamičke ruting protokole, može doći do loše replikacije ruta na sekundarni uređaj. U tom slučaju se preporučuje modifikovanje route parametara u okviru HA podešavanja.
```
config system ha
	set route-hold 30
	set route-wait 30
	set route-ttl 0
end
```

[Technical Tip: Controlling how HA synchronizes routing table updates](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Controlling-how-HA-synchronizes-routing-table/ta-p/191310)

Sa podrazumevanom konfiguracijom, tokom upgrade-a uređaja dolazi do 2 failover-a. U slučaju da je potreban što veći nivo timeout-a, ili se zahteva provera servisa na prvoj upgrade-ovanoj jedinici, moguće je konfigurisati da se failover sa sekundarne na primarnu jedinicu ne dogodi automatski. Sam upgrade proces se kontroliše komandom ```ha-uptime-diff-margin``` koja stopira failover loop proces prilikom reboot-a uređaja(ili restarta uptime-a). Podrazumevana vrednost je 15 minuta.
```
config system ha
	set ha-uptime-diff-margin 60
end
```

[Technical Tip: HA age time difference (HA cluster uptime)](https://community.fortinet.com/t5/FortiGate/Technical-Tip-HA-age-time-difference-HA-cluster-uptime/ta-p/230805)

Neki svičevi ignorišu Gratuitious ARP(GARP) pakete i ne promene ulaz u ARP tabeli prilikom failover-a. U tom slučaju se može uključiti opcija sa kojom bi uređaj prilikom failover-a odradio bounce interfejsa.
```
config system ha
	set linked-failed-signal enable
end
```


### Konfiguracija VDOM particija
U slučaju da je potrebno kreirati klaster gde je za jedan VDOM primarni jedan firewall, za drugi VDOM drugi firewall, koristi se VDOM partitioning.

Najčešći slučaj je podela VDOM-ova po lokaciji, gde u okviru dva datacentra postoji jedan FGCP klaster.
```
config system ha
	set vklasterstatus enable
	config vklaster
        edit 1
            set override enable
            set priority 200
            set vdom "<IME-VDOM1>" "<IME-VDOM2>"
        next
        edit 2
            set override enable
            set priority 100
            set vdom "<IME-VDOM3>" "<IME-VDOM4>"
        next
    end
end
```

Kod geografski razdvojenih uređaja, preporučuje se i modifikacija HB intervala.
```
config system ha
	set hb-interval 5
end
```

[Technical Tip: Configuring HA virtual cluster with VDOM Partitioning](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Configuring-HA-virtual-cluster-with-VDOM/ta-p/268820)





## Podešavanja interfejsa


### Blokiranje intra-zone saobraćaja
FortiGate ne mora biti, ali bi ga trebalo konfigurisati kao zone-based firewall. Kada se definišu zone, potrebno je konfigurisati intra-zone blokiranje pravila, gde bi u okviru firewall polise definisali propuštanja po potrebi.
```
config system zone
    edit <IME-ZONE> 
        set intrazone deny
    next
end
```

[Technical Tip: Block or allow intra-zone traffic](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Block-or-allow-intra-zone-traffic/ta-p/279733)


### Gašenje nekorišćenih interfejsa
Podrazumevano podešavanje interfejsa je da su upaljeni na većini manjih uređaja. Potrebno ih je ugasiti(i eventualno izbaciti iz hardverskog sviča).
```
config system interface
    edit <IME-INTERFEJSA>
		unset ip
        set status down
    next
end
```


### Brisanje nekorišćenih DHCP servera
Većina manjih uređaja dolaze sa unapred konfigurisanim DHCP serverom za FortiLink Subinterfejse(i eventualno hardverski svič). S obzirom da nije preporuka koristiti njih, potrebno ih je izbrisati.
```
config system dhcp server
	delete <ID-DHCP-SERVERA>
end
```


### Gašenje menadžment servisa na svim interfejsima koji nisu za menadžment
Podrazumevana konfiguracija sadrži veliki broj interfejsa na kojima su upaljeni menadžment servisi. Potrebno je ugasiti svaku od njih na mestima na kojima se ne koristi, ili ne treba da se koristi.
``` 
config system interface
	edit <IME-INTERFEJSA>
		unset allowaccess
	next
end
```


### Definisanje protoka na WAN interfejsima
ISP linkovi skoro uvek imaju niži protok od same brzine linka. Definisanjem brzine linka imamo tri benefita: dozvoljava statistiku na WAN linku preko FortiAnalyzer-a, potreban za rad nekih od SD-WAN modova i omogućava SD-WAN analitiku na FortiAnalyzer-u. 
```
config system interface
    edit "<IME-INTERFEJSA>"  
        set monitor-bandwidth enable
    next
end
```


### Konfiguracija detekcije uređaja
FortiGate ima opciju prikupljanja informacija o krajnjim uređajima tako što sluša saobraćaj na LAN linkovima i obrađuje ga u jednom preglednom i značajnom prikazu.
```
config system interface
    edit "<IME-INTERFEJSA>"  
        set device-identification enable
    next
end
```

Treba napomenuti da ova opcija na interfejsima sa većim opsezima može povećati opterećenje uređaja.

[Technical Tip: Enable 'Device Detection' to allow FortiOS to monitor networks](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Enable-Device-Detection-to-allow-FortiOS-to/ta-p/190901)





## Administratorski pristup


### Konfiguracija password polise
Od verzije 7.6.5 podrazumevano pravilo za šifre postavljeno je na minimum 12 karaktera, dok je na starijim verzijama potrebno definisati ponašanje kroz password polisu.

U slučaju definisanja password polise, potrebno je promeniti password prilikom sledećeg pristupa uređaju.
``` 
config system password-policy
    set status enable
    set minimum-length 15
    set min-lower-case-letter 1
    set min-upper-case-letter 1
    set min-non-alphanumeric 1
    set min-number 1
    set reuse-password disable
end
```


### Konfiguracija administratora
Zaprepašćujući broj korisnika ne briše podrazumevanog korisnika koji se koristi tokom inicijalizacije uređaja. Potrebno je u svakoj implementaciji izbrisati podrazumevanog korisnika, kao i definisati IP adrese preko kojih administratori mogu pristupiti uređaju.
```
config system admin
	delete admin
    edit "<IME-ADMINISTRATORA>"
        set trusthost1 <MENADŽMENT1-IP/MASK>
		set trusthost2 <MENADŽMENT2-IP/MASK>
		set trusthost3 <MENADŽMENT3-IP/MASK>
        set accprofile "<IME-ADMINISTRATORSKOG-PROFILA>"
        set vdom "root"
        set password <ŠIFRA-ADMINISTRATORA>
    next
end
```


### Konfiguracija Multi-Factor Authentication(MFA) za administratora
Za administratorski pristup se preporučuje implementacija MFA. Fortinet uz uređaj dostavlja dva FortiToken-a besplatno, što se može iskoristiti. Pored toga, podržava se SAML SSO opcije u vidu Microsoft Entra ID, Cisco Duo, Okta itd...
```
config system admin
	edit "<IME-ADMINISTRATORA>"
		set two-factor fortitoken
        set fortitoken "<TOKEN-ID>"
        set email-to "<IMEJL-ADRESA>"
	next
end
```

Primer za SAML SSO sa Microsoft Entra ID možete naći ispod:
[Technical Tip: Configuring SAML SSO login for FortiGate administrators with Entra ID acting as SAML IdP](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Configuring-SAML-SSO-login-for-FortiGate/ta-p/194656)


### Konfiguracija break-glass administratora
Preporuka je da za svaki korisnički nalog postoji MFA konfiguracija. U tom slučaju je dobra praksa imati jedan rezervni nalog u slučaju pada MFA servisa. 

Taj nalog se konfiguriše sa pristupom sa određenih IP adresa(ili samo konzolno) i sa kompleksnom šifrom sa visokim brojem karaktera(32 ili više). Ispod možete naći primer konfiguracije administratora za break-glass pristup samo preko konzole.
```
config system admin
	edit "<IME-ADMINISTRATORA>"
        set vdom "root"
        set trusthost1 0.0.0.0/32
        set accprofile "super_admin"
        set password <ŠIFRA-ADMINISTRATORA>
	next
end
```


### Konfiguracija administratorskog profila
Preporučuje se kreiranje novih administratorskih profila sa nivoem pristupa po tipskoj potrebi administratora. 

Jedan primer pravilnog limitiranja pristupa postoji u našem CT Cloud-u, gde se za određenog korisnika dopušta pristup CyberStellar alatu koji dinamički upisuje blacklist-ovane IP adrese kroz adresne objekte u postojeću adresnu grupu koja se koristi u firewall pravilu.
```
config system accprofile
    edit "<IME-ADMINISTRATORSKOG-PROFILA>"
        set fwgrp custom
        config fwgrp-permission
            set address read-write
        end
    next
end
```


### Modifikovanje podrazumevanih menadžment portova
Napadači često traže vektor napada po predefinisanim setovima poznatih portova. Preporučuje se promena portova za menadžment pristup uređaju.
```
config system global
    set admin-port <HTTP-PORT>
    set admin-sport <HTTPS-PORT>
    set admin-ssh-port <SSH-PORT>
    set admin-telnet-port <TELNET-PORT>
end
```

Telnet port ne mora da se menja, s obzirom da se u svakoj implementaciji preporučuje gašenje Telnet servisa na FortiGate uređaju.
```
config system global
    set admin-telnet disable
end
```


### Povećavanje timeout-a za administratorski pristup
Podrazumevana vrednost je 60 sekundi. Preporučuje se povećanje vremena zaključanja na bar 5 minuta(300 sekundi)
``` 
config system global
    set admin-lockout-duration 600
end
```

### Promena timeout-a za idle stanje administratora
Podrazumevana vrednost je 5 minuta. Ne preporučuje se značajno povećanje, obično to definišemo na 15 minuta.
``` 
config system global
    set admintimeout 15
end
```


### Pre-login banner
Pre-login banner se prikazuje prilikom Web pristupa pre prikazivanja stranice za logovanje. Nije prikazan prilikom CLI pristupa.
```
config system global
	set pre-login-banner enable
end
```

U segmentu **Replacement Messages**->**Pre-login Disclaimer Message** može se dodatno konfigurisati pre-login banner.


### Post-login banner
Post-login banner se prikazuje prilikom Web i CLI pristupa nakon logovanja na uređaj. S obzirom da se prikazuje prilikom svakog načina pristupa, preporučuje se njegovo korišćenje.
```
config system global
	set post-login-banner enable
end
```

U segmentu **Replacement Messages**->**Post-login Disclaimer Message** može se dodatno konfigurisati pre-login banner.


### Gašenje USB auto install opcije
Preporučeno je ugasiti opciju automatskog instaliranja verzije i konfiguracije na firewall uređaju pomoću USB interfejsa. Time štitimo našu mrežnu infrastrukturu od lica koji imaju fizički pristup opremi.

U slučajevima kada izvršavamo ZTP, odnosno LTP, implementaciju uređaja pomoću USB-a se preporučuje podrazumevano podešavanje dok se ZTP proces ne završi.
```
config system auto-install
	set auto-install-config disable
	set auto-install-image disable
end
```

[Technical Tip: Automatic installation of Firmware and system configuration](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Automatic-installation-of-Firmware-and-system/ta-p/197938)


### Gašenje FortiCloud SSO pristupa
Preporučeno je gašenje FortiCloud SSO pristupa zbog velikog broja verzija koje imaju slabost kroz ovaj tip pristupa.
``` 
config system global
    set admin-forticloud-sso-login disable
end
```

[Technical Tip: Understanding the FortiOS critical vulnerability (FG-IR-25-647, FG-IR-26-060) upgrade prompt in GUI](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Understanding-the-FortiOS-critical-vulnerability/ta-p/426944)





## Logovanje i performanse uređaja


### Kreiranje revizije nakon logout
Preporučuje se paljenje opcije kreiranja revizije na FortiGate uređaju lokalno, ako ne postoji FortiManager ili lokalni repozitorijum sa automatski bekap uređaja. 
``` 
config system global
    set revision-backup-on-logout enable
end
```


### Uključivanje korišćenja CDN-a
Preporučena konfiguracija za korišćenje CDN-a za ubrzavanje GUI odziva. Podrazumevana konfiguracija je da je opcija upaljena, ali treba znati za nju.
``` 
config system global
    set gui-cdn-usage enable
end
```

[Loading artifacts from a CDN for improved GUI performance](https://docs.fortinet.com/document/fortigate/7.0.0/new-features/205105/loading-artifacts-from-a-cdn-for-improved-gui-performance-7-0-4)


### Uključivanje korišćenja lokalnog ISDB keša
FortiGate lokalno čuva ISDB ulaze za veliki broj servisa. Korišćenje lokalnog keša pomaže u smanjivanju opterećenja manjih uređaja kod povlačenja razlike ulaza.
``` 
config system settings
    set internet-service-database-cache enable
end
```


### Uključivanje automatske provere diska
Kada FortiGate uređaj ima SSD, prilikom neočekivanog reboot-a, potrebno je da se odradi manuelna provera diska za šta je potreban reboot. Automatskom proverom diska izbegavamo dodatni reboot.
``` 
config system global
    set autorun-log-fsck enable
end
```


### Logovanje CLI komandi
Podrazumevano, FortiGate ne loguje CLI komande.
``` 
config system global
    set cli-audit-log enable
end
```

[Technical Tip: Enable audit log via CLI](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Enable-audit-log-via-CLI/ta-p/266822)


### Proširenje logovanja i prikaza logova
Podrazumevana podešavanja ne loguju implicit deny pravila, local-in i local-out saobraćaj, dodatno logovanje, razrešavanje IP adresa i portova, API akcije i mapiranje imena zona.
``` 
config log setting
    set fwpolicy-implicit-log enable
    set local-in-allow enable
    set local-in-deny-unicast enable
    set local-in-deny-broadcast enable
    set local-out enable
    set extended-log enable
    set extended-utm-log enable
    set resolve-ip enable
    set resolve-port enable
    set rest-api-set enable
    set rest-api-get enable
    set zone-name enable
end
config log memory filter
    set local-traffic enable
end
```


### Uključivanje logovanja na disk
FortiGate sa diskom bi trebalo da loguje saobraćaj na disk, sa time da kada se popuni disk, brisanje kreće od najstarijih ka novijim logovima.
``` 
config log disk setting
    set status enable
    set maximum-log-age 0
end
```

[Technical Tip: How to configure logging to disk on the FortiGate using the GUI or the CLI](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-configure-logging-to-disk-on-the-FortiGate/ta-p/216995)





## Firewall polise
Firewall polise obuhvataju dva seta polisa:
 - Local-in polisa
 - Security polisa


### Geografski objekti
Geografski objekti su predefinisane liste IP adresa koje su u vlasništvu internet provajdera u određenim državama. Mogu se koristiti u različitim polisama, između ostalog u local-in polisi. U budućnosti ćemo pomenuti Threat feed-ove koji sadrže liste IP adresa provajdera definisani kroz njihov jedinstveni AS.
```
config firewall address
	edit <IME-GEO-OBJEKTA1>
		set type geography
		set country "<ŠIFRA-DRŽAVE1>"
	next
	edit <IME-GEO-OBJEKTA2>
		set type geography
		set country "<ŠIFRA-DRŽAVE2>"
	next
end
```

### ISDB objekti
ISDB objekti su lista internet servisa koji sadrže mapiranje destinaconih IP-adresa:Port.

Preporuka je kreiranje tri tipa ISDB grupa:
 - Grupa sa malicioznim internet servisima
 - Grupa sa vendorskim internet skenerima
 - Grupa sa opsezima vendora za hosting servise
```
config firewall internet-service-group
	edit <IME-ISDB-GRUPE1>
		set direction source
		set member "VPN-Anonymous.VPN" "Tor-Tor.Node" "Tor-Relay.Node" "Tor-Exit.Node" "Spam-Spamming.Server" "Proxy-Proxy.Server" "Phishing-Phishing.Server" "Malicious-Malicious.Server" "Hosting-Bulletproof.Hosting" "Botnet-C&C.Server"
	next
	edit <IME-ISDB-GRUPE2>
		set direction source
		set member "Censys-Scanner" "Stretchoid-Scanner" "InterneTTL-Scanner" "Shodan-Scanner" "Tenable-Tenable.io.Cloud.Scanner" "NetScout-Scanner" "Recyber-Scanner" "Cyber.Casa-Scanner" "BinaryEdge-Scanner" "UK.NCSC-Scanner" "CriminalIP-Scanner" "Internet.Census.Group-Scanner" "Shadowserver-Scanner" "LeakIX-Scanner" "Hadrian-Scanner" "Rapid7-Scanner" "ONYPHE-Scanner" "Modat-Scanner" "Palo.Alto.Networks-Cortex.Xpanse.Scanner"
	next
	edit <IME-ISDB-GRUPE3>
		set direction source
		set member "Hosting-Bulletproof.Hosting" "ColoCrossing-ColoCrossing.Hosting.Service" "THE.Hosting-THE.Hosting.Hosting.Service" "SERVERD-SERVERD.Hosting.Service" "EGI-EGI.Hosting.Service" "M247-M247.Hosting.Service" "Quintex-Quintex.Hosting.Service" "Aeza-Aeza.Hosting.Service" "Amanah-Amanah.Hosting.Service" "Cloudzy-Cloudzy.Hosting.Service" "3xK-3xK.Hosting.Service"
	next
end
```
 
Treba napomenuti da se ovi objekti koriste u produkcionim okruženjima na različitim projektima.

[Policy and Objects: Internet Services](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/849970/internet-services)


### Eksterni konektori
FortiGate podržava različite tipove eksternih konektora kroz threat feed-ove. Koncentrisaćemo se na one koje se koriste u različitim segmentima Local-in i Security polisa.
 - FortiGuard Category Threat Feed -- Koristi se za povlačenje eksternih lista URL-ova koji se referenciraju u Web filtering-u.
 - IP Address Threat Feed -- Koristi se za povlačenje eksternih lista IP adresa koji se referenciraju u Local-in i Security polisama kao source i destinacioni objekti.
 - Domain Name Threat Feed -- Koristi se za povlačenje eksternih lista domenskih imena koji se referenciraju u DNS filtering-u.
 - MAC Address Threat Feed -- Koristi se za povlačenje eksternih lista MAC adresa koji se referenciraju u Security polisi, između ostalog.
 - Malware Hash Threat Feed -- Koristi se za povlačenje eksternih lista Hash-eva koji se referenciraju u Antivirus profilima.

[Fortinet Security Fabric: External feeds](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/9463/external-feeds)


### Local-in polisa
Local-in polisa sadrži pravila koja dozvoljavaju pristup koji se terminira(počinje i završava) na samim interfejsima FortiGate uređaja.

Kada se selektuje menadžment servis na samom interfejsu, ili se upali funkcionalnost, automatski se kreira Local-in pravilo.
```
config firewall local-in-policy
	edit 0
		set intf <IME-WAN-INTERFEJSA>
		set dstaddr all
		set internet-service-src enable
		set internet-service-src-group <IME-ISDB-GRUPE1> <IME-ISDB-GRUPE2> <IME-ISDB-GRUPE3>
		set action deny
	next
	edit 0
		set intf <IME-WAN-INTERFEJSA>
		set dstaddr all
		set srcaddr-negate enable
		set srcaddr <IME-GEO-OBJEKTA1> <IME-GEO-OBJEKTA2>
		set action deny
		set service <IME-SERVISNE-GRUPE>
	next
end
```

[Technical Tip: Creating a Local-In policy (IPv4 and IPv6) on GUI](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Creating-a-Local-In-policy-IPv4-and-IPv6-on-GUI/ta-p/326547)


### Virtual patching na Local-in polisi
Konfiguracija local-in polise obuhvata segment zaštite uređaja na L4 nivou. Od verzije 7.4.1, uveden je koncept virtual patching-a, gde uređaj koristi svoju IPS bazu kako bi blokirao pokušaje napada poznatih slabosti FortiGate uređaja. Virtual patching se između ostalog može koristiti i u local-in pravilima.

[Local-in Policies: Virtual patching on the local-in management interface](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/393161/virtual-patching-on-the-local-in-management-interface)





### Security polisa
Konfiguracija security polise je isrcpan proces sa velikim brojem opcija koje administrator ima, kako bi što preciznije definisao tip saobraćaja koji hoće da propusti. U ovom delu teksta se koncentrišemo na baseline pravila koja se preporučuju prilikom inicijalnog definisanja pravila.



## Security profili