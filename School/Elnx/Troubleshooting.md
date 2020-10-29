# **Troubleshooting**

### Bottom-up Approach

#### Lagen  (tcpip zal volstaan voor onze troubleshoot)

* Physical (cables)

* Network acces layer of Data link laag
* Internet layer
* Transport layer
* Application layer

#### SELinux

#### Fase1: 

###### Netwerk Acces Layer

* Kabels checken in VirtualBox

  * Onder Network > advanced > Cable connected aanvinken

  

* Virtuele adapters checken + settings

#### Fase2:

###### Internet layer

* Local network configurarion
  * Ip adres checken: **ip a** 
    * Heeft elke adapter een Ip adr - subnet - dhcp or fixed ip 
    * Check configuration voor elke adapter **/etc/sysconfig/network-scripts/ifcfg-*** 
      * Voor elke adapter config file, openen met **vim ifcfg-eth0** (VB)
      * Als het ip goed staat in de config file maar niet in het resultaat van de opvraag met cat -> herstarten netwerk
    * Na veranderingen netwerk-service herstarten: **sudo systemctl restart network** 
  * Default gateway checken of aanwezig: **ip r**
    * ip adres + subnet?
  * Dns: **cat /etc/resolv.conf**  
    * Krijgen we een nameserver?
  * Route-Tabel opvragen: **ip route**
  * Ping between hosts
  * Ping default GW/DNS  
  * Query DNS (Werkt dns??)
    * dig icanhazip.com (package bind-utils nodig hiervoor)
    * **nslookup** icanhazip.com 
    * getent ahosts icanhazip.com (als de bind-utils niet geinstalled zijn / je hebt geen internet connectie)

#### Fase3:

###### Transport layer

* Service running?
  * alle services: **systemctl list-units --type=service** (--all) voor ook niet actieve services te tonen
  * systemctl status httpd (draait apache)
    * **sudo systemctl start httpd** om te starten
* Correct port/interface?
  * Use ss
    * TCP: **sudo ss -tlnp**
    * UDP: **sudo ss -ulnp**
    * Poorten/interface checken: **tree /etc/servicename/conf/servicename.conf** en zoeken naar het .conf bestand
      * **sudo vim /etc/httpd/conf/httpd.conf +/8080** (vb voor naar de plek te gaan in het bestand op het eerste voorkomen van '8080') HTTPD poorten zijn 443 en 80
      * HERSTART SERVICE
  * Firewall settings
    * **sudo firewall cmd --list -all** (alle configuratie)
      * Is de service toegelaten/aanwezig?
        * **sudo firewall-cmd --add-service http --permanent**
        * **sudo firewall-cmd --add-service https --permanent**
      * HERSTART DE SERVICE **-- sudo firewall-cmd --reload**

#### Fase4:

###### Application layer

* logs, open aparte terminal om kijken
* Check config file syntax (elke applicatie is dit anders)
  * bij apache: **apachectl configtest**
* Use command line client tools
  * curl
  * smbclient
  * dig
  * netcat

#### SELinux

* Acces denied op een php script -> **/var/www/html/phpfile.php**

* Permissies van bestanden in een folder: **ls -l**
  * voor SELinux permissies erbij: **ls -lZ**
* Status SELinux: sestatus
* Logs -> bekijk SELinux prevent + bekijk Oplossing ervoor
* Indien apache; in de html folder ls -lz de context moet httpd_sys_content_t zijn
  * Set file context to default value: **sudo restorecon -r /var/www**
  * Booleans opvragen en aanpassen
    * **getsebool -a | httpd**
    * **sudo setsebool -P httpd_can_network_connect_db on** (P voor permanent)



#### Andere opmerkingen/commando's

- (Op Linux) **./query_db.sh** om DB server te testen
- Logs bekijken: **sudo journalctl -f**



