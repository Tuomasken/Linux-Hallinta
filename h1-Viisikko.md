# H1 Viisikko


## Harjoituksessa käytetyn koneen parametrejä

- Kannettavan tietokoneen malli: HP Elitebook 840 G4
- Käyttöjärjestelmä: Windows 10 Pro, versio 22H2
- Prosessori: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz
- Näytönohjain: Inter(R) HD Graphics 620
- RAM: 8 gb



## b)

Latasin julkisen avaimen

![image](https://github.com/user-attachments/assets/bec5030d-f17c-43ee-a72f-bb7330a2f957)

Loin Salt-pakettivaraston asennuskonfiguraation

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

Tarkistetaan vielä lopputulema:

![image](https://github.com/user-attachments/assets/38d72fc2-c9e8-49e4-bf80-ac367db0f31f)

### Service

Seuraavaksi service-tilan testaamista komennolla:

    $ sudo salt-call --local -l info state.single service.running apache2

Tämä komento pyrkii lopputulokseen, jossa apache2-demoni on toiminnassa. Testissäni apache2 oli jo pystyssä, joten lopputulos komnennosta oli seuraavanlainen:

![image](https://github.com/user-attachments/assets/da4295ca-07fe-468a-a0ec-acec769d1d64)

### Pkg

Seuraavaksi kokeilin pakettien asentamista, käyttäen komentoa:

    $ sudo salt-call --local -l info state.single pkg.installed tree

Ja lopputulema oli seuraavanlainen:

![image](https://github.com/user-attachments/assets/823ba0f1-8a27-4111-9951-9e309c08f9bc)


### User

Ja viimeiseksi kokeilin uuden käyttäjän luomista komennolla:

    $ sudo salt-call --local -l info state.single user.present salttestaaja

Joka onnistui myös, kuten kuvasta näkyy:

![image](https://github.com/user-attachments/assets/73d30124-bfaf-4dba-9691-e2cc4665dec2)

## d)

Idempotenssi viittaa tilanteeseen, jossa ajettava komento tuottaa saman tuloksen, vaikka se ajettaisiin useita kertoja. 













