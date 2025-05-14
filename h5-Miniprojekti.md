# H5 Miniprojekti - Käyttäjienhallintaa Salt:lla

  
## Harjoituksessa käytetyn koneen parametrejä

Kannettavan tietokoneen malli: HP Elitebook 840 G4
Käyttöjärjestelmä: Windows 10 Pro, versio 22H2
Prosessori: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz
Näytönohjain: Inter(R) HD Graphics 620
RAM: 8 gb

----------------------------------------------------------

## Alkuvalmistelut

Aluksi pistin pystyyn testiympäristön käyttäen Vagrant:ia (Vagrantfilen luontia täällä [H2](https://github.com/Tuomasken/Linux-Hallinta/blob/main/h2-Soitto-kotiin.md) ja lisäksi päivitin sitä soveltamalla https://github.com/gianglex/Courses/blob/main/Palvelinten-Hallinta/h4-pkg-file-service.md 
ohjeita relevantein osin). Pystytettyäni testiympäristön ja varmistettuani, että Salt:n herra ja orja -arkkitehtuuri toimi oikein testikoneiden välillä, siirryin tekemään ensimmmäisenä tarvittavia pilareita.

Käyttäjätiedot sisältävä users-pilari:

      users:
        Matti:
           ssh_key: "ssh-rsa AAAA... matti@matinkone"
           shell: "/bin/bash"
           home: "/home/matti"
        Teppo:
           ssh_key: "ssh-rsa AAAA... teppo@hallinto"
           shell: "/bin/bash"
           home: "/home/teppo"

Tärkeimmät tiedot tästä pilarista on käyttäjien nimet, mutta lisäsin sinne myös muita alakohtia (ssh_key suojattua etäyhteyttä varten, oletus shell, home eli kotikansio), jotta voin harjoitella myös niiden alustamista. 

Lisäksi loin top.sls pilarin, joka kartoittaa mitkä pilarit pätevät mihinkin orjakoneisiin. Tässä harjoituksessa tällä ei ole niin paljoa merkitystä, kun testiympäristö on sen verran yksinkertainen, mutta pilarien kartoittaminen nousee tärkeäksi heti kun
ympäristö muuttuu kompleksimmaksi. Top-pilari:

       base:
         '*':
          - users




##Lähteet

1. Karvinen, Tero 2025, Palvelinten Hallinta. Viitattu 14.05.2025. https://terokarvinen.com/palvelinten-hallinta/
2. https://github.com/gianglex/Courses/blob/main/Palvelinten-Hallinta/h4-pkg-file-service.md
3. 
