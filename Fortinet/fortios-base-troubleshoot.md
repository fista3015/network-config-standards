# FortiGate baseline konfiguracija prilikom inicijalizacije uređaja

***[DRAFT VERZIJA!!!]***

Baseline konfiguracija FortiGate firewall-a prilikom inicijalizacije uređaja pre implementacije u produkciono okruženje.

- [Početak dokumenta](#fortigate-baseline-konfiguracija-prilikom-inicijalizacije-uređaja)
	- [Sistemska podešavanja - Troubleshooting](#sistemska-podešavanja---troubleshooting)
		- [Upgrade firewall uređaja](#upgrade-firewall-uređaja)
		- [Konfiguracija DNS servera](#konfiguracija-dns-servera)
		- [Konfiguracija NTP servera](#konfiguracija-ntp-servera)
		- [Konfiguracija SNMP servera](#konfiguracija-snmp-servera)
	- [Konfiguracija High-Availability(HA) - Troubleshooting](#konfiguracija-high-availabilityha---troubleshooting)
		- [Inicijalna konfiguracija HA](#inicijalna-konfiguracija-ha)
		- [Pregled logova u slučaju neočekivanog failover-a](#pregled-logova-u-slučaju-neočekivanog-failover-a)
		- [Replikacija sesija](#replikacija-sesija)
		- [Failover kriterijumi](#failover-kriterijumi)
		- [Failover opcije](#failover-opcije)
		- [Konfiguracija VDOM particija](#konfiguracija-vdom-particija)
	- [Podešavanja interfejsa](#podešavanja-interfejsa)
		- [Blokiranje intra-zone saobraćaja](#blokiranje-intra-zone-saobraćaja)
		- [Isključivanje nekorišćenih interfejsa](#isključivanje-nekorišćenih-interfejsa)
		- [Brisanje nekorišćenih DHCP servera](#brisanje-nekorišćenih-dhcp-servera)
		- [Isključivanje menadžment servisa na svim interfejsima koji nisu za menadžment](#isključivanje-menadžment-servisa-na-svim-interfejsima-koji-nisu-za-menadžment)
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
		- [Isključivanje USB auto install opcije](#isključivanje-usb-auto-install-opcije)
		- [Isključivanje FortiCloud SSO pristupa](#isključivanje-forticloud-sso-pristupa)
	- [Logovanje i performanse uređaja](#logovanje-i-performanse-uređaja)
		- [Kreiranje revizije nakon logout](#kreiranje-revizije-nakon-logout)
		- [Uključivanje korišćenja CDN-a](#uključivanje-korišćenja-cdn-a)
		- [Uključivanje korišćenja lokalnog ISDB keša](#uključivanje-korišćenja-lokalnog-isdb-keša)
		- [Uključivanje automatske provere diska](#uključivanje-automatske-provere-diska)
		- [Logovanje CLI komandi](#logovanje-cli-komandi)
		- [Proširenje logovanja i prikaza logova](#proširenje-logovanja-i-prikaza-logova)
		- [Uključivanje logovanja na disk](#uključivanje-logovanja-na-disk)
	- [Firewall polise](#firewall-polise)
		- [Geografski objekti](#geografski-objekti)
		- [ISDB objekti](#isdb-objekti)
		- [Eksterni konektori](#eksterni-konektori)
		- [Local-in polisa](#local-in-polisa)
		- [Virtual Patching](#virtual-patching)
		- [Security polisa](#security-polisa)
	- [Security profili](#security-profili)


## Sistemska podešavanja - Troubleshooting


### Upgrade firewall uređaja
Po preporuci Fortineta, upgrade se vrši uz konzolni pristup kako bi se pratio svaki ispis uređaja. Često to nije moguće u manjim okruženjima ili udaljenim lokacijama. Nakon upgrade-a, dobra je praksa pregledati najznačajnije delove sistema:

- Pregled logova za promene konfiguracije
```
diagnose debug config-error-log read
```

- Pregled logova za neočekivani pad servisa
```
diagnose debug crashlog read
```

- Pregled performansi i stanja sistema
```
get system status
get system performance status
execute sensor list
```

- Pregled FGCP HA dostupnosti
```
get system ha status
diagnose sys ha history read
diagnose sys ha dump-by group
```

- Pregled stanja interfejsa
```
diagnose hardware deviceinfo nic <IME-INTERFEJSA>
get system interface transceiver <IME-INTERFEJSA>
diagnose ip address list
```

- Pregled tabele rutiranja
```
get router info routing-table details
get router info routing-table database
get router info kernel
get router info bgp summary
get router info bgp network
get router info ospf route
get router info ospf database brief
diagnose firewall proute list
diagnose sys sdwan service4
```

- Pregled statusa IPSec tunela
```
diagnose vpn ike gateway list
diagnose vpn tunnel list
```


### Konfiguracija DNS servera
U slučaju da postoji problem sa komunikacijom ka DNS serveru, potrebno je proveriti DNS podešavanja, dostupnost komunikacije sa Source IP adrese i logove. Komande koje prikazuju DNS podešavanja, kao i razrešene DNS zapise u kešu uređaja.
```
diagnose test application dnsproxy 3
diagnose test application dnsproxy 5
```

U najgorem slučaju je potrebno izvršiti debug, i koriste se sledeće komande:
```
diagnose debug disable
diagnose debug reset
diagnose debug application dnsproxy 255
diagnose debug enable
```


### Konfiguracija NTP servera
U slučaju da postoji problem sa komunikacijom ka NTP serveru, izvršava se provera dostupnosti i sinhronizacije vremena
```
diagnose sys ntp status
```

U najgorem slučaju je potrebno izvršiti debug, i koriste se sledeće komande:
```
diagnose debug disable
diagnose debug reset
diagnose debug application ntpd -1
diagnose debug enable
```


### Konfiguracija SNMP servera
U slučaju da postoji problem sa komunikacijom sa SNMP serverom, izvršava se provera poslatih poruka i njihovog statusa. Moguće je i generisati test trap poruku.
```
diagnose test application snmpd 2
diagnose test application snmpd 4
```


U najgorem slučaju je potrebno izvršiti debug, i koriste se sledeće komande:
```
diagnose debug disable
diagnose debug reset
diagnose debug application snmpd -1
diagnose debug enable
```





## Konfiguracija High-Availability(HA) - Troubleshooting
Postoje različiti problemi koji mogu nastati prilikom konfiguracije FGCP HA ili zbog određenih promena na sistemu.


### Uspostava HA
Najosnovniji problem može nastati prilikom uspostave FGCP HA cluster-a.

U tom slučaju je potrebno proveriti konfiguraciju na oba uređaju, prilikom čega se moraju poklapati osnovni parametri kao što su **Group ID**, **Group name**, **HA mod**(A-P, A-A), **Password** i **Hearbeat interfejsi**.

Pored poklapanja interfejsa, potrebno je definisati prioritet svakog od uređaja, kao i način odlučivanja koji je uređaj aktivan(primaran).

U slučaju da se pod menijem ne prikazuje drugi FortiGate uređaj, potrebno je izvršiti proveru komunikacije pomoću komandi za prikaz statusa.
```
get system ha status
```

Ako se na prikazu ne vidi status drugog uređaja, potrebno je izvršiti debug HA poruka.
```
diagnose debug disable
diagnose debug reset
diagnose debug application hatalk -1
diagnose debug application hasync -1
diagnose debug enable
```


### Sinhronizacija HA članova
Nakon što se uređaji povežu, potrebno je da se automatski sinhronizuju. U starijim verzijama, kao i u retkim slučajevima, proces se ne izvršava automatski ili se izvršava jako sporo.

Kada je to slučaj, prvo se proverava segment sinhronizacije pomoću komande za prikaz statusa.
```
get system ha status
```

Detaljniji pregled je moguć prikazom koji naznačava tačan segment konfiguracije koji nije sinhronizovan, kao i VDOM u multi-VDOM okruženjima.
```
diagnose sys ha checksum test
diagnose sys ha checksum show
```

U slučaju da je potrebno izolovati segment konfiguracije, to je moguće uraditi grep komandom. Za slučaj da je u pitanju firewall sa jednim VDOM-om, u komandi se koristi root.
```
diagnose sys ha checksum show root | grep system
diagnose sys ha checksum show root | grep firewall
diagnose sys ha checksum show root | grep router
```

Nekada je sinhronizacija sporija, čime se neko vreme ne izvršava rekalkulacija checksum-a, pa iako su uređaji sinhronizovani, to nije prikazano u sistemu. Kako bi ubrzali proces rekalkulacije checksum-a konfiguracije, moguće je primeniti zasebnu komandu.
```
diagnose sys ha checksum recalculate
```

Detaljniji koraci u slučaju da navedeni prikazi ne pomognu prilikom sinhronizacije ili je uređaj afektiran nekim od poznatih problema, moguće je pratiti zasebno uputstvo za ručnu sinhronizaciju uređaja u HA.
[Troubleshooting Tip: How to troubleshoot HA synchronization issue using GUI and CLI on FortiGate/FortiProxy](https://community.fortinet.com/fortigate-3/troubleshooting-tip-how-to-troubleshoot-ha-synchronization-issue-using-gui-and-cli-on-fortigate-fortiproxy-95628)


### Pregled logova u slučaju neočekivanog failover-a
U slučaju neočekivanog failover-a, potrebno je proveriti status cluster-a, da li su svi uređaji konektovani sa primarnom jedinicom i da li su sinhronizovani. Pored toga se mogu videti i greške na heartbeat interfejsima pomoću komande za prikaz statusa.
```
get system ha status
```

U slučaju da je potrebno prikazati veći broj prošlih događaja cluster-a, to je moguće pomoću komande ispod.
```
diag sys ha history read
```

Za najbrži prikaz stanja cluster-a i njegove istorije, moguće je primeniti komandu ispod.
```
diag sys ha dump-by group
``` 

Komanda iznad daje sledeće značajne informacije:
```
linkfails = <BROJ-OBORENIH-INTERFEJSA>
chg_time = <2(primarni) ili 3(sekundarni)>(work)
mondev: <IME-INTERFEJSA>(...output omitted, status = <1 radi ili 0 ne radi>)
'SN uređaja': ...output omitted, link_failure=<BROJ-PADA-INTERFEJSA>, ...output omitted, uptime/reset_cnt=<UPTIME-UREĐAJA>/<BROJ-PADA-MONITOR-INTERFEJSA>
```

Brzi prikaz MAC adresa cluster-a se može naći na primarnoj jedinici komandom ispod.
```
diag sys ha mac
```


### Replikacija sesija
Podrazumevana vrednost ne uključuje replikaciju sesija na sekundarni uređaj. Preporučuje se repliciranje svih TCP, UDP, SCTP i ICMP sesija.
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
	Kako bi se desio failover u klasteru nakon visoke iskorišćenosti memorije uređaja, potrebno je upaliti monitoring memorije u HA procesu. Preporuka je da se i kod manjih uređaja poveća limit sa conserve mod, dokle god je preporučena verzija za uređaje 7.4.x.
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
	Kada monitoring interfejsa nije dovoljan, potrebno je testirati dostupnost sa udaljenom IP adresom pomoću FortiGate link-monitor procesa. Potrebno je ugasiti opcije link monitora koje utiču na rutiranje.
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

	Podrazumevana vrednost za protokol je 1(ICMP).
	
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
U slučaju da je potrebno kreirati klaster gde je za jedan VDOM primarni jedan uređaj, za drugi VDOM drugi uređaj, koristi se VDOM partitioning.

Najčešći slučaj je podela VDOM-ova po lokaciji, gde u okviru dva datacentra postoji jedan FGCP klaster.
```
config system ha
	set vcluster-status enable
	config vcluster
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


### Isključivanje nekorišćenih interfejsa
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


### Isključivanje menadžment servisa na svim interfejsima koji nisu za menadžment
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
FortiGate ima opciju prikupljanja informacija o krajnjim uređajima tako što sluša saobraćaj na LAN linkovima i obrađuje ga u jednom preglednom i korisnom prikazu.
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
Veliki broj korisnika ne briše podrazumevanog korisnika koji se koristi tokom inicijalizacije uređaja. Potrebno je u svakoj implementaciji izbrisati podrazumevanog korisnika, kao i definisati IP adrese preko kojih administratori mogu pristupiti uređaju.
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
Preporuka je da za svaki korisnički nalog postoji MFA konfiguracija. U tom slučaju, dobra je praksa imati jedan rezervni nalog u slučaju pada MFA servisa. 

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
Preporučuje se kreiranje novih administratorskih profila sa nivoem pristupa po potrebama administratora. 

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

Telnet port ne mora da se menja, s obzirom da se u svakoj implementaciji preporučuje isključivanje Telnet servisa na FortiGate uređaju.
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


### Isključivanje USB auto install opcije
Preporučeno je ugasiti opciju automatskog instaliranja verzije i konfiguracije na firewall uređaju pomoću USB interfejsa. Time štitimo našu mrežnu infrastrukturu od lica koji imaju fizički pristup opremi.

U slučajevima kada izvršavamo ZTP, odnosno LTP, implementaciju uređaja pomoću USB-a se preporučuje podrazumevano podešavanje dok se ZTP proces ne završi.
```
config system auto-install
	set auto-install-config disable
	set auto-install-image disable
end
```

[Technical Tip: Automatic installation of Firmware and system configuration](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Automatic-installation-of-Firmware-and-system/ta-p/197938)


### Isključivanje FortiCloud SSO pristupa
Preporučeno je isključivanje FortiCloud SSO pristupa zbog velikog broja verzija koje imaju slabost kroz ovaj tip pristupa.
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
FortiGate sa diskom bi trebalo da loguje saobraćaj na disk, pri čemu, kada se popuni disk, brisanje kreće od najstarijih ka novijim logovima.
``` 
config log disk setting
    set status enable
    set maximum-log-age 0
end
```

[Technical Tip: How to configure logging to disk on the FortiGate using the GUI or the CLI](https://community.fortinet.com/t5/FortiGate/Technical-Tip-How-to-configure-logging-to-disk-on-the-FortiGate/ta-p/216995)





## Firewall polise
Firewall Security obuhvata dva seta polisa:
 - Local-in polisa
 - Security polisa

Svaka polisa koristi različite tipove objekata koji su detaljnije definisani ispod.

### Geografski objekti
Geografski objekti su predefinisane liste IP adresa koje pripadaju internet provajderima u određenim državama. Mogu se koristiti u različitim polisama, između ostalog u local-in polisi. U budućnosti ćemo pomenuti Threat feed-ove koji sadrže liste IP adresa provajdera definisani kroz njihov jedinstveni AS broj.
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
ISDB objekti su lista internet servisa koji sadrže mapiranje destinacionih IP adresa i portova.

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
 
Slične objekte koristimo u produkcionim okruženjima na određenim projektima.

[Policy and Objects: Internet Services](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/849970/internet-services)


### Eksterni konektori
FortiGate podržava različite tipove eksternih konektora kroz threat feed-ove. Fokusiraćemo se na one koje se koriste u različitim segmentima Local-in i Security polisa.
 - FortiGuard Category Threat Feed - Koristi se za povlačenje eksternih lista URL-ova koji se referenciraju u Web filtering-u.
 - IP Address Threat Feed - Koristi se za povlačenje eksternih lista IP adresa koji se referenciraju u Local-in i Security polisama kao source i destinacioni objekti.
 - Domain Name Threat Feed - Koristi se za povlačenje eksternih lista domenskih imena koji se referenciraju u DNS filtering-u.
 - MAC Address Threat Feed - Koristi se za povlačenje eksternih lista MAC adresa koji se referenciraju u Security polisi, između ostalog.
 - Malware Hash Threat Feed - Koristi se za povlačenje eksternih lista Hash-eva koji se referenciraju u Antivirus profilima.

[Fortinet Security Fabric: External feeds](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/9463/external-feeds)


### Local-in polisa
Local-in polisa sadrži pravila koja dozvoljavaju pristup koji se terminira(počinje i završava) na samim interfejsima FortiGate uređaja.

Kada se omogući menadžment servis na samom interfejsu, ili se upali funkcionalnost, automatski se kreira Local-in pravilo.
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


### Virtual Patching
Konfiguracija local-in polise obuhvata segment zaštite uređaja na L4 nivou. Od verzije 7.4.1, uveden je koncept virtual patching-a, gde uređaj koristi IPS bazu kako bi blokirao pokušaje eksploatacije poznatih ranjivosti FortiGate uređaja. Virtual patching se između ostalog može koristiti i u local-in pravilima.

[Local-in Policies: Virtual patching on the local-in management interface](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/393161/virtual-patching-on-the-local-in-management-interface)


### Security polisa
Security polisa je sastavljena od pravila koja blokiraju ili dopuštaju saobraćaj koji prolazi kroz FortiGate firewall.

Konfiguracija security polise je iscrpan proces sa velikim brojem opcija koje su dostupne administratoru, kako bi što preciznije definisao tip saobraćaja koji želi da propusti. U ovom delu teksta se koncentrišemo na baseline pravila koja se preporučuju prilikom inicijalnog definisanja pravila.

Na sličan način kao i Local-in polisa se konfigurišu i pravila za Security polisu.
Baseline pravila se definišu sa sličnim objektima:
 - ISDB objekti - Treba obratiti pažnju na smer ISDB objekta. U zavisnosti od toga da li se ISDB objekat koristi za Source ili Destinaciju u Security pravilima, potrebno je prilagoditi grupu objekata.

 - Geografski objekti - Za saobraćaj sa interneta ka internoj mreži se mogu definisati samo države sa kojih je moguć pristup. Za saobraćaj ka internetu se može blokirati pristup ka državama za koje pristup nikad nije potreban(kao što su Kina, Severna Koreja, Rusija, Sirija, Iran, Brazil...)

 - Eksterni objekti - U nastavku ostavljamo nabrojane eksterne IP liste koje koristimo na nekim od projekata:
	- [Cumry Bogon lista](https://www.team-cymru.org/Services/Bogons/fullbogons-ipv4.txt)
	- [Emerging threats Block IPs](https://rules.emergingthreats.net/fwrules/emerging-Block-IPs.txt)
	- [Emerging threats Compromised IPs](https://rules.emergingthreats.net/blockrules/compromised-ips.txt)
	- [BBcan177 MS1 Block IPs](https://gist.githubusercontent.com/BBcan177/bf29d47ea04391cb3eb0/raw/)
	- [BBcan177 MS3 Block IPs](https://gist.githubusercontent.com/BBcan177/d7105c242f17f4498f81/raw/f69be712a06e998191adfe4c86d74e8cacf08d28/MS-3)
	- [CINSscore Bad Guys](http://cinsscore.com/list/ci-badguys.txt)
	- [Blocklist.de](https://lists.blocklist.de/lists/all.txt)

 - ASN objekti - U okviru eksternih objekata se može definisati i blokiranje po AS broju mreže servis provajdera. Pretragu ASN-a možete pronaći na ovom [linku](https://asn.ipinfo.app/search), dok link do liste izgleda ovako ```https://asn.ipinfo.app/api/text/list/AS<<ASN-BROJ>>```.
 
	Primer: [Informacije o ASN-u](https://asn.ipinfo.app/AS49402), [Lista ASN IP adresa](https://asn.ipinfo.app/api/text/list/AS49402)
 
 - Schedule objekti - Po potrebi, može se definisati vreme tokom kojeg je aktivno Security pravilo. U slučaju da je to više slotova, može se definisati i Schedule grupa.
 
 - Negate opcija - Za svaki objekat se može definisati negacija u okviru pravila negate opcijom.
 
 - Korisničke grupe - U Security pravilu se može izvršiti filtracija po grupama. Ovo je šira tema koja obuhvata tipove pristupa na pasivnu i aktivnu autentifikaciju korisnika.
 
	[User & Authentication](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/732715/user-definition-groups-and-settings)


### Send packet deny opcija
Podrazumevana podešavanja FortiGate firewall-a nalažu da uređaj 'tiho' odbacuje pakete bez obaveštenja korisnika. Većina drugih proizvođača šalje ICMP poruku Source IP adresi kao potvrdu da je saobraćaj blokiran od strane firewall-a. Slično ponašanje se može uključiti i na FortiGate firewall-ima.
```
config firewall policy
	edit <ID-SECURITY-PRAVILA>
		set send-deny-packet enable
	next
end
```

[Troubleshooting Tip: FortiGate did not reply with TCP RST when 'set send-deny-packet' is enabled](https://community.fortinet.com/t5/FortiGate/Troubleshooting-Tip-FortiGate-did-not-reply-with-TCP-RST-when/ta-p/379841)





## Security profili
