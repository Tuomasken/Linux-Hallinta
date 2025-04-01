# H1 Viisikko


## Harjoituksessa käytetyn koneen parametrejä

- Kannettavan tietokoneen malli: HP Elitebook 840 G4
- Käyttöjärjestelmä: Windows 10 Pro, versio 22H2
- Prosessori: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz
- Näytönohjain: Inter(R) HD Graphics 620
- RAM: 8 gb


## x) 

### Karvinen 2021, Run Salt command locally (https://terokarvinen.com/2021/salt-run-command-locally/)

- Tässä artikkelissa Karvinen käy läpi Salt tilafunktioiden testaamista paikallisesti.
- Tärkeimmät tilafunktiot: pkg, file, service, user ja cmd.
- Huom! Komento "$ sudo salt-call --local sys.state_doc" ohjeita varten.

### Karvinen 2018, Salt Quickstart (https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/)

- Tässä artikkelissa Karvinen käy läpi Salt Masterin ja Minionin asentamista ja alkukonfigurointia.
- Master ja Minion välinen liikenne tapahtuu porttien 4505 ja 4506 kautta, eli Master-koneelle pitää tehdä vastaavat reiät palomuuriin.

### Karvinen 2006, Raportin kijroittaminen (https://terokarvinen.com/2006/06/04/raportin-kirjoittaminen-4/)

- Tässä artikkelissa Karvinen käy läpi, mitä kurssin tehtävien raporttien tulee sisältää. Nämä periaatteet on kuitenkin yleistettävissä muihinkin tietoteknisiin raportteihin.
- Raportin sisällön tulee olla: toistettavissa, täsmällinen, helppolukuinen sekä sisältää kaikki käytetyt lähteet.
- Pahoja virheitä raportin kirjoittamisessa ovat plagiointi ja sepittäminen.

### WMWare Inc 2025, Salt Install Guide: Linux (DEB) (https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html)

- Tässä artikkelissa käydään läpi, kuinka asentaa Salt-paketit Debian-pohjaisiin käyttöjärjestelmiin.
- Lataus tapahtuu julkisen ja yksityisen avainparia käyttäen.


## b)

Aluksi loin avainrenkaalle kansion polkuun /etc/apt/keyrings.

Tämän jälkeen latasin Salt Projektin julkisen avaimen, jotta saan luotua siihen yhteyden:

