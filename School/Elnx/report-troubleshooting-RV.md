# Enterprise Linux Laboverslag - Troubleshooting

- Student: Verstraeten Renaud

- Klasgroep:  TIN2-TI/3B (Aalst)

  

## Verslag

### Fase 1: Netwerk Acces Layer

* Kabels checken : onder network in virtual box checken of onder het luik advanced van elke adapter "cable connected" is aangevinkt
    
    * **oke**
* Virtuele adapters checken, 1 is nat de andere is Itnet
  * Volgens mij moet de itnet veranderen naar een host-only adapter zoals in de demo
    
    * **Verander adapter naar host-only adapter**
    
      

### Fase 2: Internet Layer

* Lokale netwerk configuratie:
  * Ip adres checken van elke adapter: **ip a** moet mij voor elke adapter een ip tonen
    * **Ip a** geeft aan dat Eth0 geen ip configuratie heeft dus ik ga naar zijn config bestand zoeken in: **/etc/sysconfig/network-scripts/**
      * ik weet welke adapter dus ik doe gewoon: **sudo vi /etc/syscinfig/network-scripts/ifcfg-eth0**
      * Ik zie geen ip adres noch subnet mask aanwezig , ik zie ook dat bootproto op none staat, ik ga dit even veranderen naar **BOOTPROTO = DHCP** om te kijken of Eth0 een ip krijgt van onze DHCP server via Virtual box (i voor insert), **:wq** voor write en quit in vi
      * Herstarten netwerk: **sudo systemctl restart network**
      * Ik doe opnieuw **ip a** en ik zie 10.0.2.15/24 (mask is fout maar via virtual box bla bla .. zie uitleg Meneer Van Vreckem)
      * Nu pas snap ik waarom ik niet kon SSH'en naar de server
    * Rest van de adapters lijkt juist op het eerste zicht

  * Default gateway checken of aanwezig: **ip r**
    * Gateways aanwezig
    * ik kan pingen naar 10.0.2.2 default -> ping succesvol

  * DNS server opvragen: **cat /etc/resolv.conf**
    * We krijgen dns server 10.0.2.2
    * Werkt dns wel: **dig icanhazip.com**
      * package bind-utils niet geïnstalleerd
        * **sudo yum install bind-utils** moet werken want we hebben internet connectie door de nat interface
      * **dig icanhazip.com** geeft antwoord dus DNS werkt



### Fase 3: Transport layer

* Services running?
  * Het gaat om een lamp stack dus httpd service moet draaien: opvragen via **systemctl status httpd**
    * We zien dat hij disabled is dus we starten via: **sudo systemctl  start httpd** 
      * Starten werkt niet, mijn log venster verteld me dat er syntax fouten zijn in de conf dus we gaan is kijken: **sudo vi /etc/httpd/conf/httpd.conf**

* We zien in de logs dat de ServerRoot fout is, dit moet zijn **/etc/httpd** dus we veranderen dit
      * We zien direct dat de poort niet op Listen 80 staat wat nodig is voor een HTTPD service, we veranderen de poort naar 80 voor verdere problemen te vermijden passen we dit ook direct aan

* Starten lukt nog steeds niet , dus ik ga kijken welke poorten al gebruikt worden (listen): **ss -tlnp | grep :80** 
      
  * we zien dat een ander proces(lighttpd) deze poort al gebruikt, dit klopt niet
          * We stoppen dit proces: **sudo sytemctl kill lighttpd**
      *  Nu lukt starten wel: **sudo systemctl start httpd**
        * opvragen status, status moet actief zijn: **sudo systemctl status httpd**

* Snel eens naar de firewall settings kijken: **sudo firewall-cmd --list-all**
  * hier moeten oftewel de ports 443 en 80 toegelaten zijn ofwel de services http/https toegelaten zijn
    * ik zie http maar voor de secure versie erbij te hebben voegen we aan services https toe: **sudo firewall-cmd --add-service https --permanent**
    * herstarten van de service: **sudo firewall-cmd --reload**



### Application layer

* Config file van apachectl checken via de apache test
  * apachectl configtest
    * syntax ok
    * ik krijg wel de Could not reliably determine the server's fully .. maar ik denk dat dit normaal is aangezien geen dns services



### Tijd om te testen

* Ingeven van 192.168.56.8 op fysieke machine zou mij de tweets pagina moeten geven
  * ik krijg iets maar Acces denied

* SELinux dan maar
  * ik ga eens kijken naar de phpfile in: **/var/www/html/**
    * om de permissies (ook deze van SELinux) te bekijken doen we: **ls -lZ**
      * hier zien we dat de Users kunnen lezen, groepen kunnen schrijven en lezen, all kan niets
        * ik denk dat we dit moeten aanpassen naar read voor iedereen: **sudo chmod a+r index.php**
      * we zien ook dat de context fout is, is nu user_home en dit moet **httpd_sycontent_t** zijn
        * cmd hiervoor: **sudo restorecon -r /var/www** om de file context naar default value te brengen
  
* Nu geven we nog is 192.168.56.8 in 
  * We zien de webpagina!!!



## Eindresultaat

Alles werkt! 

[320279rv@fixme bin]$ acceptance.bats
 ✓ SELinux shoud be enforcing
 ✓ The firewall should be running
 ✓ I should have the correct IP address
 ✓ The Apache service should be running
 ✓ The correct website should be served

5 tests, 0 failures

## Referenties

<https://github.com/Renaud132/MyRepo/blob/main/School/Elnx/Troubleshooting.md>

<https://stackexchange.com/>
