> [!CAUTION]
> Cette page regroupe des techniques d'attaque actives, enfreignant ainsi la loi Godfrain 1988, pouvant mener à des poursuites pénales si utilisées contre un STAD non autorisé préalablement.

"Network Mapper". Outil d'énumération pour reconnaissance réseau.

# Usage

Lancer via :
`nmap [options] ip`
Les 1000 ports les plus fréquents par défaut.

# Options

## Entrée

| Option             | Utilité                          |
| ------------------ | -------------------------------- |
| `-iL`              | Liste d'hôtes depuis un fichier. |
| `-iR [...n]`       | Sélectionner les hôtes `n`.      |
| `--exclude [...n]` | Exclure les hôtes `n`.           |
| `--excludefile`    | Fichier d'exclusion.             |

## Port

| Option            | Utilité                                                                   |
| ----------------- | ------------------------------------------------------------------------- |
| `-p`              | Spécifie les ports.<br>`1`<br>`1,2`<br>`1-26`<br>`U:1,2,T:3,4` : UDP/TCP. |
| `-exclude-ports`  | Exclusion des ports.                                                      |
| `-F`              | Fast scan (plage de ports réduite).                                       |
| `-r`              | Ordre séquentiel de scan.                                                 |
| `--top-ports [n]` | Scanne les `n` ports les plus fréquents.                                  |

## Scan d'hôtes

| Option                                                   | Utilité                                        |
| -------------------------------------------------------- | ---------------------------------------------- |
| `-sn`                                                    | Ping scan (sans port).                         |
| `-Pn`                                                    | Désactive la découverte d'hôtes.               |
| `-PS [...ports]`<br>`-PA [...ports]`<br>`-PU [...ports]` | TCP SYN<br>TCP ACK<br>UDP                      |
| `-PE`<br>`-PP`<br>`-PM`                                  | ICMP Echo.<br>ICMP Timestamp.<br>ICML Netmask. |
| `-n`                                                     | Aucune résolution DNS.                         |
| `-R`                                                     | Toujours résolution DNS.                       |

## Scan de ports

| Option                           | Utilité                                                                                          |
| -------------------------------- | ------------------------------------------------------------------------------------------------ |
| `-sS`<br>`-sT`<br>`-sA`<br>`-sU` | Scan TCP/SYN.<br>Scan TCP/Connect.<br>Scan TCP/ACK.<br>Scan UDP. Aucune réponse = ouvert/filtré. |
| `-sN`<br>`-sF`<br>`-sX`          | TCP NULL.<br>TCP FIN.<br>TCP Xmas (bypass pare-feux).                                            |
| `-sO`                            | Protocoles IP supportés.                                                                         |
| `-sI [zombie host[:port]]`       | Scan ultra discret via hôtes zombies.                                                            |

## Versioning

| Option | Utilité                               |
| ------ | ------------------------------------- |
| `-sV`  | Donne les versions.                   |
| `-O`   | Détection OS.                         |
| `-A`   | Active `O`, `sV`, tracert et scripts. |

## Performance

| Option    | Utilité                            |
| --------- | ---------------------------------- |
| `-T[0-5]` | Performance. 0 = lent, 5 = rapide. |

## Contournement

| Option                  | Utilité                    |
| ----------------------- | -------------------------- |
| `--badsum`              | Envoie des paquets bugués. |
| `-f [v]`<br>`--mtu [v]` | Fragmentation des paquets. |
| `-S [IP]`               | IP spoofing.               |
| `--ttl [v]`             | Fixe le Time-To-Live.      |
| `--spoof-mac [mac]`     | MAC spoofing.              |

## Sortie

| Opton                                            | Utilité                                                                                                                                                                     |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-oN [f]`<br>`-oX [f]`<br>`-oS [f]`<br>`-oG [f]` | Sortie dans un fichier textuel.<br>Sortie d'un fichier xml.<br>Sortie d'un fichier "script kiddie" (juste drôle mdr)<br>Sortie d'un fichier grepable (utilisable avec grep) |
| `-oA [base]`                                     | Sortie dans les formats `.nmap`, `.xml` et `.grepable`.                                                                                                                     |
| `--append-output`                                | Append le contenu dans les fichiers au lieu d'overwrite.                                                                                                                    |
| `-v[...v]`                                       | Niveau de verbosité.                                                                                                                                                        |
| `-d[...d]`                                       | Niveau de debugging.                                                                                                                                                        |
| `--reason`                                       | Donne la raison de pourquoi un port est dans l'état décrit.                                                                                                                 |
| `--open`                                         | Liste uniquement les ports ouverts.                                                                                                                                         |
| `--iflist`                                       | Donne les interfaces de l'hôte.                                                                                                                                             |

## Divers

| Option | Utilité           |
| ------ | ----------------- |
| `-6`   | Active scan IPv6. |
