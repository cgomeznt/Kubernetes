Cluster Kubernetes con kubeadm en CentOS7
=========================================

https://www.kubeclusters.com/docs/How-to-Deploy-a-Highly-Available-kubernetes-Cluster-with-Kubeadm-on-CentOS7

Kubernetes es un orquestador para contenedores de Docker. En otras palabras, Kubernetes es un software o herramienta de código abierto que se utiliza para organizar y administrar contenedores de Docker en clúster ambiente. Kubernetes también se conoce como k8s y fue desarrollado por Google y donado a "Cloud Native Computing foundation”

En Kubernetes con configuración ETCD, tenemos al menos tres (3) nodos Master y varios nodos Worker. Los nodos de clúster se conocen como nodo Worker o Minion. Desde el nodo Master gestionamos el clúster y su nodos que utilizan el comando "kubeadm" y "kubectl".

El clúster de Kubernetes es altamente configurable. Muchos de sus componentes son opcionales. Nuestro despliegue consiste de los siguientes componentes: 
** Kubernetes, Etcd, Docker, Flannel Network, Dashboard y Heapster.**

Como estará configurado el laboratorio
++++++++++++++++++++++++++++++++++++++++


+---------------+-----------------------+-----------------------+
|Server Name	|IP Address		|Role			|
+---------------+-----------------------+-----------------------+
|k8-master01	|192.168.1.20		|Master Node		|
|		|			|			|
|k8-master02	|192.168.1.21		|Master Node		|
|		|			|			|
|k8-master03	|192.168.1.22		|Master Node		|
|		|			|			|
|k8-worker01	|192.168.1.30		|Worker Node		|
|		|			|			|
|k8-registry01	|			|			|
+---------------+-----------------------+-----------------------+


Preparando los servidores
========================

Hay algunas configuraciones previas que se deben realizar en los servidores.

Deshabilitar Selinux
+++++++++++++++++++++
::

	# setenforce 0
	# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

Deshabilitar swap
+++++++++++++++++
::

	# swapoff -a
	# sed -i 's/^.*swap/#&/' /etc/fstab

Deshabilitar firewalld
++++++++++++++++++++++++
::

	# systemctl stop firewalld
	# systemctl disable firewalld

Editar el archivo /etc/hosts
+++++++++++++++++++++++++++++++
::

	# vi  /etc/hosts

	192.168.1.20    k8master01
	192.168.1.21    k8master02
	192.168.1.22    k8master03
	192.168.1.23    k8worker01
	
Cambiar los Hostname de cada uno de los servidores
+++++++++++++++++++++++++++++++++++++++++++
En cada uno de los servidores coloque el mismo nombre como lo coloco en el archivo /etc/hosts::

	hostnamectl set-hostname k8master01
	
	hostnamectl set-hostname k8master02
	
	hostnamectl set-hostname k8master03
	
	hostnamectl set-hostname k8worker01

Reiniciar servidores y verificar selinux
++++++++++++++++++++++++++++++++++++++++++++
::

	# shutdown -r now

	# sestatus 
	SELinux status:                 disabled

	# systemctl status firewalld
	● firewalld.service - firewalld - dynamic firewall daemon
	   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
	   Active: inactive (dead)
	     Docs: man:firewalld(1)




Instalar NTP
++++++++++++++++
::

	# yum install -y ntp
	# systemctl start ntpd
	# systemctl enable ntpd

Establecer la timezone
++++++++++++++++++++++
::

	# timedatectl set-timezone America/Caracas

Instalar Docker
++++++++++++++++

La mejor documentación es la oficial de Docker https://docs.docker.com/engine/install/

Esto es como instalar Docker::

	yum install -y yum-utils

	yum-config-manager --add-repo  https://download.docker.com/linux/centos/docker-ce.repo

	yum install docker-ce docker-ce-cli containerd.io


Cambiar el cgroupdrive al docker de init a systemd, tal como lo recomienda Kubernetes::


	vi /usr/lib/systemd/system/docker.service
	ExecStart=/usr/bin/dockerd --exec-opt native.cgroupdriver=systemd


Recargamos el servicio, lo habilitamos y lo iniciamos::

	systemctl daemon-reload

	systemctl enable docker

	systemctl restart docker

Docker para Kubernetes debe tener el Storage Drive de overlay2. Para saber si Docker es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ta utilizando el Driver de overlay2::

	# docker info | grep Storage
	 Storage Driver: overlay2

Nos aseguramos que Cgroup Driver sea  systemd::

	# docker info | grep -i cgroup
	 Cgroup Driver: systemd
	 Cgroup Version: 1

Hay un bug con  CentOS 7, XFS y el soporte d_type. El soporte d_type debe estar habilitado en el filesystem de XFS, ver los siguientes link, por ejemplo, en donde tenga instalado Docker, en este ejemplo esta en /var/docker::


	# xfs_info /var/docker/
	meta-data=/dev/sdb               isize=512    agcount=4, agsize=1310720 blks
		 =                       sectsz=512   attr=2, projid32bit=1
		 =                       crc=1        finobt=0 spinodes=0
	data     =                       bsize=4096   blocks=5242880, imaxpct=25
		 =                       sunit=0      swidth=0 blks
	naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
	log      =internal               bsize=4096   blocks=2560, version=2
		 =                       sectsz=512   sunit=0 blks, lazy-count=1
	realtime =none                   extsz=4096   blocks=0, rtextents=0




