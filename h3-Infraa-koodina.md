# H3 Infraa koodina

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

Kokeilin ajaa sls-tiedoston paikallisesti komennolla:

    $ sudo salt-call --local state.apply esim1

Suorittaessani komennon mitään ei kuitenkaan tapahtunut noin minuuttin, jonka jälkeen tuli seuraavanlainen virheilmoitus:

![image](https://github.com/user-attachments/assets/3c5a4976-e672-4c54-a5b2-e2bca0d1d3a5)

Kävin lisäämässä file.managed perään ":", kuten virheilmoituksessa pyydettiin ja kokeilin ajaa komentoa uudelleen. Sain uuden virheilmoituksen:

![image](https://github.com/user-attachments/assets/5c252f34-6bf0-40b1-ab03-3a2ab5232e17)

Tämän pohjilta päätin käydä lukemassa uudelleen Salt:n syntaksirakenteesta osoitteessa https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml

Hoksasin, että en ollut sisentänyt sls-tiedoston sisältöä oikein. Kävin korjaamassa, ja ajoin komennon uudelleen. Tällä kertaa sain uudenlaisen virheilmoituksen:

![image](https://github.com/user-attachments/assets/572fe29b-5738-420a-a607-a947e08182be)

Virheilmoituksen mukaisesti kävin poistamassa file.managed perästä kaksoispisteen. Jonka jälkeen kokeilin uudelleen ajaa tiedostoa, ja tällä kertaa onnistuen:

![image](https://github.com/user-attachments/assets/daf8c1c7-fb43-4c72-adff-12abb7f7f6d2)

Tarkisin vielä ls-komennolla, että onhan tiedosto varmasti luotu:

![image](https://github.com/user-attachments/assets/6af45ff9-96e7-4fce-9356-dee1e4d16b30)


-------------------------------------------------------------------

## b)

Seuraavaksi kokeilin ajaa sls-tiedostoa orja-koneelle t002, käyttäen komentoa:

        $ sudo salt -l info 't002' state.apply esim1

Tässä sain seuraavanlaisen virheilmoituksen:

![image](https://github.com/user-attachments/assets/b7ccb022-13d3-416a-b02d-77bf6e00a9bd)

Virheilmoituksen mukaisesti ensiksi lähdin tarkastamaan salt-master:n tilaa komennolla:

        $ sudo systemctl status salt-master

Tämän kautta näin, että salt-master ei ollut päällä, joten yritin käynnistää sen uudelleen:

![image](https://github.com/user-attachments/assets/fcf464b7-d629-42ba-86ce-e42df28fb0ac)

Kuten kuvasta näkyy, niin sen tila vaihtui aktiiviseksi. Seuraavaksi kokeilin ajaa esim1-moduulia t002-orjalla uudelleen. Tällä kertaa se onnistui: 

![image](https://github.com/user-attachments/assets/4fa08deb-bbc6-4bd6-8f12-8e1a189117b5)

------------------------------------------------------------------

## c)

Tässä osiossa lähdin testaamaan useampaa tilafunktiota käyttävää sls-tiedostoa. Samoin kuin aiemmin, loin uuden moduulin esim2, siirryin sinne ja loin tiedoston init.sls.

        $ sudo mkdir -p /srv/salt/esim2/
        $ cd /srv/salt/esim2/
        $ sudoedit init.sls

Koska aiemmassa esimerkissä kokeilin jo file-tilaa, niin nyt ajattelin testata pkg- ja user-tiloja. Tutustuin Saltin käyttöoppaaseen (https://docs.saltproject.io/salt/user-guide/en/latest/topics/states.html) ja kirjoitin seuraavanlaisen syntaksin.

       install_tree:
         pkg.installed:
           - name: tree

        make_user_testaaja2:
          user.present:
            - name: testaaja2

Kokeilin ajaa moduulin t002-orjalla käyttäen komentoa:

        $ sudo salt -l info 't002' state.apply esim2

Onnistui!

![image](https://github.com/user-attachments/assets/83252257-2de4-4187-9aa7-7beb874f5d63)

Tree ei asentunut, koska olin unohtanut että se oli tullut asennettua orjakoneelle jo aiemmassa harjoituksessa. Mutta tulos kuitenkin osoittaa, että syntaksi kuitenkin toimi (ja samalla tuli osoitettua komennon idempotenttisuus, sillä tree-ohjelmaa ei asennettu uudelleen).

Seuraavaksi varmistin, että komennot olivat toimineet. Ensiksi käyttäjän testaaja2 tarkastelu:

        $ sudo salt -l info 't002' state.single cmd.run 'id testaaja2'

![image](https://github.com/user-attachments/assets/ccfda2c4-6040-42e8-9085-1ed54a7cf9a7)

Kyseinen käyttäjä löytyi, eli esim2-moduuli oli ainakin siltä osin toiminut. Seuraavaksi katson onko tree asennettuna:

        $ sudo salt -l info 't002' state.single pkg.installed tree

![image](https://github.com/user-attachments/assets/53c9d423-2b5d-4f4b-8e30-46f434c3cd48)

Paketti on asennettuna. Seuraavaksi kokeilen esim2-moduulin idempotenttisuutta suorittamalla sen uudelleen.

![image](https://github.com/user-attachments/assets/29a74e2b-209f-472a-9892-ad4a46f3f0a8)

Ajaessa moduulin uudelleen komennot toimivat, mutta eivät tee mitään koska niiden tavoitetila on jo toteutettuna. Eli esim2 on idempotenttinen.

---------------------------------------------------------------------------------------

## Lähteet

1. Karvinen, Tero 2025, Palvelinten Hallinta. Viitattu 16.04.2025. https://terokarvinen.com/palvelinten-hallinta/
2. Karvinen 2014: Hello Salt Infra-as-Code. Viitattu 16.04.2025. https://terokarvinen.com/2024/hello-salt-infra-as-code/
3. Salt Project: Salt overview. Viitattu 16.04.2025. https://docs.saltproject.io/salt/user-guide/en/latest/topics/overview.html#rules-of-yaml
4. Salt Project: Salt states. Viitattu 16.04.2025. https://docs.saltproject.io/salt/user-guide/en/latest/topics/states.html
