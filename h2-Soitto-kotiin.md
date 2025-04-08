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

Seuraavaksi testasin Saltin herra-orja hierarkiaa kohdassa c) luoduilla virtuaalikoneilla. Aluksi lähdin asentamaan ohjeiden https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/linux-deb.html mukaisesti Salt:n 
molemmille koneille ja tekemään t001:stä herran ja t002:sta orjan.

Alkaessani tekemään tätä vaihetta huomasin kuitenkin ongelman; Virtualbox:n Guestadditions ei toiminut ssh-yhteyden kautta, joten leikepöydän käyttö ei onnistunut. Tämä teki Salt:n asentamisesta hyvin hankalaa, joten lähdin selvittämään saisiko Guestadditions:n toimimaan. Löysin seuraavanlaiset ohjeet (https://skillslane.com/vagrant-virtualbox-guest-additions-issue-resolution/) ja kokeilin niitä.

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

 
















