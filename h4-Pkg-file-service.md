# H4 Pkg-file-service

## Harjoituksessa käytetyn koneen parametrejä

- Kannettavan tietokoneen malli: HP Elitebook 840 G4
- Käyttöjärjestelmä: Windows 10 Pro, versio 22H2
- Prosessori: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz
- Näytönohjain: Inter(R) HD Graphics 620
- RAM: 8 gb

------------------------------------------------------------------------

## x) 


---------------------------------------------------------------------------------------

Seuraavissa harjoituksissa tutustun Salt:n pkg-file-service tilafunktioihin. 

## a)

Aluksi kokeilen asentaa Apachen ja korvata testisivun manuaalisesti. Ohjeistuksena käytän relevantein osin https://terokarvinen.com/2018/04/10/name-based-virtual-hosts-on-apache-multiple-websites-to-single-ip-address/.

Yrittäessäni alustaa Vagrantilla uudet virtuaalikoneet törmäsin kuitenkin ongelmaan. VM:n asennukset kestivät aina kauan ja lopulta jumiutuivat ilman virheilmoituksia. Lopulta testattuani eri laatikoilla ja vagrantfile-konfiguraatioilla päädyin 1) asentamaan vagrantin uusimpaan versioon ja 2) poistamaan aiemmassa harjoituksessa asentamani vbguest-pluginin (https://skillslane.com/vagrant-virtualbox-guest-additions-issue-resolution/). Tämän jälkeen uusien VM:n alustus onnistui, eli hypoteesini on että uusi vagrantin versio ei ollut enää yhteensopiva kyseisen pluginin kanssa.

Luotuani kaksi virtuaalikonetta harjoituksen [H2](https://github.com/Tuomasken/Linux-Hallinta/blob/main/h2-Soitto-kotiin.md) mukaisesti, otin ssh-yhteyden herrakoneeseen t001 ja asensin apachen (ja curlin, palomuurin ja micro-editorin) komennolla:

    $ sudo apt-get -y install apache2 curl ufw micro

Tämän jälkeen testasin näkyykö testisivu komennolla:

    $ curl localhost

Tämä toimi joten siirryin korvaamaan testisivun sisällön komennolla:

    $ echo "Testisivu" | sudo tee /var/www/html/index.html

![image](https://github.com/user-attachments/assets/cb47fb9f-ea6f-441d-bcaf-f4dd441eb1c7)

Seuraavaksi lähdin asentamaan herra-orja arkkitehtuuria virtuaalikoneilleni. Asensin harjoituksen [H2](https://github.com/Tuomasken/Linux-Hallinta/blob/main/h2-Soitto-kotiin.md) mukaisesti Salt:n herrakoneelle t001 ja orjakoneelle t002. Tässä kohtaa kuitenkin kävi nolo moka, ja ajatuksissani asentaessani tarvittavia ohjelmia orjakoneelle, asensin myös apachen. Vaikka tämän harjoituksen ideana on asentaa apache etänä herrakoneella orjakoneelle, niin se että apache on valmiina asennettuna orjakoneelle ei vielä täysin invalidoi harjoitusta. Salt:n tilafunktiot kertovat onnistuiko komento, vaikka sen lopputulema olisi jo entuudestaan tosi. Eli päätin edetä harjoituksessa normaalisti.

Asennettuani herra-orja arkkitehtuurin lähdin luomaan sls-tiedostoa, jolla ajaa komentoja herralta orjalle.



## Lähteet

1. Karvinen, Tero 2025, Palvelinten Hallinta. Viitattu 16.04.2025. https://terokarvinen.com/palvelinten-hallinta/
