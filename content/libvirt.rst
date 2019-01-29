****************
Etude de libvirt
****************

:Title: Some doc about libvirt
:Date: 2018-11-05 01:23
:Category: IT
:Tags: Linux, virtualization, libvirt

Installation de libvirt et virt-manager


dnsmasq
=======

libvirt utilise dnsmasq

Exemple de fichier de configuration:

::

	strict-order
	pid-file=/var/run/libvirt/network/default.pid
	except-interface=lo
	bind-dynamic
	interface=virbr0
	dhcp-range=192.168.122.2,192.168.122.254
	dhcp-no-override
	dhcp-leasefile=/var/lib/libvirt/dnsmasq/default.leases
	dhcp-lease-max=253
	dhcp-hostsfile=/var/lib/libvirt/dnsmasq/default.hostsfile
	addn-hosts=/var/lib/libvirt/dnsmasq/default.addnhosts

dnsmasq est lancé par libvirt avec la ligne de commande :

	/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --dhcp-script=/usr/libexec/libvirt_leaseshelper

`/usr/libexec/libvirt_leaseshelper` est un programme binaire

A voir pour unne utilisation pour puppet autonome.

Creation d'un bridge
====================

::

  # Creation du bridge avec NetworkManager
  nmcli connection add type bridge ifname br0
  nmcli connection add type bridge-slave ifname p4p2 master br0
  nmcli connection up bridge-br0
  # Declaration dans libvirt
  cat > br0.xml<<EOF_NET
  <network>
     <name>br0</name>
     <forward mode="bridge" />
     <bridge name="br0" />
  </network>
  EOF_NET
  virsh net-define br0.xml
  virsh net-start br0
  virsh net-autostart br0
  virsh net-list



Installation Centos 7 minimal
=============================

J'utilise le Cd d'installation de CentOS.
Je choisis l'installation minimal.
Le kickstart contient l'installation du group de paquets `@core`
Le réseau est configuré mais pas lancé au démarrage.
Le système utilise env 1GB de dd.

::

	[user@localhost ~]$ df -H
	Sys. de fichiers        Taille Utilisé Dispo Uti% Monté sur
	/dev/mapper/centos-root   4,3G    833M  3,5G  20% /
	devtmpfs                  515M       0  515M   0% /dev
	tmpfs                     522M       0  522M   0% /dev/shm
	tmpfs                     522M    7,0M  515M   2% /run
	tmpfs                     522M       0  522M   0% /sys/fs/cgroup
	/dev/vda1                 521M    101M  420M  20% /boot

Création d'une VM centOS
========================

::

	virt-install --name srv01 --ram 1024  --vcpus=1 --os-variant=rhel7 --boot cdrom,network --disk /home/depots/Virtu/Kvm/srv01.qcow2,size=5 --initrd-inject=/tmp/srv01.ks  --extra-args "ks=file:/my.ks" --location http://mirror.centos.org/centos/7/os/x86_64/

ou

::
   
        sudo  mount -oloop /home/depots/ISO/CentOS-7-x86_64.iso /mnt
	virt-install --name srv01 --ram 1024  --vcpus=1 --os-variant=rhel7 --boot cdrom,network --disk /home/depots/Virtu/Kvm/srv01.qcow2,size=5 --initrd-inject=/tmp/srv01.ks  --extra-args "ks=file:/my.ks" --location  /mnt
    

Annexes
=======


Fichier kickstart
-----------------

:: 

        #version=RHEL7
        # System authorization information
        auth --enableshadow --passalgo=sha512
        
        # Use CDROM installation media
        cdrom
        # Run the Setup Agent on first boot
        firstboot --enable
        ignoredisk --only-use=vda
        # Keyboard layouts
        keyboard --vckeymap=fr --xlayouts='fr (oss)'
        # System language
        lang fr_FR.UTF-8
        
        # Network information
        network  --bootproto=dhcp --device=eth0 --onboot=off --ipv6=auto
        network  --hostname=localhost.localdomain
        # Root password: {tootoor}!
        rootpw --iscrypted $6$0rWj5wdTTvCfY4ak$uTBvgroOoeqHqjO.kvgBN2kyLaDvw7pmTzsY8c3MjA9LLllL3gmo4f5vAEZpdj75Eic06LByjCb0lg.i2K.lT.
        # System timezone
        timezone Europe/Paris --isUtc
        # user passwd is : +install00!
        user --groups=wheel --homedir=/home/user --name=user --password=$6$7C4NqqetkHmanVCv$LqQnmhs9Py8JaYXEOBg6ct47gn/n1VsEBvjmFQLzqjjGDr3V.FpMfWjeARySG07ycPHIb.MYyhIhmUJW.Z4KS0 --iscrypted --gecos="user"
        # System bootloader configuration
        bootloader --location=mbr --boot-drive=vda
        autopart --type=lvm
        # Partition clearing information
        clearpart --none --initlabel 
        
        %packages
        @core
        
        # network is down
        # to launch it, just run sudo ifup eth0
        # set --onboot parameter to on to enable interface on boot.
        # sshd is launched
        
        
        %end




Les hooks dans LIbvirt
======================

TODO: vérifié cette histoire de hooks !!

Créer le fichier dans /etc/libvirt/hooks/qemu
::

  !/bin/bash
  # used some from advanced script to have multiple ports: use an equal number of guest and host ports
  
  
  echo `date` hook/qemu "${1}" "${2}" >>/root/hook.log
  
  
  # Update the following variables to fit your setup
  
  
  ### First VM
  Guest_name=VM_1_NAME
  Guest_ipaddr=192.168.122.4
  Host_port=(  '1234' )
  Guest_port=( '22' )
  
  
  length=$(( ${#Host_port[@]} - 1 ))
  if [ "${1}" = "${Guest_name}" ]; then
  if [ "${2}" = "stopped" ] || [ "${2}" = "reconnect" ]; then
      for i in `seq 0 $length`; do
              echo "kvm-Ho." >>/root/hook.log
              /sbin/iptables -D FORWARD -o virbr0 -d  ${Guest_ipaddr} -j ACCEPT
              /sbin/iptables -t nat -D PREROUTING -p tcp --dport ${Host_port[$i]} -j DNAT --to ${Guest_ipaddr}:${Guest_port[$i]}
      done
  fi
  if [ "${2}" = "start" ] || [ "${2}" = "reconnect" ]; then
      for i in `seq 0 $length`; do
              echo "kvm-Hey." >>/root/hook.log
              /sbin/iptables -I FORWARD -o virbr0 -d  ${Guest_ipaddr} -j ACCEPT
              /sbin/iptables -t nat -I PREROUTING -p tcp --dport ${Host_port[$i]} -j DNAT --to ${Guest_ipaddr}:${Guest_port[$i]}
      done
  fi
  fi



