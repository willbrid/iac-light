# Installation et configuration de vault [PART4]

Nous installerons et configurerons **vault** sur un serveur Rocky Linux 8.

1. **Installation**

```
sudo yum install -y yum-utils
```

```
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```

```
sudo yum -y install vault
```

Pour vérifier si vault est bien installé, nous exécutons :

```
vault -h
```

2. **Configuration**

Nous créeons d'abord le repertoire de configuration de **vault** :

```
sudo mkdir -p /etc/vault.d/
```

S'il existe déjà, alors nous pouvons ignorer cette étape.

Nous éditons le fichier de configuration **vault.hcl**

```
sudo vim /etc/vault.d/vault.hcl
```

```
ui = true

#storage "file" {
#  path = "/opt/vault/data"
#}

storage "consul" {
  address = "127.0.0.1:8500"
  path    = "vault/"
}

# HTTPS listener
listener "tcp" {
  address       = "0.0.0.0:8443"
  tls_cert_file = "/etc/letsencrypt/live/notredomaine/fullchain.pem"
  tls_key_file  = "/etc/letsencrypt/live/notredomaine/privkey.pem"
}
```

Nous créeons le service **vault.service**

```
sudo vim /etc/systemd/system/vault.service
```

```
[Unit]
Description=Vault
Documentation=https://www.vault.io/

[Service]
ExecStart=/usr/bin/vault server -config=/etc/vault.d/vault.hcl
ExecReload=/bin/kill -HUP $MAINPID
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

L'argument **server** permet de configurer un serveur **vault** avec son option **-config** qui permet de préciser le fichier de configuration.

```
sudo systemctl daemon-reload
sudo systemctl enable vault
sudo systemctl start vault
```

Nous pouvons vérifier le statut de ce service :

```
sudo systemctl status vault
```

Nous pouvons aussi consulter les logs de **vault** via **journalctl**

```
sudo journalctl -f -u vault
```

Nous configurons de manière permanente la variable d'environnement **VAULT_ADDR**

```
export VAULT_ADDR="https://notredomaine:8443"
echo "export VAULT_ADDR=https://notredomaine:8443" >> ~/.bashrc
```

Nous initialisons un cluster vault

```
vault operator init
```

Si tout se passe bien, aucune erreur ne sera affichée.

<br>

Nous pouvons réinitialiser la configuration de notre cluster vault :
```
sudo systemctl stop vault
consul kv delete -recurse vault/
sudo systemctl start vault
vault operator init
```

L'argument **kv** de la commande **consul** permet d'nteragir avec son magasin **clé-valeur**. Avec son argument **delete** et son option **-recurse** (de l'argument **delete**), il permet de supprimer de manière récursive un magasin **clé-valeur** précisé en argument (**vault/**).

<br>

Après avoir éxécuté la commande **vault operator init**, nous obtenons dans le résultat le message : 

```
Vault does not store the generated root key. Without at least 3 keys to
reconstruct the root key, Vault will remain permanently sealed!

# Vault ne stocke pas la clé racine générée. Sans au moins 3 clés pour
reconstruire la clé racine, Vault restera définitivement scellé !
```

Nous devons desceller au moins 3 clés (issus du résultat de la commande **vault operator init**) avec la commande :

```
vault operator unseal valeurCle
```