Para más detalle de XFS y d_type ver estos link::

	https://www.thegeekdiary.com/how-to-create-an-xfs-filesystem/

	https://medium.com/@khushalbisht/docker-on-centos-7-with-xfs-filesystem-can-cause-trouble-when-d-type-is-not-supported-64cee61b39ab




Verificamos el status de Docker ::

	# systemctl status docker
	● docker.service - Docker Application Container Engine
	   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
	   Active: active (running) since mar 2021-12-21 21:29:04 -04; 5min ago
	     Docs: https://docs.docker.com
	 Main PID: 1413 (dockerd)
	    Tasks: 8
	   Memory: 32.5M
	   CGroup: /system.slice/docker.service
		   └─1413 /usr/bin/dockerd --exec-opt native.cgroupdriver=systemd -g /var/docker


Realizamos una prueba de Docker::

	# docker run hello-world
	
	Unable to find image 'hello-world:latest' locally
	latest: Pulling from library/hello-world
	2db29710123e: Pull complete 
	Digest: sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
	Status: Downloaded newer image for hello-world:latest

	Hello from Docker!
	This message shows that your installation appears to be working correctly.

	To generate this message, Docker took the following steps:
	 1. The Docker client contacted the Docker daemon.
	 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
	    (amd64)
	 3. The Docker daemon created a new container from that image which runs the
	    executable that produces the output you are currently reading.
	 4. The Docker daemon streamed that output to the Docker client, which sent it
	    to your terminal.

	To try something more ambitious, you can run an Ubuntu container with:
	 $ docker run -it ubuntu bash

	Share images, automate workflows, and more with a free Docker ID:
	 https://hub.docker.com/

	For more examples and ideas, visit:
	 https://docs.docker.com/get-started/
	 

Crear el archivo /etc/sysctl.d/k8s.conf con el siguiente contenido y luego ejecutar el comando
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

	# vi /etc/sysctl.d/k8s.conf
	vm.dirty_expire_centisecs = 500
	vm.swappiness = 10
	net.ipv4.conf.all.forwarding=1
	net.bridge.bridge-nf-call-iptables = 1
	net.bridge.bridge-nf-call-ip6tables = 1
	kernel.pid_max = 4194303

	# sysctl -p /etc/sysctl.d/k8s.conf o sysctl --system

Explícitamente el modulo br_netfilter esta cargado, lo consultamos para estar 100% seguros::

	# lsmod | grep br_netfilter
	br_netfilter           22256  0 
	bridge                151336  1 br_netfilter


Instalar kubelet, kubeadm, kubectl
++++++++++++++++++++++++++++++++
::

	cat <<EOF > /etc/yum.repos.d/kubernetes.repo
	[kubernetes]
	name=Kubernetes
	baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
	enabled=1
	gpgcheck=1
	repo_gpgcheck=1
	gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
	EOF

Listamos todas las versiones de kubeadm, kubelet, kubectl, por si queremos escoger una en particular::

	yum list --showduplicates kubeadm --disableexcludes=kubernetes
	
	yum list --showduplicates kubelet --disableexcludes=kubernetes
	
	yum list --showduplicates kubectl --disableexcludes=kubernetes
	
Instalamos los paquetes kubeadm, kubelet, kubectl y en este ejemplo utilizamos la versión 1.19.15-0::

	yum install -y kubeadm-1.19.15-0  kubectl-1.19.15-0  kubelet-1.19.15-0  --disableexcludes=kubernetes
	

Solo habilitamos el servicio kubelet, NO se debe iniciar porque sino tendrá errores::

	systemctl daemon-reload
	
	systemctl enable --now kubelet


Completar comando docker kubeadm kubectl
+++++++++++++++++++++++++++++++++++++++++++

Esto consume muchos recursos en el bash, solo se debería bajo una evaluación::

	yum install bash-completion
	source /usr/share/bash-completion/bash_completion
	Desloguea y vuelve a ingresar al perfil. Prueba con:
	type _init_completion
	kubectl completion bash >/etc/bash_completion.d/kubectl
	kubeadm completion bash >/etc/bash_completion.d/kubeadm
	source /usr/share/bash-completion/completions/docker
	

Crear certificados (ejecutar en todos los nodos)
++++++++++++++++++++++++++++++++++++++++++++++++++

Vamos a descargar los PKI and TLS toolkit::

	# curl -o /usr/local/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64

	# curl -o /usr/local/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

	# chmod +x /usr/local/bin/cfssl*

	# mkdir -p /etc/kubernetes/pki/etcd && cd /etc/kubernetes/pki/etcd
	
	

**Hasta aquí es igual tanto para Master como para Workers**

