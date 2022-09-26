# TP5 - Systèmes de fichiers, partitions et disques

## Exercice 1. Disques et partitions

1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM
2. Vérifiez que ce nouveau disque dur est bien détecté par le système
3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)

```bash
$ sudo fdisk /dev/sdb
```

4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions (pensez à consulter le manuel !)

```bash
$ sudo mkfs -t ext4 /dev/sdb1
$ sudo mkfs -t ntfs /dev/sdb2
```

5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?*

```bash
$ df -T
```

Elle ne fonctionne pas car le système de fichiers n'est pas monté.

6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)

```bash
$ sudo mkdir /data
$ sudo mkdir /win
$ sudo nano /etc/fstab
```

Ajouter les lignes suivantes :

```bash
/dev/sdb1 /data ext4 defaults 0 2
/dev/sdb2 /win ntfs defaults 0 2
``` 

7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration

```bash
$ sudo mount -a
```

8. Montez votre clé USB dans la VM
9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox).

```bash
$ sudo apt install virtualbox-guest-additions-iso
$ sudo mkdir /media/usb
$ sudo mount /dev/sdc1 /media/usb
```

## Exercice 2. Partitionnement LVM

1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab

```bash
$ sudo umount /data
$ sudo umount /win
$ sudo nano /etc/fstab
```
2. Supprimez les deux partitions du disque, et créez une partition unique de type LVM

```bash
$ sudo fdisk /dev/sdb
```

3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay

```bash
$ sudo pvcreate /dev/sdb1
$ sudo pvdisplay
```

4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay

```bash
$ sudo vgcreate vg1 /dev/sdb1
$ sudo vgdisplay
```

5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.

```bash
$ sudo lvcreate -n lvData -l 100%FREE vg1
```

6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.

```bash
$ sudo mkfs -t ext4 /dev/vg1/lvData
$ sudo mkdir /data
$ sudo nano /etc/fstab
```

Ajouter la ligne suivante :

```bash
/dev/vg1/lvData /data ext4 defaults 0 2
```

7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.

```bash
$ sudo fdisk /dev/sdc
$ sudo pvcreate /dev/sdc1
$ sudo pvdisplay
```

8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes

```bash
$ sudo vgextend vg1 /dev/sdc1
$ sudo vgdisplay
```

9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.

```bash
$ sudo lvresize -l +100%FREE /dev/vg1/lvData
$ sudo resize2fs /dev/vg1/lvData
```
