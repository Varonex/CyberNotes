# MAC & IP

## Commandes

| Commande     | Description                                                                                                                                                    | Options                                                                                              | Descriptions Options                                                                                                                                                                                                                                                                                    |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `arp`        | Affiche la table ARP. Commande<br>legacy.                                                                                                                      | `-a`<br>`-d [e]`<br>`-D [a]`<br>`-e`<br>`-f [f]`<br>`-i [i]`<br>`-p`<br>`-s [e]`<br>`-v`             | -Affiche tout au format BSD.<br>-Ajoute une entrée.<br>-Utilise le hardware spécifié.<br>-Affiche tout au format linux.<br>-Lit depuis un fichier ou depuis<br>/etc/ethers.<br>-Utilise l'interface spécifiée.<br>-Protocole.<br>-Supprime une entrée.<br>-Verbosité.                                   |
| `ifconfig`   | Configuration d'une interface.<br>Commande legacy.                                                                                                             |                                                                                                      |                                                                                                                                                                                                                                                                                                         |
| `ip [o]`     | Commande généraliste (moderne)<br>qui gère tout relatif aux interfaces,<br>au routage, au protocole ARP.<br><br>Voir tableau en dessous pour<br>les objets IP. | `-a`<br>`-c`<br>`-d`<br>`-f [f]`<br>`-h`<br>`-N`<br>`-r`<br>`-s`<br>`-t`<br>`-V`<br><br>`-4`<br>`-6` | -All.<br>-Ajout de couleur.<br>-Détails complémentaires.<br>-Famille IP (inet, inet6, ...).<br>-Lisible par les humains.<br>-Affiche le resolver système au<br>lieu de l'IP cible pdt DNS.<br>-Statistiques complémentaires<br>selon le contexte.<br>-Timestamp.<br>-Version.<br><br>- IPv4.<br>- IPv6. |
| `ping [i]`   | Protocole ICMP vers un utilisateur.                                                                                                                            |                                                                                                      |                                                                                                                                                                                                                                                                                                         |
| `route`      | Configuration de la table de<br>routage.                                                                                                                       |                                                                                                      |                                                                                                                                                                                                                                                                                                         |
| `traceroute` | Envoi progressif de paquets ICMP<br>vers un destinataire, en<br>incrémentant le TTL pour<br>identifier chaque nœud<br>intermédiaire.                           |                                                                                                      |                                                                                                                                                                                                                                                                                                         |

## Objets notables de la commande IP

| Objet        | Description                                                      | Sous-objet                                      | Descriptions                                                                                                                              |
| ------------ | ---------------------------------------------------------------- | ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `a[ddress]`  | Gère les composantes liées à l'adresse<br>IP sur les interfaces. | `add [i]`<br>`del [i]`                          | -Ajoute une IP.<br>-Supprime une IP.                                                                                                      |
| `h[elp]`     | Affiche l'aide. **EST UTILISABLE AVEC<br>TOUS LES OBJETS !**     |                                                 |                                                                                                                                           |
| `l[ink]`     | Gère les interfaces et leur état (MAC, ...).                     | `set [l]`                                       | -Gère l'interface.                                                                                                                        |
| `r[oute]`    | Table de routage.                                                | `add [r]`<br>``                                 | -Ajoute une règle.<br>                                                                                                                    |
| `n[eighbor]` | Table ARP.                                                       | `add`<br>`change`<br>`del`<br>`flush`<br>`show` | -Ajoute une association.<br>-Change une association.<br>-Supprime une<br>association.<br>-Supprime la table ARP.<br>-Montre la table ARP. |

# DNS

| Commande       | Description | Options | Descriptions Options |
| -------------- | ----------- | ------- | -------------------- |
| `dig [n]`      |             |         |                      |
| `host [n]`     |             |         |                      |
| `nslookup [n]` |             |         |                      |

# TCP/UDP

| Commande  | Description | Options | Descriptions Options |
| --------- | ----------- | ------- | -------------------- |
| `nc`      |             |         |                      |
| `ncat`    |             |         |                      |
| `netstat` |             |         |                      |
| `ss`      |             |         |                      |
| `openssl` |             |         |                      |
| `tcpdump` |             |         |                      |
| `tshark`  |             |         |                      |

# HTTP

| Commande   | Description                                                                                                     | Options | Descriptions Options |
| ---------- | --------------------------------------------------------------------------------------------------------------- | ------- | -------------------- |
| `curl [u]` | Utilitaire couteau suisse pour<br>lancer des requêtes HTTP(S).<br>Gère aussi d'autres protocoles<br>comme SMTP. |         |                      |
| `wget [u]` | Web-get, permet de faire du<br>HTTP(S) vers des urls, mais<br>aussi du SFTP.                                    |         |                      |

# SSH

| Commande          | Description                                                                                                                                     | Options | Descriptions Options |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------- | -------------------- |
| `scp [u]`         | Protocole SCP basé sur SSH<br>pour le transfert de fichiers<br>orienté CLI (utile pour les<br>scripts).                                         |         |                      |
| `sftp [u]`        | Protocole SFTP basé sur FTP<br>passant sur SSH pour le<br>transfert de fichiers orienté<br>instructions (utile pour<br>l'usage humain et GUIs). |         |                      |
| `ssh [u]`         | Protocole SSH héritant de<br>TELNET permettant la<br>connexion à distance.                                                                      |         |                      |
| `ssh-add`         |                                                                                                                                                 |         |                      |
| `ssh-agent`       |                                                                                                                                                 |         |                      |
| `ssh-copy-id [u]` |                                                                                                                                                 |         |                      |
| `ssh-keygen`      | Utilitaire pour générer des<br>couples (pub, priv) de clés<br>SSH.                                                                              |         |                      |
| `ssh-keyscan [s]` |                                                                                                                                                 |         |                      |

# Firewall

| Commande    | Description | Options | Descriptions Options |
| ----------- | ----------- | ------- | -------------------- |
| `arptables` |             |         |                      |
| `ebtables`  |             |         |                      |
| `iptables`  |             |         |                      |
| `ip6tables` |             |         |                      |
| `nftables`  |             |         |                      |
| `ufw`       |             |         |                      |
