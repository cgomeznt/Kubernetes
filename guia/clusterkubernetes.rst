Implementar un clúster HA Kubernetes en CentOS7
==================

En este laboratorio vamos a trabajar con tres (3) maquinas:

* una (1) Master

* dos (2) Worker

Nuestro despliegue consiste de los siguientes componentes: 

* Kubernetes

* Etcd

* Docker

* Flannel Network

* Dashboard

* Heapster

**Arquitectura**

*k8-master01 - 192.168.1.20

*k8-worker01 - 192.168.1.21

*k8-worker02 - 192.168.1.22

Preparando los servidores
+++++++++++++++++++++++++++

Las siguientes tareas se debe realizar en todos los servidores (Master y Worker)


1) Deshabilitar Selinux::

	# setenforce 0
	# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

2) Deshabilitar swap::

	# swapoff -a
	# sed -i 's/^.*swap/#&/' /etc/fstab

3) Deshabilitar firewalld::

	# systemctl stop firewalld
	# systemctl disable firewalld

4) Reiniciar servidores y verificar selinux::

	# shutdown -r now
	# sestatus

5) Editar el archivo /etc/hosts::

	192.168.20.20	k8-master01 K8-MASTER01
	192.168.20.21	k8-master02 K8-MASTER02
	192.168.20.22	k8-master03 K8-MASTER03
	192.168.20.24	k8-worker01 K8-WORKER01
	192.168.20.25	k8-worker02 K8-WORKER02
	192.168.20.26	k8-worker03 K8-WORKER03
	192.168.20.27	k8-registry01 K8-REGISTRY01




