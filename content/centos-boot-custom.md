Title: Custom boot
Date: 2019-01-29 22:33
Category: IT
Tags: Linux, Custom, boot, grub, plymouth


Pour changer l'image sur le menu grub
-------------------------------------

Choisir une image et remplacer /boot/grub2/splatch.png

Modifier /etc/default/grub

       GRUB_TIMEOUT=5
       GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
       GRUB_DEFAULT=saved
       GRUB_DISABLE_SUBMENU=true
    -> #GRUB_TERMINAL_OUTPUT="console"
       GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
       GRUB_DISABLE_RECOVERY="true"
    -> GRUB_BACKGROUND="/boot/grub2/splash.png"


Regenérer le grub

    grub2-mkconfig > /boot/grub2/grub.cfg


Pour l'animation plymouth
--------------------------

Plymouth utilise les themes situés dans /usr/share/plymouth/themes/

Pour choisir  le thème par défaut, 

    plymouth-set-default-theme -R <nom_du_thee>

L'option -r refabrique le initrd immédiatement pour y inclure le thème.


Plymouth uilise le fichier de configuration /etc/plymouth/plymouth.conf

    [Daemon]
    Theme=<nom_du_theme>

TODO: voir si la commande du dessus met à jour ce fichier !!!



Un thème contient son fichier de desription <nom_du_theme>/<nom_du_theme>.plymouth.

     [Plymouth Theme]
     Name=UN_TRUC
     Description=une doc
     ModuleName=(script|base|charge|two-step)
     
     # La suite de la conf dépend du type de module (ModuleName)
     # Dans le cas de script

     [script]
     ImageDir=/usr/share/plymouth/themes/fc
     ScriptFile=/usr/share/plymouth/themes/fc/fc.script

Thats' all


Voir aussi
----------

* plymouth-core-libs
* plymouth-theme-charge
* plymouth-system-theme
* plymouth-plugin-two-step
* plymouth-plugin-script


