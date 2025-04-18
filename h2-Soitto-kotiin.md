# H2 Soitto kotiin


## Harjoituksessa käytetyn koneen parametrejä

- Kannettavan tietokoneen malli: HP Elitebook 840 G4
- Käyttöjärjestelmä: Windows 10 Pro, versio 22H2
- Prosessori: Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz 2.71 GHz
- Näytönohjain: Inter(R) HD Graphics 620
- RAM: 8 gb

------------------------------------------------------------------------

## x) 

### Karvinen 2021: Two Machine Virtual Network With Debian 11 Bullseye and Vagrant (https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/)

- Tässä artikkelissa Karvinen käy läpi, miten luoda useamman virtuaalikoneen verkon Vagrantin avulla.
- Vagrantfile avulla voimme määrittää minkälaisella kokoonpanolla ja alkutoimilla haluamme pystyttää virtuaalikoneet.
- Kun Vagrantfile on määritelty, komento "vagrant up" luo koneet ja "vagrant destroy" tuhoaa ne.

### Karvinen 2018: Salt Quickstart (https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20quickstart%20salt%20stack%20master%20and%20slave%20on%20ubuntu%20linux)

- Tässä artikkelissa Karvinen käy läpi, kuinka luodaan herra-orja yhteys koneiden välille käyttäen Salt-ohjelmaa.
- Huom. Salt pitää nykyään asentaa käyttäen ohjeita, jotka saa osoitteesta https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html

### Karvinen 2023: Salt Vagrant - automatically provision one master and two slaves (https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file)

- Artikkelin kohdissa "infra as code" ja "your wishes as a text file" Karvinen käy läpi miten koota orjilla ajettava koodi tekstitiedostoihin ja niiden käyttö.

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

Myös SSH-yhteys isäntäkoneelta onnistui:

