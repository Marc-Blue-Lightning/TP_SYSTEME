# 🛠️ Mise en place d’un Serveur DNS et DHCP sous Linux

Ce guide décrit étape par étape la procédure pour installer et configurer un serveur DNS (BIND9) et un serveur DHCP (ISC-DHCP-SERVER) sur une machine Linux (Ubuntu/Debian).

---

## 📌 Prérequis

- Système Linux (Ubuntu Server ou Debian)
- Accès root ou sudo
- Connexion réseau active
- Adresse IP statique (ex : 192.168.20.1)

---

## 📡 Partie 1 – Configuration du Serveur DHCP

### 1. Installer le serveur DHCP
```bash
sudo apt update
sudo apt install isc-dhcp-server
```

### 2. Configurer le fichier `/etc/dhcp/dhcpd.conf`
```bash
subnet 192.168.20.0 netmask 255.255.255.0 
{
  range 192.168.20.10 192.168.20.100;
  option domain-name-servers 192.168.20.1;
  option domain-name "lightningMarc.sn";
  default-lease-time 600;
  max-lease-time 7200;
}
```

### 3. Définir l’interface d’écoute DHCP
Modifier `/etc/default/isc-dhcp-server` :
```bash
INTERFACESv4="enp0s3"
```

### 4. Redémarrer le service
```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

---

## 🌐 Partie 2 – Configuration du Serveur DNS (BIND9)

### 1. Installer BIND9
```bash
sudo apt install bind9 bind9utils bind9-doc
```

### 2. Déclarer les zones DNS
Modifier `/etc/bind/named.conf.local` :
```bash
zone "lightning.sn" 
{
    type master;
    file "/etc/bind/direct";
};

zone "20.168.192.in-addr.arpa" 
{
    type master;
    file "/etc/bind/inverse";
}
```

### 3. Créer la zone directe `/etc/bind/direct`
```bash
$TTL 604800
@   IN  SOA     lightning.lightningMarc.sn. root.lightning.lightningMarc.sn. (
             2         ; Serial
        604800         ; Refresh
         86400         ; Retry
       2419200         ; Expire
        604800 )       ; Negative Cache TTL

@       IN  NS      lightning.lightningMarc.sn.
lightning IN  A     192.168.20.1
www    IN  CNAME    lightning
```

### 4. Créer la zone inverse `/etc/bind/inverse`
```bash
$TTL 604800
@   IN  SOA     lightning.lightningMarc.sn. root.lightning.lightningMarc.sn. (
             2         ; Serial
        604800         ; Refresh
         86400         ; Retry
       2419200         ; Expire
        604800 )       ; Negative Cache TTL

@          IN  NS      lightning.lightningMarc.sn.
lightning  IN  A       20.168.190.
1          IN  PTR     lightning
```

### 5. Redémarrer BIND9
```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

---

## ✅ Tests

### DHCP
- Sur un poste client, vérifier que l'IP est automatiquement attribuée :
```bash
ip a
```

### DNS

```

---

## 📂 Fichiers modifiés
- `/etc/dhcp/dhcpd.conf`
- `/etc/default/isc-dhcp-server`
- `/etc/bind/named.conf.local`
- `/etc/bind/direct`
- `/etc/bind/inverse`

---

## 🧠 Remarques
- Vérifiez que le pare-feu autorise les ports 53 (DNS) et 67/68 (DHCP).
- Adaptez les IP à votre topologie réseau.
- Utilisez `journalctl -xe` pour diagnostiquer les erreurs.

