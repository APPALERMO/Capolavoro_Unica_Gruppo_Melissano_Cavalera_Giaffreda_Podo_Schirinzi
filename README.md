**Rete Aziendale**

Melissano – Cavalera – Giaffreda – Podo – Schirinzi

Simulazione basata su due edifici con tre piani, uniti da una rete globale e collegati via internet tramite il NAT.

**1° Edificio (maschera fissa)**

Nel primo edificio, al primo piano, troviamo il concentratore, accessibile esclusivamente dal PC1 tramite protocollo Telnet, e un server DHCP configurato per assegnare indirizzi IP a partire da 192.168.0.2, con un massimo di 254 host. Gli host della rete sono distribuiti tra il secondo e il terzo piano.

Per aumentare la sicurezza della rete abbiamo impostato delle password al concentratore e un messaggio che apparirà agli utenti che vogliono entrare all’interno del CLI dello switch tramite i seguenti comandi:
````
Switch>enable
Switch#config t
Switch(config)#enable secret 1234
Switch(config)#line console 0
Switch(config-line)#password 12345678
Switch(config-line)#login
Switch(config-line)#exit
Switch(config)#banner motd #Benvenuto nello Switch Concentratore dell'Edificio. Se non sei un tecnico o uno degli amministratori della rete non sei autorizzato ad accedere....#
Dopo di che, per configurare l’accesso da remoto via Telnet, abbiamo utilizzato i seguenti comando tramite il CLI del concentratore:
Switch(config)#line vty 0 15
Switch (config-line)#password 123456789
Switch (config-line)#login
Switch (config-line)#exit
Switch (config)#ip default-gateway 192.168.0.254
Switch (config)#interface vlan 1
Switch (config-if)#ip address 192.168.0.253 255.255.255.0
Switch (config-if)#no shutdown
````
In fine, per salvare tutte le modifiche applicate, abbiamo utlizzato il seguente comando:

````
Switch (config)#do copy running-conf startup-conf
````
**2°Edificio (maschera fissa)**

Nel secondo edificio, strutturato sempre su tre piani, utilizziamo il server presente nel primo edificio come helper per poter utilizzare il protocollo DHCP, potendo così configurare tutti gli host presenti.

Inoltre, per poter accedere alle pagine web, abbiamo configurato il protocollo HTTP nel server presente al terzo piano.

**Rete globale**

Nella rete presente tra i due edifici abbiamo configurato il RIP, utilizzando i seguenti comandi:
````
(modalità globale)

router rip

version 2

no auto-summary

network <indirizzo di rete adiacente>
````
Nel router 6 abbiamo dovuto aggiungere i seguenti comandi, necessari per il poter utilizzare il server DCHP anche nel secondo edificio:
````
Router(config)#interface Gig3/0
Router(config-if)#ip helper-address 192.168.0.3
````
Infine, per tradurre gli indirizzi privati in pubblici, abbiamo configurato lo strumento NAT Many-1 ai router di confine utilizzando i seguenti comandi:
````
(modalità globale)

interface <interfaccia interna>;
ip nat inside
interface <interfaccia esterna>;

ip nat outside  
access-list <id dell’access-list> permit <indirizzo della rete privata> <wildcard mask>;  
ip nat inside source list <id dell’access-list< interface <interfaccia esterna> overload
````
E il NAT 1-1 nel server del primo edificio con i seguenti comandi:
````
(modalità globale)

interface <interfaccia interna>

ip nat inside

interface <interfaccia esterna>

ip nat outside

ip nat inside source static <indirizzo privato del server> <indirizzo pubblico del gateway>
````
