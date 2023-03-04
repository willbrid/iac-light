# Installation de virtualbox 6.1 et vagrant 2.3
Pour démarrer avec notre premier serveur virtuel local, nous devons télécharger et installer Vagrant et VirtualBox, et configurer un simple Vagrantfile, qui décrira le serveur virtuel.
<br><br>
Nous considérons ubuntu Desktop 20.04 comme notre système de base via lequel nous installerons virtualbox et vagrant.

- Installation de virtualbox

Ajoutons la ligne suivante à notre **/etc/apt/sources.list**
```
deb [arch=amd64 signed-by=/usr/share/keyrings/oracle-virtualbox-2016.gpg] https://download.virtualbox.org/virtualbox/debian focal contrib
```

Ajoutons la clé publique Oracle de vérification des signatures :
```
wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc | sudo gpg --dearmor --yes --output /usr/share/keyrings/oracle-virtualbox-2016.gpg
```

Installons virtualbox 6.1
```
sudo apt-get update
sudo apt-get install virtualbox virtualbox-dkms virtualbox-guest-additions-iso
```

- Installation de vagrant
```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

```
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

```
sudo apt update && sudo apt install vagrant
```

Enfin nous ajoutons le plugin vagrant-vbguest de vagrant
```
vagrant plugin install vagrant-vbguest
```

Nous pouvons ajouter une box vagrant pour rocky linux 8 :
```
vagrant box add willbrid/rockylinux8
```

Nous pouvons faire un test pour vérifier nos installations :
```
mkdir ~/ansible-test
cd ansible-test
```

```
vagrant init willbrid/rockylinux8
```

Un fichier Vagrantfile sera créé. Ensuite nous lançons notre vm contenant les configurations par défaut définies dans le fichier Vagrantfile :
```
vagrant up
```

<br>

**Sources**: 
- [Virtualbox](https://www.virtualbox.org/wiki/Linux_Downloads)
- [Vagrant](https://www.vagrantup.com/downloads)