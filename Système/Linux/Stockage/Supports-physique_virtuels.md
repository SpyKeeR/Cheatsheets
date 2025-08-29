# Stockage, partitions, LVM & fichiers — condensé 💿

## Disques & device names
- IDE : `/dev/hda*` ; SCSI/SATA/USB : `/dev/sdX*` ; NVMe : `/dev/nvme0n1p1`.
- Forcer détection nouveau disque (SCSI) : 
	- `udevadm info --query=path --name=sda`
	- `echo "- - -" > /sys/class/scsi_host/hostN/scan` 

## Partition tables
- MBR : 512 octets, boot code (446o), table partitions (64o), limites : 4 partitions, ≤2.2 TiB.  
- GPT : moderne, ~128 partitions par défaut, supporte très grands disques (Zetta).

## fdisk (menu rapide)
- `fdisk -l` → lister partitions.  
- `fdisk /dev/sdb` → menu : `m` aide, `p` print, `n` new, `d` delete, `t` type, `a` toggle boot, `w` write, `q` quit.

## LVM (workflow essentiel)
- Préparer partition type `8E` (LVM) si nécessaire.  
- `pvcreate /dev/sdb1 /dev/sdb2` → initialiser PV.  
- `vgcreate vggroup1 /dev/sdb1 /dev/sdb2` → créer VG.  
- `lvcreate -n LvHome -L 20G vggroup1` → créer LV.  
- Afficher résumé : `pvs`, `vgs`, `lvs`.  
- Infos détaillées : `pvdisplay`, `vgdisplay`, `lvdisplay`.  
- Étendre VG : `vgextend vg-group1 /dev/sdd`.  
- Étendre LV : `lvextend -l +512M /dev/vg-group/lv1` ou `lvextend -L 1G /dev/vg-group/lv2`.  
- Réduire LV : `lvreduce -L 500M /dev/vg-group/lv1` (prudence > Reduce FS be4).  
- Option `-r` peut redimensionner FS automatiquement (si supporté).



