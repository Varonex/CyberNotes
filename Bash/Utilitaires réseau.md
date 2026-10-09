# MAC & IP
| Commande     | Description                                                                                                                          | Options                                                                                  | Descriptions Options                                                                                                                                                                                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `arp`        | Affiche la table ARP. Commande<br>legacy.                                                                                            | `-a`<br>`-d [e]`<br>`-D [a]`<br>`-e`<br>`-f [f]`<br>`-i [i]`<br>`-p`<br>`-s [e]`<br>`-v` | -Affiche tout au format BSD.<br>-Ajoute une entrée.<br>-Utilise le hardware spécifié.<br>-Affiche tout au format linux.<br>-Lit depuis un fichier ou depuis<br>/etc/ethers.<br>-Utilise l'interface spécifiée.<br>-Protocole.<br>-Supprime une entrée.<br>-Verbosité. |
| `ifconfig`   | Configuration d'une interface.<br>Commande legacy.                                                                                   |                                                                                          |                                                                                                                                                                                                                                                                       |
| `ip`         | Commande généraliste (moderne)<br>qui gère tout relatif aux interfaces,<br>au routage, au protocole ARP.                             |                                                                                          |                                                                                                                                                                                                                                                                       |
| `ping [i]`   | Protocole ICMP vers un utilisateur.                                                                                                  |                                                                                          |                                                                                                                                                                                                                                                                       |
| `route`      | Configuration de la table de<br>routage.                                                                                             |                                                                                          |                                                                                                                                                                                                                                                                       |
| `traceroute` | Envoi progressif de paquets ICMP<br>vers un destinataire, en<br>incrémentant le TTL pour<br>identifier chaque nœud<br>intermédiaire. |                                                                                          |                                                                                                                                                                                                                                                                       |
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
| Commande     | Description                                                                                                                                     | Options | Descriptions Options |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------- | -------------------- |
| `scp [u]`    | Protocole SCP basé sur SSH<br>pour le transfert de fichiers<br>orienté CLI (utile pour les<br>scripts).                                         |         |                      |
| `sftp [u]`   | Protocole SFTP basé sur FTP<br>passant sur SSH pour le<br>transfert de fichiers orienté<br>instructions (utile pour<br>l'usage humain et GUIs). |         |                      |
| `ssh [u]`    | Protocole SSH héritant de<br>TELNET permettant la<br>connexion à distance.                                                                      |         |                      |
| `ssh-keygen` | Utilitaire pour générer des<br>couples (pub, priv) de clés<br>SSH.                                                                              |         |                      |
# Firewall
| Commande | Description | Options | Descriptions Options |
| -------- | ----------- | ------- | -------------------- |
| `ufw`    |             |         |                      |
