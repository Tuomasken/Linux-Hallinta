# H2 Soitto kotiin


## Harjoituksessa käytetyn koneen parametrejä

- Kannettavan tietokoneen malli: HP Elitebook 840 G4
- Käyttöjärjestelmä: Windows 10 Pro, versio 22H2
- Prosessori: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz
- Näytönohjain: Inter(R) HD Graphics 620
- RAM: 8 gb

------------------------------------------------------------------------

## x) 

------------------------------------------------------------------------

## a) 

Aluksi lähdin asentamaan Vagrant-ohjelmaa isäntäkoneelle käyttäen Vagrantin omia ohjeita (https://developer.hashicorp.com/vagrant/docs/installation).
Latasin Vagrantista version AMD64 2.4.3 (https://developer.hashicorp.com/vagrant/install) ja ajoin asennusohjelman, joka toimi ongelmitta.

Tämän jälkeen siirryin isäntäkoneeni komentokehotteeseen ja kokeilin onko Vagrant asennettu komennolla:

    vagrant -v

Tämä komento palauttaa asennetun vagrantin version:

![image](https://github.com/user-attachments/assets/2ecafdc7-fefa-4cbb-a235-f6751f0a7102)

-----------------------------------------------------------------------

## b)

Tässä osiossa loin Vagrantilla uuden linux-virtuaalikoneen Virtualbox ympäristöön, käyttäen luennolla läpikäytyä komentoa, joka alustaa käytettävän virtuaalikonemallin:

        vagrant init debian/bookworm64

Komennon ajamisessa kesti pieni hetki, mutta mitään indikaattoria edes prosessin alkamisesta ei tullut, joten piti odottaa kärsivällisesti kunnes komento meni läpi. Tässä kesti kuitenkin alle minuutti.
Lopputulema:

![image](https://github.com/user-attachments/assets/daff2788-641b-4c0e-bfdc-b41b80b85c51)

Kehotteen mukaisesti kokeilin seuraavaksi luoda virtuaalikoneen komennolla:

        vagrant up

Jonka lopputulema näytti tältä:

![image](https://github.com/user-attachments/assets/59a2cf4c-7711-4ada-954c-8799f11342c2)
![image](https://github.com/user-attachments/assets/c82142d1-46ab-4c4c-a3ca-265d029efac3)

Tulosteessa on paljon mielenkiintoista informaatiota. Siitä selviää, että Vagrant osasi automaattisesti tehdä Virtualbox:n kautta tehdä tämän virtuaalikoneen asennuksen. Lisäksi Vagrant automaattisesti loi uuden avaimen SSH-yhteyttä varten.

Lopuksi vielä tarkastin Virtualbox:sta, että löytyykö uusi VM sieltä:

![image](https://github.com/user-attachments/assets/a424aeb4-2731-4e72-a27e-9fedb6caed07)

---------------------------------------------------------------------------------------











