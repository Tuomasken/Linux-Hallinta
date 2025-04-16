# H2 Infraa koodina

## Harjoituksessa käytetyn koneen parametrejä

- Kannettavan tietokoneen malli: HP Elitebook 840 G4
- Käyttöjärjestelmä: Windows 10 Pro, versio 22H2
- Prosessori: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz
- Näytönohjain: Inter(R) HD Graphics 620
- RAM: 8 gb

------------------------------------------------------------------------

## x) 

### Karvinen 2014: Hello Salt Infra-as-Code (https://terokarvinen.com/2024/hello-salt-infra-as-code/)

- Tässä artikkelissa Karvinen käy läpi miten tehdä sls-tiedostoja, joiden avulla hallita orja-koneita.
- Sls-tiedosto sisältää Salt:n omasta kielestä koostuvaa syntaksia, joilla voidaan ajaa monimutkaisempia komentokokonaisuuksia.

### Salt contributors: Salt overview (https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml)

- Tämä Salt Project:n käyttöoppaan kohdat YAML:sta käsittelevät Salt:n omaa tapaa kirjoittaa sen konfiguraatiotiedostot, ja sen rakennetta.
- YAML koostuu kolmesta eri perustyypistä; Skalaarinen, listamainen ja sanakirjamainen.

--------------------------------------------------------------------------

## a)

Tässä tehtävässä lähdin harjoittelemaan tietokoneiden hallintaa Saltilla käyttäen sls-tiedostoja. Ensiksi kuitenkin vain paikallisesti. Seurasin sivun https://terokarvinen.com/2024/hello-salt-infra-as-code/ ohjeita.

Aluksi loin uuden testimoduulin polkuun /srv/salt/esim1/ ja siirryin sinne.

    $ sudo mkdir -p /srv/salt/esim1/
    $ cd /srv/salt/esim1/

Ja lähdin luomaan sls-tiedostoa komennolla

    $ sudoedit init.sls

Ja kirjoitin sinne syntaksin, joka luo uuden tiedoston käyttäen Saltin file-tilaa:

    /tmp/esimerkki1:
      file.managed









## Lähteet

1. Karvinen, Tero 2025, Palvelinten Hallinta. Viitattu 08.04.2025. https://terokarvinen.com/palvelinten-hallinta/