![image](https://github.com/user-attachments/assets/5b74fd6e-5684-4020-ad1d-8672bc9c14e5)


---------------------------------------------------------------------------------------

## c) 

Seuraavaksi lähdin luomaan useamman virtuaalikoneen verkkoa Vagrantilla, käyttäen ohjeita osoitteesta https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/ .

Aloitin luomalla tehtävälle oman kansion ja siirtymällä sinne komennoilla:

        mkdir twohost
        cd twohost

Ja kohdekansiossa loin vagrantfile-tiedoston ja avasin sen notepadilla käyttäen komentoja:

        vagrant init debian/bookworm64
        notepad Vagrantfile

Kopioin sinne osoitteesta https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/ config-tiedoston sisällön, muuttaen kuitenkin käytettävän boxin bullseye64 -> bookworm64.

        # -*- mode: ruby -*-
    # vi: set ft=ruby :
    # Copyright 2019-2021 Tero Karvinen http://TeroKarvinen.com

    $tscript = <<TSCRIPT
    set -o verbose
    apt-get update
    apt-get -y install tree
    echo "Done - set up test environment - https://terokarvinen.com/search/?q=vagrant"
    TSCRIPT

    Vagrant.configure("2") do |config|
	config.vm.synced_folder ".", "/vagrant", disabled: true
	config.vm.synced_folder "shared/", "/home/vagrant/shared", create: true
	config.vm.provision "shell", inline: $tscript
	config.vm.box = "debian/bookworm64"

	config.vm.define "t001" do |t001|
		t001.vm.hostname = "t001"
		t001.vm.network "private_network", ip: "192.168.88.101"
	end

	config.vm.define "t002", primary: true do |t002|
		t002.vm.hostname = "t002"
		t002.vm.network "private_network", ip: "192.168.88.102"
	end
	
    end

Tallensin muutokset tiedostoon ja ajoin komennon:

        vagrant up

Tämä komento asensi kaksi uutta virtuaalikonetta; t001 ja t002, kuten Vagrantfile-tiedostossa määriteltiin. Lisäksi se asensi kyseisille koneille tree-ohjelman (ja ajoi echo-komennon), mikä oli myös määritelty asennettavaksi kyseisessä tiedostossa:

![image](https://github.com/user-attachments/assets/4dac4d73-1f07-40bc-89ed-419ed6e19d2c)

Seuraavaksi kokeilin ottaa ssh-yhteyden vuorollaan koneisiin (aiemmin käytetty komento vagrant ssh "kohteen nimi") ja pingata niillä toisiaansa:

![image](https://github.com/user-attachments/assets/0c4ad43a-c3df-474a-9afd-d54d04b09b2e)
![image](https://github.com/user-attachments/assets/1715a459-0c7b-402b-b5ec-d27fe66fdf83)

Onnistui!

-------------------------------------------------------------

## d)

Seuraavaksi testasin Saltin herra-orja hierarkiaa kohdassa c) luoduilla virtuaalikoneilla. Aluksi lähdin asentamaan ohjeiden https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html mukaisesti Salt:n molemmille koneille ja tekemään t001:stä herran ja t002:sta orjan.

Alkaessani tekemään tätä vaihetta huomasin kuitenkin ongelman; Virtualbox:n Guestadditions ei toiminut ssh-yhteyden kautta, joten leikepöydän käyttö ei onnistunut. Tämä teki Salt:n asentamisesta hieman hankalampaa, joten lähdin selvittämään saisiko Guestadditions:n toimimaan. Löysin seuraavanlaiset ohjeet (https://skillslane.com/vagrant-virtualbox-guest-additions-issue-resolution/) ja kokeilin niitä.

Tämän jälkeen poistin vielä varmuuden vuoksi olemassa olevat virtuaalikoneet vagrant destroy -komennolla ja kokeilin asentaa ne uudelleen vagrant up -komennolla. Tästä seuraava asennus kesti kuitenkin huomattavasti kauemmin kuin aiemmin ja sisälsi valtavasti enemmän vaiheita. Päättelin, että muutos johtui juuri asennetusta Vagrant:n pluginista. Asennus jumiutui jossain vaiheessa, ja odotettuani noin 10 minuuttia keskeytin virtuaalikoneiden luonnin (ctrl-C) ja poistin vagrant destroy -komennolla sen mitä asennuksessa oli jo luotu. 

Selvitin asiaa netistä, ja löysin potentiaalisen ratkaisun sivulta https://github.com/dotless-de/vagrant-vbguest/issues/333 . Kävin lisäämässä Vagrantfile-tiedostoon kohdan

	if Vagrant.has_plugin?("vagrant-vbguest")
   	config.vbguest.auto_update = false
	end

Ajoin uudelleen Vagrant up -komennon, ja tällä kertaa asennus onnistui ja oli nopeampi (noin 1-2 minuuttia). Otin ssh-yhteyden t001-virtuaalikoneeseen ja testasin leikepöydän toimivuutta. Ei toiminut. Totesin hetken harkittuani, että nopeampi vaan kirjoittaa käsin. Eli siirryin takaisin tehtävän tekoon.

Otin SSH-yhteyden t001-virtuaalikoneeseen ja aluksi asensin curl-toiminnon:

	$ sudo apt-get install curl
 
Seuraavaksi loin avainrenkaille kansion:

	$ mkdir -p /etc/apt/keyrings

Ja latasin Salt:n julkisen avaimen:

	$ curl -fsSL https://packages.broadcom.com/artifactory/api/security/keypair/SaltProjectKey/public | 		sudo tee /etc/apt/keyrings/salt-archive-keyring.pgp

Jonka jälkeen loin apt-repositorion määritystiedoston:

	$ curl -fsSL https://github.com/saltstack/salt-install-guide/releases/latest/download/salt.sources | 		sudo tee /etc/apt/sources.list.d/salt.sources

Tämän jälkeen päivitin metadatan ja asensin salt-masterin sekä otin sen käyttöön:

	$ sudo apt update
 	$ sudo apt-get install salt-master
  	$ sudo systemctl enable salt-master
   	$ sudo systemctl start salt-master

t001-virtuaalikoneen IP määritettiin Vagrantfile-tiedostossa, mutta tarkistin sen kuitenkin vielä komennolla:

	$ hostname -I

![image](https://github.com/user-attachments/assets/b0791ec9-c6e3-404f-9add-ae648348bda2)

Koska master-koneella ei ole palomuuria asennettuna, niin minun ei tarvinnut tehdä siihen reikiä salliakseni herra-orja liikenteen.

Seuraavaksi otin yhteyttä t002-virtuaalikoneeseen ja kävin siinä samat askeleet läpi kuin t001 kohdalla, paitsi salt-master sijaan asensin salt-minion.

Tämän jälkeen kävin määrittämässä orja-koneella mikä on sen herra, käyttäen komentoa:

	$ sudoedit /etc/salt/minion

Jossa lisäsin seuraavat kohdat:

![image](https://github.com/user-attachments/assets/9fef20a9-66ea-4ef7-a127-dc932693128f)

Ja käynnistin salt-minion demonin uudelleen, jotta muutokset tulevat voimaan:

	$ sudo systemctl restart salt-minion.service

Seuraavaksi siirryin takaisin herra-virtuaalikoneelle t001 hyväksymään herra-orja yhteyden t002 kanssa, käyttäen komentoa:

	$ sudo salt-key -A

![image](https://github.com/user-attachments/assets/b02ccf51-2927-489d-a4ac-b92c0e539bf5)

Ja testasin yhteyttä komennolla:

	$ sudo salt 't002' cmd.run 'whoami'
 
![image](https://github.com/user-attachments/assets/27790397-10a5-43a9-95c3-b47722c6aff4)

Orja t002 vastasi, joten yhteys toimii.

----------------------------------------------------

## e)

Kokeilin aluksi file-tilan käyttöä komennolla:

	$ sudo salt -l info 't002' state.single file.managed /tmp/salttesti contents="testi 08.04.2025"

![image](https://github.com/user-attachments/assets/23774fd5-3945-4941-b335-655a1ff19c19)

Seuraavaksi kokeilin luoda uuden käyttäjän orjakoneelle käyttäen komentoa:

	$ sudo salt -l info 't002' state.single user.present salttestaaja

 
![image](https://github.com/user-attachments/assets/404b833e-72cd-48ae-87e7-b7943f1e57c8)


----------------------------------------------------------

## Lähteet

1. Karvinen, Tero 2025, Palvelinten Hallinta. Viitattu 08.04.2025. https://terokarvinen.com/palvelinten-hallinta/
2. Karvinen, Tero 2021, Two Machine Virtual Network With Debian 11 Bullseye and Vagrant. Viitattu 08.04.2025. https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/
3. Karvinen 2018: Salt Quickstart – Salt Stack Master and Slave on Ubuntu Linux. Viitattu 08.04.2025. https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/?fromSearch=salt%20quickstart%20salt%20stack%20master%20and%20slave%20on%20ubuntu%20linux
4. WMWare Inc 2025, Salt Install Guide: Linux (DEB). Viitattu 08.04.2025. https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html
5. Karvinen 2023, Salt Vagrant - automatically provision one master and two slaves. Viitattu 08.04.2025. https://terokarvinen.com/2023/salt-vagrant/#infra-as-code---your-wishes-as-a-text-file
6. Hashcorp, Install Vagrant. Viitattu 08.04.2025. https://developer.hashicorp.com/vagrant/docs/installation
7. Skillslane 2023, [Solved] Vagrant VirtualBox Guest Additions Issue Resolution. Viitattu 08.04.2025.  https://skillslane.com/vagrant-virtualbox-guest-additions-issue-resolution/
8. https://github.com/dotless-de/vagrant-vbguest/issues/333

