![image](https://github.com/user-attachments/assets/bec5030d-f17c-43ee-a72f-bb7330a2f957)

Loin Salt-pakettivaraston asennuskonfiguraation:

![image](https://github.com/user-attachments/assets/fa87f372-d7d6-474f-9de0-00e33abe0f30)

Asensin Salt masterin ja minionin komennolla:

    $ sudo apt-get install salt-master salt-minion

## c)

### Cmd

Testasin aluksi lokaalisti Salt:n toimintaa ajamalla komennon:

    $ sudo salt-call --local cmd.run 'whoami; ls; pwd'

Komennolla salt-call ajetaan minionin toimintoja, ja argumentti --local poistaa yhteydenoton masteriin. Komento toimi, kuten seuraavasta kuvasta näkyy:

![image](https://github.com/user-attachments/assets/40ada73b-2665-48e5-8362-0a1d88d4f7e5)

### File

Tämän jälkeen siirryin testaamaan muita tilafunktioita. Ensiksi file komennolla:

    $ sudo salt-call --local -l info state.single file.managed /tmp/salttesti1 contents="testi 01.04.2025"

Komennon lisäparametrit "info" antaa lisäinformaatiota komennon lopputuloksesta ja state.single viittaa siihen, kuinka ajetaan vain yksittäinen salt-tila. file.managed viittaa siihen, mikä on toivottu lopputulos;
tässä tapauksessa lopputuloksen pitäisi sisältää "salttesti1" nimisen tekstitiedoston, jossa on sisältönä "testi 01.04.2025".

Komento onnistui, ja lisäinformaatiosta käy ilmi, että muutoksia tapahtui, eli uusi tiedosto sisältöineen luotiin:

![image](https://github.com/user-attachments/assets/d0ed1d00-2c5c-44cb-b0c7-62b03b1570e1)

Tarkistin vielä lopputuleman:

![image](https://github.com/user-attachments/assets/38d72fc2-c9e8-49e4-bf80-ac367db0f31f)

### Service

Seuraavaksi service-tilan testaamista komennolla:

    $ sudo salt-call --local -l info state.single service.running apache2

Tämä komento pyrkii lopputulokseen, jossa apache2-demoni on toiminnassa. Testissäni apache2 oli jo pystyssä, joten lopputulos komennosta oli seuraavanlainen:

![image](https://github.com/user-attachments/assets/da4295ca-07fe-468a-a0ec-acec769d1d64)

### Pkg

Seuraavaksi kokeilin pakettien asentamista, käyttäen komentoa:

    $ sudo salt-call --local -l info state.single pkg.installed tree

Ja lopputulema oli seuraavanlainen:

![image](https://github.com/user-attachments/assets/823ba0f1-8a27-4111-9951-9e309c08f9bc)

Kuten kuvasta voi päätellä, komento onnistui ja tree niminen paketti asennettiin. Lisäinformaatiosta myös selviää, että tree-paketista ei ollut aiempaa versiota asennettuna, sekä asennetun paketin versio.

### User

Ja viimeiseksi kokeilin uuden käyttäjän luomista komennolla:

    $ sudo salt-call --local -l info state.single user.present salttestaaja

Joka onnistui myös:

![image](https://github.com/user-attachments/assets/73d30124-bfaf-4dba-9691-e2cc4665dec2)

Lisäinformaatiossa kerrotaan myös mm. mihin ryhmään tämä uusi käyttäjä lisättiin (tässä tapauksessa käyttäjä on vain omassa ryhmässään) ja kotikansion polun. Huomioitavaa on, että käyttäjälle ei ole asetettu salasanaa tämän komennon yhteydessä. Jos haluaa mahdollistaa salasanalla kirjautumisen käyttäjälle, niin salasana täytyy erikseen määrittää.

## d)

Idempotenssi viittaa tilanteeseen, jossa ajettava komento tuottaa saman tuloksen, vaikka se ajettaisiin useita kertoja. Salt-komennoissa ilmoitetaan toivottu lopputulos, ja kun komento ajetaan, salt vertaa nykyistä tilaa tavoitetilaan. Jos tavoitetila on jo voimassa, komento ei tee muutoksia.

Esimerkkinä tästä aiemmin käytetty komento 

        $ sudo salt-call --local -l info state.single pkg.installed tree

Kun komento ajettiin ensimmäisen kerran, kyseinen paketti "tree" asennettiin (kuva ylempänä). Nyt kun ajan saman komennon uudelleen, saan lopputuloksen:

![image](https://github.com/user-attachments/assets/992696c4-280c-4953-9029-1d2c993cc13e)

Ja kuten komennon lopputuloksen lisäinformaatiosta käy ilmi, salt-komento tunnistaa, että kyseinen paketti on jo asennettuna. Ja muutoksia ei tehty, mikä näkyy kohdassa "Changes".



## Lähteet

1. Karvinen, Tero 2025, Palvelinten Hallinta. Viitattu 01.04.2025. https://terokarvinen.com/palvelinten-hallinta/
2. Karvinen, Tero 2021, Run Salt command locally. Viitattu 01.04.2025. https://terokarvinen.com/2021/salt-run-command-locally/
3. Karvinen, Tero 2018, Salt Quickstart. Viitattu 01.04.2025. https://terokarvinen.com/2018/03/28/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/
4. Karvinen, Tero 2006, Raportin Kirjoittaminen. Viitattu 01.04.2025. https://terokarvinen.com/2006/06/04/raportin-kirjoittaminen-4/
5. WMWare Inc 2025, Salt Install Guide: Linux (DEB). Viitattu 01.04.2025. https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html










