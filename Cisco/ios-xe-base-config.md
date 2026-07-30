# Cisco IOS XE baseline konfiguracija prilikom inicijalizacije uređaja

***[DRAFT VERZIJA!!!]***

Baseline konfiguracija Cisco IOS XE svičeva prilikom inicijalizacije uređaja pre implementacije u produkciono okruženje.

- [Početak dokumenta](#fortigate-baseline-konfiguracija-prilikom-inicijalizacije-uređaja)
	- [Sistemska podešavanja](#sistemska-podešavanja)
		- [Podešavanje imena sviča](#podešavanje-imena-sviča)
		- [Upgrade sviča](#upgrade-sviča)
		- [StackWise stack-ovanje svičeva](#stackwise-stack-ovanje-svičeva)
		- [StackWise Virtual(SVL) stack-ovanje svičeva](#stackwise-virtualsvl-stack-ovanje-svičeva)
		- [Konfiguracija DNS servera](#konfiguracija-dns-servera)
		- [Konfiguracija NTP servera](#konfiguracija-ntp-servera)
		- [Konfiguracija SNMP servera](#konfiguracija-snmp-servera)
		- [Inicijalna sistemska konfiguracija](#inicijalna-sistemska-konfiguracija)
		- [Konfiguracija administrativnog pristupa](#konfiguracija-administrativnog-pristupa)
		- [Message-Of-The-Day(MOTD) banner](#message-of-the-daymotd-banner)
		- [Kreiranje alias komande](#kreiranje-alias-komande)
		- [FortiGate HA menadžment interfejs](#fortigate-ha-menadžment-interfejs)
	- [Podešavanja interfejsa](#podešavanja-interfejsa)
		- [Osnovna konfiguracija interfejsa](#osnovna-konfiguracija-interfejsa)
		- [Definisanje makro seta interfejsa](#definisanje-makro-seta-interfejsa)
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


## Sistemska podešavanja


### Podešavanja imena uređaja
Podrazumevana konfiguracija je da je ime uređaja ```Switch```. Preporučuje se podešavanje imena bez razmaka sa donjom crtom(_) i crticom(-).
```
hostname <IME-UREĐAJA>
```

Ako dodeljeno ime više od 20 karaktera, svič prikazuje upozorenje u porekoračenju broja karaktera. Iako će se ime promeniti, limit za prikaz upozorenja može da se modifikuje pomoću komande ```prompt config hostname-length <DUŽINA-IMENA>```.


### Upgrade sviča
U trenutku pisanja, Cisco preporučuje softverske verzije 17.15.4 i 17.12.6 za IOS-XE Cisco Catalyst 9200, 9300, 9400, 9500 i 9600 modele.

U ovom delu će biti definisana upgrade procedura za modele Cisco Catalyst 9200 i 9300 svičeve. Kako bi izvršili SMU(Software Maintence Upgrade) koji se zove još i cold patching, zbog prekida saobraćaja kroz uređaj tokom procesa nadogradnje.

Prvi korak nadogradnje je upload softverske verzije na uređaj. Cisco svičevi podržavaju veliki broj protokola za transfer fajlova, preporučeni su sigurni protokoli: SCP ili SFTP. Naknadno, transfer se može izvršiti i preko USB diska, sa time da je preporuka izvršiti formatiranje diska na FAT32, sa veličinom od 2GB ili 4GB. Pre transfera fajla na svič, preporučuje se brisanje svih softverskih verzija koje se ne koriste.
```
install remove inactive
copy <UPLOAD-PROTOKOL>:<SERVER>/<PUTANJA>/<IME-FAJLA> flash:<IME-FAJLA>
dir flash:<IME-FAJLA>
```

Poslednjom komandom proveravamo uspešnost prebacivanja softverske verzije na svič.

Nakon toga je potrebno promeniti boot varijablu i prebaciti način pokretanja sviča.
```
configure terminal
 boot system flash:package.conf
 no boot manual
 write memory
exit
show boot
```

Poslednjom komandom proveravamo promenu boot varijable.

Za kraj je potrebno instalirati softversku verziju na flash. Potvrdom procesa se svič restartuje i pokreće sa novom verzijom.
```
install add file flash:<IME-FAJLA> activate commit
```

Komandom ispod proveravamo verziju nakon pokretanja sviča.
```
show version
```

Modeli svičeva Cisco Catalyst 9400, 9500 i 9600 podržavaju i ISSU(In-Service Software Upgrade) pored SMU načina. ISSU se izvršava na sličan način kao i SMU. Razlike su u prikazu stanja.
```
show redudancy
show issu details
```

Instaliranje softverske verzije se izvršava komandom ispod.
```
install add file flash:<IME-FAJLA> activate issu commit
```

[SMU upgrade procedura](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9200/software/release/17-10/release_notes/ol-17-10-9200/upgrading_the_switch_software.html)
[ISSU upgrade procedura](https://www.cisco.com/c/en/us/support/docs/switches/catalyst-9400-series-switches/222283-upgrading-catalyst-9400-switches.html#toc-hId-1788513133)


### StackWise stack-ovanje svičeva
Cisco Catalyst 9200 i 9300 svičevi imaju opciju stack-ovanja kroz StackWise portove sa maksimalno 8 članova. Dok Cisco Catalyst 9400 i 9500 podržavaju StackWise Virtual koji će biti pokriveni u narednom segmentu.

Kako bi izvršili StackWise stack-ovanje potrebno je da svi uređaji u stack-u budu isti model i da imaju istu licencu.

Pre stack-ovanja svičeva je potrebno izvršiti renumber uređaja, pomoću čega se definiše redni broj seta interfejsa na tom sviču.

Odluka o aktivnom(primarnom) sviču u stack-u se odlučuje u nekoj od ove tri metode:

- Konfiguracija prioriteta sviča - Konfiguracija se vrši komandom ```switch <BROJ-SVIČA> priority <PRIORITET-SVIČA>```

- Definisanja konfiguracije sviča - Ako je jedan svič bez konfiguracije, on postaje standby(pasivni)

- MAC adresa sviča - Svič sa manjom IP adresom postaje aktivan(primarni)

Nakon inicijalne konfiguracije prioriteta i renumber-a, oba sviča se ugase, povežu StackWise kablovima i upale. Povezivanje stack-a se uvek vrši po crochet(criss-cross) šablonu, u prsten topologiji.

U slučaju da se dodaje svič na postojeći, potrebno je da se na njemu promeni prioritet, odradi renumber, ugasiti svič, povezati na postojeći stack i upaliti novi svič.

Prilikom stack-a se koristi MAC adresa i Bridge ID aktivnog(primarnog) uređaja.


### StackWise Virtual(SVL) stack-ovanje svičeva
Cisco Catalyst 9400 i veći ne podržavaju StackWise stack-ovanje, nego isključivo StackWise Virtual(SVL) sa maksimalno 8 svičeva, gde SVL nudi više funkcionalnosti.

Network Advantage licenca je obavezna za rad SVL-a. Potrebno je da svičevi u SVL budu isti model i verzija. VLAN ID 4094 mora biti rezervisan zato što se koristi u SVL i ne sme biti korišćen nigde drugde u mreži. Za SVL virtualni link se moraju koristiti linkovi iste brzine.

Bitno je napomenuti da C9400 serija ne podržava brzine od 100G za SVL virtualne linkove, dok C9500 i C9600 ne podržava 1G SVL virtualne linkove.

Menadžment i kontrolni plane saobraćaj se obrađuje samo na aktivnom(primarnom) uređaju. Servisni plane je distribuiran na svim uređajima unutar SVL-a. To znači da u slučaju da su ingress i egress interfejsi na istom sviču u SVL-u, saobraćaj se prosleđuje direktno na tom sviču. U slučaju da je ingress i egress na različitim svičevima, obrada se vrši samo na tim svičevima, ingress obrada na jednom, egress na drugom sviču, dok tranzitni svičevi u SVL ne vrše obradu saobraćaja.

Dva protokola se koriste za komunikaciju svičeva u SVL-u. 

- Link Management Protocol(LMP) - Koji se aktivira na svakom linku čim se uspostavi konekcija. LMP služi za proveru integriteta linkova, monitoring i proveru aktivnosti linkova 

- StackWise Discovery Protocol(SDP) - Koristi se za proveru kompatibilnosti modela i verzija svičeva, i odlučuje koji uređaj postaje aktivni, a koji standby.

Cisco StackWise Virtual Header(SVH) je frame header koji služi za 'enkapsulaciju' svog saobraćaja u SVL-u. Prepoznaju ga samo Cisco svičevi koji su konfigurisani za SVL.

Kada postoji Multi-Chassis EtherChannel(MEC), gde više svičeva u SVL učestvuje u LACP-u, svič na kome se nalazi ingress 
interfejs će slati paket na egress interfejsu na njemu gde god je to moguće. Po dizajnu se izbegava zagušenje SVL virtualnih linkova. 

Ako prilikom MEC-a padne jedan link, sa tog sviča se radi load-balance saobraćaja na linkove ka ostalim SVL svičevima.

Konfiguracija samog SVL-a je slična konfiguraciji StackWise-a.
```
configure terminal
 switch <BROJ-SVIČA> renumber <NOVI-BROJ-SVIČA>
 switch <BROJ-SVIČA> priority <PRIORITET-SVIČA>
```

Nakon toga je potrebno kreirati SVL domen.
```
 stackwise-virtual
  domain <BROJ-SVL-DOMENA>
 write memory
```

Preostalo je samo konfigurisati interfejse koji se koriste za SVL virtualni link.
```
configure terminal
 interface <IME-INTERFEJSA>
  stackwise-virtual link <INTERFEJS-LINK-ID>
  exit
 write memory
```

U slučaju da SVL virtuelni link padne, oba sviča prelaze u aktivno stanje gde može doći do Dual-Active(split-brain) moda. Kako se to ne bi desilo, može se konfigurisati Dual-Active Detection(DAD) gde bi jedan link služio za slanje heartbeat poruka kao drugi vid provere statusa SVL-a.
```
configure terminal
 interface <IME-INTERFEJSA>
  stackwise-virtual dual-active-Detection
  exit
 write memory
```

DAD link može da bude jedan link, a može da bude i PAgP link, odnosno enhanced PAgP(ePAgP) link.
```
configure terminal
 interface range <IME-INTERFEJSA1>, <IME-INTERFEJSA2>
  channel-group <PAGP-ID> mode desirable
  exit
 interface port-channel <PAGP-ID>
  shutdown
  exit
 stackwise-virtual
  dual-active detection pagp
  dual-active detection pagp trust channel-group <PAGP-ID>
  exit
 interface port-channel <PAGP-ID>
  no shutdown
  exit
 write memory
```

U slučaju da na jednom sviču ne postoji interfejs sa VLAN ID-om BUM saobraćaja, flood domen se ne širi na taj svič čime se izbegava zagušenje SVL virtualnih linkova. Konfiguracija je u globalnom konfiguraciom modu.
```
configure terminal
 svl l2bum optimization
 write memory
```

Kada dođe do pada SVL-a, aktivni svič uđe u recovery mod, dok standby preuzme ulogu aktivnog. Prilikom oporavka SVL-a, svič u recovery modu izvršava reload pre nego što se konačno priključi SVL-u. Kako bi izbegli reload i povratak sviča u SVL zbog bilo kakvog razloga, potrebno je ugasiti recovery reload opciju.
```
configure terminal
 stackwise-virtual
  dual-active recovery-reload-disable
  exit
 write memory
```


### Konfiguracija DNS servera
Na svičevima je potrebno konfigurisati DNS servere.

U slučaju da ne postoji interni DNS server, preporuka je korišćenje javnih Cisco Umbrella DNS servera:
``` 
208.67.222.222
208.67.220.220
``` 

Pored toga, preporučuje se promena DNS protokola, najčešće je u pitanju DNS preko UDP/TCP porta 53, odnosno cleartext DNS.
```
configure terminal
 ip domain-name <IME-DOMENA>
 ip name-server <DNS-SERVER1> <DNS-SERVER2>
 ip domain-lookup
 write memory
```


### Konfiguracija NTP servera
Podrazumevana vrednost su FortiGuard NTP serveri. Preporuka je da se promene na interne servere sa definisanim NTP servisom.

U slučaju da ne postoji interni NTP server, preporuka je korišćenje javnih popularnih NTP servera:
```
0.pool.ntp.org
time.nist.gov
```

```
configure terminal
 ntp server <NTP-SERVER1> prefer
 ntp server <NTP-SERVER2>
 write memory
```

Pored NTP servera, potrebno je definisati pravilnu vremensku zonu:
``` 
clock timezone CET 1 0
clock summer-time CEST recurring last Sun Mar 2:00 last Sun Oct 3:00
```

Ako je potrebno konfigurisati autentifikaciju za NTP saobraćaj, postoji mogućnost.
``` 
ntp authenticate
ntp authentication-key 5 md5 <KLJUČ>
ntp trusted-key 5
ntp server <NTP-SERVER1> key 5
ntp server <NTP-SERVER2> key 5
```


### Konfiguracija SNMP servera
Najsigurniji SNMP protokol u ovom trenutku je SNMP verzija 3. Međutim, zbog kompleksnosti implementacije polling-a razumljivo je korišćenje SNMP verzije 2.

Preporučeno je definisati dodatne informacije o uređaju.
```
configure terminal
 snmp-server contact <MEJL-ADMINISTRATORA>
 snmp-server location <IME-LOKACIJE>
 write memory
```

U podrazumevanoj konfiguraciji prilikom reboot-a, menja se indeksiranje uređaja, što nije željeno ponašanje.
```
snmp ifmib ifindex persist
```

Kreiranjem grupe i korisnika pravimo template koji možemo koristiti za veći broj SNMP servera. Pored toga je potrebno limitirati MIB stablo sa samo granama koje dozvoljavamo da SNMP server polluje. U ovoj konfiguraciji smo primenili komandu ```iso``` dozvoljavamo pristup celom stablu.
```
snmp-server view <IME-PROFILA-PRIKAZA> iso included
snmp-server group <IME-PROFILA-GRUPE> v3 priv read <IME-PROFILA-PRIKAZA> access <IME-ACL>
snmp-server user <IME-KORISNIKA> <IME-PROFILA-GRUPE> v3 auth sha <ŠIFRA-KORISNIKA> priv aes 128 <ŠIFRA-PRIV>
```

Nakon toga uključujemo korišćenje SNMP trap-ova pored polling-a.
```
snmp-server enable traps
```

Na kraju je potrebno samo definisati SNMP servere i primeniti određenog korisnika za njega.
```
snmp-server host <IP-SNMP-SERVERA> version 3 priv <IME-KORISNIKA>
```


### Inicijalna sistemska konfiguracija
Inicijalna konfiguracija se sastoji od osnovne zaštite uređaja od poznatih slabosti i osnovna enkripcija osetljivih delova konfiguracije na uređaju.
```
configure terminal
 service tcp-keepalives-in
 service tcp-keepalives-out
 
 service password-encryption
 
 no service pad
 no service config
 
 no ip source-route
 
 no vstack
 
 vtp mode off
 
 write memory
```


### Konfiguracija administrativnog pristupa
Konfiguracija administrativnog pristupa je preporučena samo preko sigurnih kanala(SSH i HTTPS). Međutim, zbog velikog broja slabosti vezanog za HTTP/S pristup, preporučuje se korišćenje samo SSH pristupa.

Prvo generišemo SSH ključ i definišemo parametre u SSH meniju. Pored toga, potrebno je upaliti SCP u slučaju da se on koristi za transfer fajlova prilikom upgrade-a.
```
configure terminal
 crypto key generate rsa modulus 4096
 ip ssh version 2
 ip ssh time-out 60
 ip ssh authentication-retries 3
 ip scp server enable
 write memory
```

Kao što je pomenuto, preporučuje se gašenje administrativnih servisa koji se ne koriste.
```
no ip http server
no ip http secure server
```

Preporučeno je definisati ACL za pristup uređaju samo sa definisanih IP adresa administratora.‚
```
ip access-list standard <IME-SSH-ACL>
 permit <MENADŽMENT-MREŽA1> <MASKA1>
 permit <MENADŽMENT-MREŽA2> <MASKA2>
 deny any log
```

Zatim se definišu administratorski korisnici, lokalno ili definisanjem udaljenog autentifikacionog servera.

Kada je autentifikacija sa lokalno definisanim korisnicima, preporučuje se konfiguracija aaa segmenta svakako, iako je ona primarna za udaljene autentifikacione servere. Definišu se administratorski nalozi lokalno.
```
username <IME-ADMINISTRATORA> privilege 15 algorithm-type sha256 secret <ŠIFRA-ADMINISTRATORA>
enable algorithm-type sha256 secret <ŠIFRA-ENABLE>

aaa new-model
aaa authentication login <IME-AUTH-PROFILA> local
aaa authentication enable <IME-AUTH-PROFILA> enable
aaa authorization exec <IME-AUTH-PROFILA> local if-authenticated
aaa authorization config-commands
aaa authorization console
aaa login display number-failures
aaa login success-track-conf-time 1

line vty 0 15
 exec-timeout 30 0
 logging synchronous
 access-class <IME-SSH-ACL> in
 login authentication <IME-AUTH-PROFILA>
 authorization exec <IME-AUTH-PROFILA>
 transport input ssh
 transport preferred none
exit
line con 0
 exec-timeout 30 0
 logging synchronous
 login authenctication <IME-AUTH-PROFILA>
 transport preferred none
exit
line aux 0
 transport input none
 transport output none
 transport preferred none
exit
```

Kada je autentifikacija pomoću udaljenog servera, potrebno je dodati server preko kojeg se vrši autentifikacija, kao i protokol koji se koristi.

Radius server konfiguracija:
```
aaa new-model

radius server <IME-RADIUS-SERVERA1>
 address ipv4 <IP-RADIUS-SERVERA1> auth-port <RADIUS-PORT> acct-port <ACCOUNTING-PORT>
 key <TAJNI-KLJUČ>
exit

aaa group server radius <IME-RADIUS-GRUPE1>
 server name <IME-RADIUS-SERVERA1>
exit

aaa authentication login <IME-AUTH-SSH-PROFILA> group <IME-RADIUS-GRUPE1> local
aaa authentication login <IME-AUTH-CON-PROFILA> local
aaa authorization exec <IME-AUTH-SSH-PROFILA> group <IME-RADIUS-GRUPE1> local if-authenticated
aaa accounting exec <IME-AUTH-SSH-PROFILA> start-stop group <IME-RADIUS-GRUPE1>
aaa authorization config-commands
aaa authorization console
aaa login display number-failures
aaa login success-track-conf-time 1

line vty 0 15
 exec-timeout 30 0
 logging synchronous
 access-class <IME-SSH-ACL> in
 login authentication <IME-AUTH-PROFILA>
 authorization exec <IME-AUTH-PROFILA>
 accounting exec <IME-AUTH-PROFILA>
 transport input ssh
 transport preferred none
exit
line con 0
 exec-timeout 30 0
 logging synchronous
 login authenctication <IME-AUTH-PROFILA>
 transport preferred none
exit
line aux 0
 transport input none
 transport output none
 transport preferred none
exit
```

Opciono se mogu modifikovati tajmeri.
```
radius server <IME-RADIUS-SERVERA1>
 radius-server timeout 5
 radius-server retransmit 3
 radius-server deadtime 10
```

Jedina razlika prilikom konfiguracija LDAP-a je prvobitno definisanje servera i aaa grupe.
```
aaa new-model

ldap server <IME-LDAP-SERVERA1>
 host <IP-LDAP-SERVERA1>
 port <LDAP-PORT>
 base-dn <DISTINGUISHED-NAME>

aaa group server ldap <IME-LDAP-GRUPE1>
 server name <IME-LDAP-SERVERA1>
```


### Message-Of-The-Day(MOTD) banner
Prilikom uspešne autentifikacije imamo opciju konfiguracije MOTD upozorenja kako bi odvratili neautorizovan pristup opremi. Većina kompanija ima potrebu za MOTD upozorenjem zbog compliance-a.
```
configure terminal
 banner motd ^
 ====================================================
 UNAUTHORIZED ACCESS TO THIS SYSTEM IS FORBIDDEN!!!
 ====================================================
 ^
 banner login ^
     #     #####  #     # #######       #####    #####  ######  ######
    # #   #     # ##   ## #            #     #  #     # #     # #     #
   #   #  #       # # # # #            #        #     # #     # #     #
  #     # #       #  #  # #####        #        #     # ######  ######
  ####### #       #     # #            #        #     # #   #   #
  #     # #     # #     # #            #     #  #     # #    #  #
  #     #  #####  #     # #######        #####   #####  #     # #
 *********************************************************************
                        hostname: <IME-UREĐAJA>                
 *********************************************************************
                        MGMT IP adresa: <IP-ADRESA-UREĐAJA>                  
 *********************************************************************
 ================================================================================
 NEOVLASCENI PRISTUP CE SE SMATRATI KRIVICNIM DELOM I BICE SUDSKI PROCESUIRAN!!!
 UNAUTHORIZED ACCESS TO THIS SYSTEM IS FORBIDDEN AND WILL BE PROSECUTED BY LAW!!!
 ================================================================================
 ^
 write memory
```


### Kreiranje alias komande





## Podešavanja interfejsa


### Osnovna konfiguracija interfejsa
Osnovna konfiguracija interfejsa na Catalyst svičevima obuhvata Layer 1, 2 i 3 konfiguraciju, i biće kratko definisana u ovom segmentu. Više informacija o svakom od segmenata ispod.

- Dodavanje opisa interfejsa - Opis se može definisati na bilo kom tipu fizičkog ili logičkog interfejsa
  ```
  configure terminal
   interface <IME-INTERFEJSA>
    description <OPIS-INTERFEJSA>
   write memory
  ```

- Definisanje tipa duplex-a - Half duplex je moguće definisati samo na fizičkim interfejsima sa brzinom manjom od 1000Mb/s
  ```
  configure terminal
   interface <IME-INTERFEJSA>
    duplex <TIP-DUPLEX-A>
   write memory
  ```

- Flowcontrol na interfejsu - Prilikom zagušenja, svič može da dobije *pause* frejm kojim označava prestanak slanja i primanja frejmova dok se zagušenje ne otkloni
  ```
  configure terminal
   interface <IME-INTERFEJSA>
    flowcontrol receive on
   write memory
  ```

- Definisanje tipa konzolnog pristupa - Moguće je eksplicitno definisanje tipa konekcije koji se koristi za konzolni pristup(RJ45 ili USB), podrazumevani pristup kada su oba u funkciji je USB
  ```
  configure terminal
   line console 0
    media-type rj45
   write memory
  ```

-  

- tdr, mtu, power supply, eee, perpetual and fast poe

- gašenje usb konzole, lldp tlv, 

- l3, gre tunel,


### Definisanje makro seta interfejsa
Na Cisco Catalyst svičevima je moguće definisati makro set interfejsa umesto komande za opseg.

Moguće je kreirati veći broj macro-a koji se mogu koristiti za različite stvari(korisnički interfejsi, štampači, veze ka svičevima itd...) 
```
configure terminal
 define interface-range <IME-MACRO-A> <IME-INTERFEJSA1> - <IME-INTERFEJSA2>, <IME-INTERFEJSA3>
 interface range macro <IME-MACRO-A>
  ...konfiguracija interfejsa...
 write memory
```






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
