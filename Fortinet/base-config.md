# FortiGate baseline konfiguracija prilikom inicijalizacije uređaja

Baseline konfiguracija FortiGate firewall uređaja prilikom inicijalizacije uređaja pre implementacije u produkciono okruženje.

## Template



### Konfiguracija DNS servera
Podrazumevana vrednost su FortiGuard DNS serveri. Preporuka je da se promene na interne DNS servere.

U slučaju da ne postoji interni DNS server, preporuka je korišćenje javnog Cisco Umbrella DNS servera:
``` 
208.67.222.222
208.67.220.220
``` 

Pored toga, preporučuje se promena DNS protokola, najčešće je u pitanju DNS UDP-TCP/53, odnosno cleartext DNS.
``` 
config system dns
    set primary <IP-DNS-PRIMARNI>  
    set secondary <IP-DNS-SEKUNDARNI>  
    set protocol cleartext 
end
```

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

### Blokiranje intra-zone saobraćaja
FortiGate ne mora biti, ali bi ga trebalo konfigurisati kao zone-based firewall. Kada se definišu zonu, potrebno je konfigurisati intra-zone blokiranje pravila, gde bi u okviru firewall polise definisali propuštanja po potrebi.
```
config system zone
    edit <IME-ZONE> 
        set intrazone deny
    next
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

### Podešavanja imena uređaja
Podrazumevana konfiguracija je da je ime uređaja serijski broj. Preporučuje se podešavanje imena bez space-a sa donjom crtom i crticom.
```
config system global
	set hostname <IME-UREĐAJA>
end
```

### Upgrade firewall uređaja
Potrebno je uvek pratiti novosti PSIRT-a vezane za slabosti firmware verzija uređaja. Kada se pronađe slabost, potrebno je da se zakrpi u što kraćem vremenskom roku.

[Technical Tip: Recommended release for FortiOS](https://community.fortinet.com/t5/FortiGate/Technical-Tip-Recommended-release-for-FortiOS/ta-p/227178)

### Gašenje USB auto install opcije
Preporučeno je ugasiti opciju automatskog instaliranja verzije i konfiguracije na firewall uređaju pomoću USB interfejsa. Time štitimo našu mrežnu infrastrukturu od lica koji imaju fizički pristup opremi.

1

U slučajevima kada izvršavamo ZTP, odnosno LTP, implementaciju uređaja pomoću USB-a se preporučuje podrazumevano podešavanje dok se ZTP proces ne završi.
```
config system auto-install
	set auto-install-config disable
	set auto-install-image disable
end
```

### Konfiguracija password pravila
Od 7.6.5 je default password pravilo prebačeno na 12 karaktera, potrebno je na starijim verzijama ponašanje definisati kroz password polisu.

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
Zaprepašćujući broj korisnika ne briše podrazumevanog korisnika koji se koristi tokom inicijalizacije uređaja. Potrebno je u svakoj implementaciji izbrisati podrazumevanog korisnika, kao i definisati IP adrese preko kojeg administratori mogu pristupiti uređaju.
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

### Konfiguracija administratorskog profila
Preporučuje se kreiranje novih administratorskih profila sa nivoem pristupa po tipskoj potrebi administratora. 

Jedan primer pravilnog limitiranja pristupa postoji u našem CT Cloud-u, gde se za određenog korisnika dopušta pristup CyberStellar alatu koji dinamički upisuje blacklist-ovane IP adrese kroz adresne objekte u postojeću adresnu grupu koja se koristi u firewall pravilu.
```
config system accprofile
    edit "<IME-ADMINISTRATORSKOG-PROFILA"
        set fwgrp custom
        config fwgrp-permission
            set address read-write
        end
    next
end
```

### Menjanje podrazumevanih menadžment portova



### Povećavanje timeout-a nakon unosa pogrešne šifre administratora
Podrazumevana vrednost je 60 sekundi. Preporučuje se povećanje timer-a na bar 5 minuta(300 sekundi)
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

### Create configuration revisions on logout
Configuration revisions can be automatically created upon a logout. This can help with tracking down changes.
``` 
config system global
    set revision-backup-on-logout enable
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

### Konfiguracija SNMP servera
Najsigurniji SNMP protokol u ovom trenutku je SNMP verzija 3. Međutim, zbog kompleksnosti implementacije polling-a, je razumljivo korišćenje SNMP verzije 2.

Svakako, lakši segment konfiguracije je SNMP traps u okviru verzije 3 na koji se fokusiramo ispod.
```
config system snmp sysinfo
	set status enable
	set desciption "<IME-UREĐAJA>"
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

### Uključivanje korišćenja CDN-a za GUI performanse
Preporučena konfiguracija za korišćenje CDN-a za ubrzavanje GUI odziva. Podrazumevana konfiguracija je da je opcija upaljena, ali treba znati za nju.
``` 
config system global
    set gui-cdn-usage enable
end
```

### Uključivanje korišćenja lokalnog ISDB keša
FortiGate lokalno čuva ISDB ulaze za veliki broj servisa. Korišćenje lokalnog keša pomaže u smanjivanju opterećenja manjih uređaja kod povlačenja razlike ulaza.
``` 
config system settings
    set internet-service-database-cache enable
end
```

