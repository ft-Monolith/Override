# OverRide

Projet de sécurité binaire du cursus 42. Le sujet fournit une machine virtuelle
contenant dix binaires `setuid`, un par niveau. Chaque binaire présente une
vulnérabilité (ou un mécanisme à rétro-ingénierer) qu'il faut exploiter pour
obtenir les droits du niveau suivant et récupérer son mot de passe.

Ce dépôt rassemble le travail d'analyse pour les dix niveaux : le code source
reconstruit de chaque binaire, une explication pas à pas de l'exploitation, et
le mot de passe obtenu.

## Structure du dépôt

```
levelXX/
├── flag                        mot de passe obtenu en résolvant le niveau
├── source                      code C reconstruit à partir du binaire
└── Ressources/
    └── walkthrough.md          analyse et exploitation détaillées
```

Le niveau 06 contient en plus `Ressources/serial.py`, le keygen utilisé pour
calculer le serial attendu par le binaire.

## Contenu des niveaux

| Niveau | Technique |
| ------ | --------- |
| 00 | Mot de passe codé en dur, lu dans le désassemblage |
| 01 | Buffer overflow et ret2libc vers `system("/bin/sh")` |
| 02 | Format string : fuite de la stack pour lire le mot de passe chargé par le binaire |
| 03 | Rétro-ingénierie d'un déchiffrement XOR pour retrouver la valeur attendue |
| 04 | Buffer overflow via `gets()` et shellcode `open`/`read`/`write` contournant le filtre `ptrace` sur `execve` |
| 05 | Format string : écrasement de l'entrée GOT de `exit()` vers un shellcode |
| 06 | Reconstruction de l'algorithme de hachage du login (keygen) |
| 07 | Accès tableau hors-bornes pour écraser l'adresse de retour de `main()` |
| 08 | Binaire SUID détourné via un lien symbolique pour copier un fichier protégé |
| 09 | Buffer overflow dans une structure, redirection vers une fonction `secret_backdoor()` non appelée |

## Utilisation

Le dépôt est de la documentation : il n'y a rien à compiler pour reproduire les
exploits, ceux-ci se déroulent sur la machine virtuelle du sujet.

1. Démarrer la machine virtuelle fournie par le sujet et s'y connecter en
   `level00` avec les identifiants du sujet.
2. Suivre `level00/Ressources/walkthrough.md` pour exploiter
   `/home/users/level00/level00` et obtenir un shell privilégié.
3. Lire le mot de passe du niveau suivant, puis basculer dessus :

   ```sh
   cat /home/users/level01/.pass
   su level01
   ```

4. Répéter pour chaque niveau. Les mots de passe attendus à chaque étape sont
   ceux stockés dans les fichiers `levelXX/flag`.

Les walkthroughs s'appuient sur les outils présents sur la VM : `gdb`,
`objdump`, `strace` et `python` pour la génération des payloads.

## Sources reconstruites

Les fichiers `levelXX/source` sont des reconstructions en C, écrites à partir du
désassemblage et du décompilé de chaque binaire. Elles documentent le
comportement observé et servent de support de lecture aux walkthroughs.

Elles compilent avec `gcc`, mais reproduire l'exploitation à partir d'une
recompilation n'est pas garanti : les binaires d'origine sont en 32 bits et
l'agencement exact de la pile, les adresses et les protections dépendent du
compilateur et de l'environnement.

```sh
gcc -m32 -w -o level00 level00/source
```

## Note

Les techniques décrites ici s'appliquent à un environnement volontairement
vulnérable, fourni dans un cadre pédagogique. Elles n'ont pas vocation à être
utilisées ailleurs.
