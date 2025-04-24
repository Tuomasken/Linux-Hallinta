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

Seuraavaksi lähdin asentamaan herra-orja arkkitehtuuria virtuaalikoneilleni. Asensin harjoituksen [H2](https://github.com/Tuomasken/Linux-Hallinta/blob/main/h2-Soitto-kotiin.md) mukaisesti Salt:n herrakoneelle t001 ja orjakoneelle t002. Lopuksi ajoin orjakoneella komennon

        $ sudo apt remove apache2 curl

Jolla varmistin, että harjoituksen kannalta relevantit paketit eivät ole orjakoneella entuudestaan.

Asennettuani herra-orja arkkitehtuurin lähdin luomaan sls-tiedostoa, jolla ajaa komentoja herralta orjalle. Aluksi loin moduulin komennolla:

        $ sudo mkdir -p /srv/salt/apache

Ja sen jälkeen aloin kirjoittamaan sls-tiedostoa, joka heijastelee mitä aiemmin tehtiin manuaalisesti apachen asennuksessa ja testisivun korvaamisessa. Tukena käytin sivuja https://docs.saltproject.io/en/3006/ref/states/all/salt.states.file.html#salt.states.file.managed ja https://www.linode.com/docs/guides/configure-apache-with-salt-stack/

Kirjoitin seuraavanlaisen init.sls:

![image](https://github.com/user-attachments/assets/49b66b11-8d1d-4dd7-8f9e-b52eb80c6898)

Kokeilin ajaa tiedoston orjalle t002:

      $ sudo salt t002 state.apply apache

Ja sain seuraavanlaisen virheilmoituksen: 

![image](https://github.com/user-attachments/assets/ea813de3-df6e-441c-9bf1-05d88b04ed41)

Virheilmoituksen mukaisesti siirryin takaisin init.sls tiedostoon tarkastelemaan tilannetta. Lisäsin ":" apache2 perään ja ajoin komennon uudelleen. Sain virheilmoituksen: 

![image](https://github.com/user-attachments/assets/40086520-df9d-4498-a660-18eb9458b48c)

Siirryin takaisin sls-tiedostoon. Huomasin että index_html:n nimen määrittelystä puuttui välilyönti. Korjasin asian ja ajoin komennon uudelleen:

![image](https://github.com/user-attachments/assets/3ee8012b-6eb1-41c4-aa99-607d47371891)

Apache2-paketin asennus ja index.html-tiedoston muokkaukset onnistuivat. Apache2_service onnistui, mutta ei tehnyt muutoksia. Oletan tämän johtuvan siitä, että apache2 oli jo kertaalleen asennettuna t002-koneelle, ja remove-komento ei ollut täysin poistanut apachea siltä.

Curl-paketin asennus kuitenkin epäonnistui:

![image](https://github.com/user-attachments/assets/675aced1-8438-4aca-9363-faff775b2376)

Virheilmoituksessa puhuttiin kellonajoista, ja minulle tuli etäinen muistikuva siitä, että jos järjestelmän kello on väärässä ajassa, niin se voi vaikuttaa joihinkin asioihin. Tarkastin kellonajan komennolla

        $ timedatectl status

Ja kuinka ollakaan, se oli vuorokaudella väärässä. Konsultoin ChatGPT:tä (https://chatgpt.com/) asiasta, ja se neuvoi poistamaan automaattisen ajan synkronisoinnin hetkellisesti poispäältä ja muuttamaan ajan manuaalisesti. Komennot näyttivät tältä:

        $ sudo timedatectl set-ntp false
        $ sudo timedatectl set-time '2025-04-24 11:01:00'
        $ sudo timedatectl set-ntp true

Asetettuani uuden ajan kokeilin ajaa sls-komennot uudelleen ja sain seuraavanlaisen virheilmoituksen:

![image](https://github.com/user-attachments/assets/ca416af6-bc74-4d3a-b849-a5c502cba55b)

Päätin kokeilla käynnistää virtuaalikoneet uudelleen. Siirryin pois t001-koneelta ja ajoin isäntäkoneella komennon

        $ vagrant reload

Otin SSH-yhteyden herrakoneeseen ja ajoin uudelleen state.apply komennon. Tällä kertaa kaikki sujui kuten pitääkin:  

![image](https://github.com/user-attachments/assets/02941b22-ffe2-4b89-a9cc-b1912d4f047c)





## Lähteet

1. Karvinen, Tero 2025, Palvelinten Hallinta. Viitattu 16.04.2025. https://terokarvinen.com/palvelinten-hallinta/
