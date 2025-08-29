# Linux — Outils & trucs pratiques (condensé) 🧰

## Texte & recherches
- `grep "motif" fichier` ou `commande | grep "motif"`. 
	- -e : recherche plusieurs motifs (un -e par motif)
	- -i : ignore la casse (maj/min)
	- -l : affiche juste les noms de fichiers où le motif est trouvé
	- -n : affiche le numéro des lignes trouvées
	- -v : inverse la recherche → affiche tout sauf les lignes contenant le motif
	- -r : Recherche récursive
 
- `sed` → substitutions : `sed -i 's/ancien/nouveau/g'`
- `cut` → Couper une seule colomne et personnaliser la délimitation.
- `awk` → traitement champs (si besoin).  
- `jq` → parser JSON.

## Archivage & compression (tar)
- `tar cf archive.tar fichiers` → create.  
- `tar xvf archive.tar` → extract.  
- `tar czf archive.tar.gz` → gzip, `cjf` → bzip2, `cJf` → xz.  
- Options `-C` output extract, `-t` afficher contenu, `u` update.

## Transferts & files helpers
- `ncdu` / `ranger` → exploration disque.  
- `duf` → affichage usage disque joli.  
- `ripgrep (rg)` → grep moderne rapide.  
- `fd` → find plus rapide.  
- `fzf` → fuzzy finder pour filtrer outputs.  
- `zoxide` → cd intelligent (mémoire).  
- `exa` → ls amélioré + tree.  
- `mosh` → SSH roaming.  
- `wormhole` → transfert P2P fichier.  
- `termshark` → Wireshark en CLI.   
- `lshw` → list hardware.  
- `ifdata` → info interface claire.  
- `vipe` → utiliser vi sur un pipeline.  
- `unp` → extrait archives selon type.  
- `taskwarrior` → todo CLI.  
- `asciinema rec/play` → enregistrer terminal & replay/export.
- `shred` → supprimer fichiers en plusieurs passes. 

## Utilitaires Docker friendly
- `lazydocker` → interface CLI conviviale.

## Outils de productivité
- `watch` → Active une commande avec autorefresh
- `ts` → Ajouter des horodatages à chaque ligne de sortie (commande de pipeline)
- `fzf`→ Filtrage d'une sortie (?)
- `ripgrep` → grep moderne
- `ncdu` → Explorateur de fichiers
- `exa` → ls avec une arborescence.
- `fd` → Recherche plus rapide