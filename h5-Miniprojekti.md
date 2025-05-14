# H5 Miniprojekti - Käyttäjienhallintaa Salt:lla

  
## Harjoituksessa käytetyn koneen parametrejä

- Kannettavan tietokoneen malli: HP Elitebook 840 G4
- Käyttöjärjestelmä: Windows 10 Pro, versio 22H2
- Prosessori: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz
- Näytönohjain: Inter(R) HD Graphics 620
- RAM: 8 gb

- Virtuaalikoneet OS: Debian/Bookworm64

----------------------------------------------------------

## Alkuvalmistelut

Aluksi pistin pystyyn testiympäristön käyttäen Vagrant:ia (Vagrantfilen luontia täällä [H2](https://github.com/Tuomasken/Linux-Hallinta/blob/main/h2-Soitto-kotiin.md) ja lisäksi päivitin sitä soveltamalla https://github.com/gianglex/Courses/blob/main/Palvelinten-Hallinta/h4-pkg-file-service.md 
ohjeita relevantein osin). Pystytettyäni testiympäristön ja varmistettuani, että Salt:n herra ja orja -arkkitehtuuri toimi oikein testikoneiden välillä, siirryin tekemään projektissa tarvittavia pilareita (tukena käytin https://docs.saltproject.io/en/latest/ref/configuration/master.html#pillar-configuration-master).

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

Tärkeimmät tiedot tästä pilarista ovat käyttäjien nimet, mutta lisäsin sinne myös muita alakohtia (ssh_key suojattua etäyhteyttä varten, oletus shell, home eli kotikansio), jotta voin hieman harjoitella myös niiden alustamista. 

Lisäksi loin top.sls pilarin, joka kartoittaa mitkä pilarit pätevät mihinkin orjakoneisiin. Tässä harjoituksessa tällä ei ole niin paljoa merkitystä, kun testiympäristö on sen verran yksinkertainen, mutta pilarien kartoittaminen nousee tärkeäksi heti kun ympäristö muuttuu kompleksimmaksi. Top-pilari:

       base:
         '*':
          - users

## Käyttäjienluonti

Lähdin kirjoittamaan Salt:n tilaa käyttäjienluonnille. Halusin että se hakee aiemmassa vaiheessa luodusta users-pilarista tarvittavat tiedot kaikista käyttäjistä mitä orjakoneella pitäisi olla ja luo sen pohjalta tarvittavat käyttäjät, jos niitä puuttuu. Suunnittelun tukena käytin Salt Project:n dokumentaatiota laajemmin (esim. https://docs.saltproject.io/salt/user-guide/en/latest/topics/jinja.html) sekä ChatGPT:tä.

Lopullinen tilafunktio (polku: /srv/salt/UserManagement/create_users.sls) näytti tältä: 

    {% for user, info in pillar.get('users', {}).items() %}

    {{ user }}_user:
      user.present:
        - name: {{ user }}
        - shell: {{ info.get('shell', '/bin/bash') }}
        - home: {{ info.get('home', '/home/' ~ user) }}
        - createhome: True
    
    {{ user }}_ssh_dir:
      file.directory:
        - name: {{ info.get('home', '/home/' ~ user) }}/.ssh
        - user: {{ user }}
        - group: {{ user }}
        - mode: 700
    
    {{ user }}_authorized_keys:
      file.managed:
        - name: {{ info.get('home', '/home/' ~ user) }}/.ssh/authorized_keys
        - user: {{ user }}
        - group: {{ user }}
        - mode: 600
        - contents: |
            {{ info['ssh_key'] }}

    {% endfor %}

Ja kun ajoin komennon

    $ sudo salt 'orja1' state.apply UserManagement.create_users


Niin komento onnistuneesti loi käyttäjät, loi niille kotihakemiston ja loi SSH-credentiaalit etäyhteyden muodostamista varten:

![image](https://github.com/user-attachments/assets/dbc0cb4e-209e-4867-be52-1da4c8671102)

Kun komennon ajoi uudelleen, niin muutoksia ei tapahtunut. Eli komento on idempotentti:

![image](https://github.com/user-attachments/assets/a7594820-db38-4008-bc51-4a58299c6605)


## Olemassaolevien käyttäjien tarkastelu

Seuraavaksi halusin kirjoittaa toiminnon, jolla voin tarkastella mitä käyttäjiä orjakoneella on ja verrata niitä users-pilarin tietoihin. Tässä kuitenkin aika nopeaa huomasin, että Salt:n "kielet" yaml ja jinja olivat hieman joustamattomia tarkoituksiini. Päädyinkin lopulta kirjoittamaan pythonilla moduulin, joka tekee mitä halusin. Se näytti seuraavanlaiselta:

    import pwd

    def audit_users(min_uid=1000, exclude=None):

    if exclude is None:
        exclude = ['nobody', 'vagrant']

    expected_users = list(__pillar__.get('users', {}).keys())

    actual_users = [
        user.pw_name
        for user in pwd.getpwall()
        if user.pw_uid >= min_uid and user.pw_name not in exclude
    ]

    missing_users = sorted(set(expected_users) - set(actual_users))
    extra_users = sorted(set(actual_users) - set(expected_users))

    return {
        'expected': sorted(expected_users),
        'actual': sorted(actual_users),
        'missing': missing_users,
        'extra': extra_users,
    }

Käytin Pythonin omaa moduulia pwd hakeakseni orjan käyttäjätiedot ja vertasin niitä users-pilarin käyttäjien nimiin. Huomionarvoista moduulista on myös pw_uid kohta ja sen minimiarvo 1000, mikä viittaa vain luotuihin käyttäjiin. Systeemin käyttäjillä (esim. admin, sudo) on uid-arvot alle 1000. Lisäksi asetin erikseen käyttäjän Vagrant exclude-listalle, koska se asennettiin Vagrantilla testiympäristön alustamisen yhteydessä ja on välttämätön niiden hallinnassa.

Kun tämän moduulin ajoi komennolla:

      $ sudo salt 'orja1' check_users.audit_users

Sai tulosteeksi orjakoneen senhetkiset käyttäjät: 

![image](https://github.com/user-attachments/assets/17d1d63a-31e8-4677-adef-913ddc27dba3)

Testasin vielä miten komento näyttää "odottamattomat" käyttäjät, eli ne mitkä eivät ole edustettuna users-pilarissa. Tätä testasin lisäämällä orjakoneelle tarja-käyttäjän:

      $ sudo salt 'orja1' user.add tarja
      $ sudo salt 'orja1' check_users.audit_users

![image](https://github.com/user-attachments/assets/289026b5-2dde-4169-b359-2876f7114d43)


## Ylimääräisten käyttäjien poistaminen

Viimeiseksi työkaluksi suunnittelun toimintoa, joka poistaa käyttäjät orjakoneelta, jotka eivät ole edustettuna users-pilarissa. Tähän kirjoitin taas Salt:n tilan, joka lopulta näytti tältä:

    {% set expected_users = pillar.get('users', {}).keys() | list %}

    {% set current_users = salt['cmd.run']("awk -F: '$3 >= 1000 {print $1}' /etc/passwd").split('\n') %}
    {% set ignored_users = ['nobody', 'vagrant'] %}
    {% set users_to_remove = current_users | difference(expected_users + ignored_users) %}
    
    {% for user in users_to_remove %}
    remove_unexpected_user_{{ user }}:
      user.absent:
        - name: {{ user }}
        - purge: True
    {% endfor %}

Tämä tila hakee orjakoneelta käyttäjätiedot lokaatiosta /etc/passwd, ja vertaa niitä users-pilarin vastaaviin. Käyttäjät jotka eivät ole pilarissa, poistetaan user.absent tilafunktiolla. HUOM! Tässä pitää olla hyvin tarkkana, että ignored_users muuttuja on oikein asetettu! Muuten tilan ajaminen voi poistaa jopa järjestelmän käyttäjiä. Tässä tapauksessa käyttäjät joilla on uid alle 1000 ja Vagrant ovat poissuljettuja.

Tila ajetaan komennolla:

    $ sudo salt 'orja1' state.apply UserManagement.removeextra

![image](https://github.com/user-attachments/assets/149fa5cb-de52-4b53-a53e-e57a20ef8cb0)

Ja tarkistetaan vielä lopputulema moduulilla check_users, mitä käytettiin aiemminkin:

![image](https://github.com/user-attachments/assets/2608d25a-9ad1-4176-a1a7-52fed45c127d)

Kaikki toimii kuten pitääkin.




##Lähteet

1. Karvinen, Tero 2025, Palvelinten Hallinta. Viitattu 14.05.2025. https://terokarvinen.com/palvelinten-hallinta/
2. Giang 2025, H4 Pkg-File-Service. Viitattu 14.05.2025. https://github.com/gianglex/Courses/blob/main/Palvelinten-Hallinta/h4-pkg-file-service.md
3. Salt Project, 2025. Pillar Configuration. Viitattu 14.05.2025. https://docs.saltproject.io/en/latest/ref/configuration/master.html#pillar-configuration-master
4. Salt Project, 2025. Using Jinja with Salt. Viitattu 14.05.2025 https://docs.saltproject.io/salt/user-guide/en/latest/topics/jinja.html
5. ChatGPT-keskustelu. Käyty 14.05.2025. ChatGPT.com
