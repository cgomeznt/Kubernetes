Kubernetes tomcat Deployment usando CLI y YAML
===================================================

**Descripción general:**

Primero usaremos la CLI para crear una implementación que ejecute la imagen tomcat.
En la parte de la sección, crearemos la misma implementación usando un archivo yaml pero con 4 contenedores tomcat y luego verificaremos a través de la línea de comando.

Primero crearemos la implementación usando el siguiente comando::

	ubectl create deployment tomcat-project --image=tomcat

Utilice el siguiente comando para obtener los detalles de la implementación::

	kubectl describe deployment tomcat-project

Para obtener los registros, podemos ejecutar el siguiente comando. implementación de registros de kubectl/proyecto tomcat

Ahora necesitamos eliminar nuestra implementación::

	kubectl logs deployment/tomcat-project

Hemos eliminado nuestro despliegue y continuamos con la segunda parte.


Para crear lo que teníamos anteriormente pero con 4 contenedores, usaré el siguiente archivo yaml::

	# vi tomcat.yml
	
	apiVersion: v1
	kind: Service
	metadata:
	  name: tomcat-service-nautilus
	spec:
	  type: NodePort
	  selector:
		app: tomcat
	  ports:
		- port: 80
		  protocol: TCP
		  targetPort: 8080
		  nodePort: 32227
	---
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: tomcat-deployment-nautilus
	spec:
	  replicas: 3
	  selector:
		matchLabels:
		  app: tomcat
	  template:
		metadata:
		  labels:
			app: tomcat
		spec:
		  containers:
			- name: tomcat-container-nautilus
			  image: gcr.io/kodekloud/centos-ssh-enabled:tomcat
			  ports:
				- containerPort: 8080



Guárdelo en un archivo y use el siguiente comando para ejecutarlo::

	# kubectl apply -f tomcat.yaml
	service/tomcat-project created
	deployment.apps/tomcat-project created


Ahora hacemos un describe::

	# kubectl get service
	NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
	kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   42h
	[root@lrkdevcrediappma01 ~]# kubectl describe deployment tomcat-project
	Name:                   tomcat-project
	Namespace:              default
	CreationTimestamp:      Fri, 21 Jun 2024 10:07:29 -0400
	Labels:                 app=tomcat-project
	Annotations:            deployment.kubernetes.io/revision: 1
	Selector:               app=tomcat-project
	Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
	StrategyType:           RollingUpdate
	MinReadySeconds:        0
	RollingUpdateStrategy:  25% max unavailable, 25% max surge
	Pod Template:
	  Labels:  app=tomcat-project
	  Containers:
	   tomcat:
		Image:         tomcat
		Port:          <none>
		Host Port:     <none>
		Environment:   <none>
		Mounts:        <none>
	  Volumes:         <none>
	  Node-Selectors:  <none>
	  Tolerations:     <none>
	Conditions:
	  Type           Status  Reason
	  ----           ------  ------
	  Available      True    MinimumReplicasAvailable
	  Progressing    True    NewReplicaSetAvailable
	OldReplicaSets:  <none>
	NewReplicaSet:   tomcat-project-7b5d9946b6 (1/1 replicas created)
	Events:
	  Type    Reason             Age   From                   Message
	  ----    ------             ----  ----                   -------
	  Normal  ScalingReplicaSet  29s   deployment-controller  Scaled up replica set tomcat-project-7b5d9946b6 to 1



Podemos ver que se ha escalado a 3.

También podemos obtener información específica relacionada con los pods en sí, las implementaciones y los servicios ejecutando los siguientes comandos respectivamente::


	# kubectl get service
	NAME            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
	kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP        43h
	tomcat-project   LoadBalancer   10.109.225.186   <pending>     80:31465/TCP   34m

	# kubectl get deploy
	NAME            READY   UP-TO-DATE   AVAILABLE   AGE
	tomcat-project   3/3     3            3           34m

	# kubectl get pods
	NAME                            READY   STATUS    RESTARTS   AGE
	tomcat-project-c9cd95999-gr9l4   1/1     Running   0          34m
	tomcat-project-c9cd95999-hzzf8   1/1     Running   0          34m
	tomcat-project-c9cd95999-sdvn7   1/1     Running   0          34m


Hay un detalle al momento de hacer el diagnostico en un kubernetes 1.30.2, no se visualizan los puertos en los master ni en los worker.

Veamos si obtenemos respuesta de localhost con el siguiente comando::

	# curl localhost:31465
	<!DOCTYPE html>
	<html>
	<head>
	<title>Welcome to tomcat!</title>
	<style>
		body {
			width: 35em;
			margin: 0 auto;
			font-family: Tahoma, Verdana, Arial, sans-serif;
		}
	</style>
	</head>
	<body>
	<h1>Welcome to tomcat!</h1>
	<p>If you see this page, the tomcat web server is successfully installed and
	working. Further configuration is required.</p>

	<p>For online documentation and support please refer to
	<a href="http://tomcat.org/">tomcat.org</a>.<br/>
	Commercial support is available at
	<a href="http://tomcat.com/">tomcat.com</a>.</p>

	<p><em>Thank you for using tomcat.</em></p>
	</body>
	</html>

Aunque deberiamos consultar por el puerto 80, no logramos dicha consulta y lo hacemos por el puerto del kubernete, que en este ejemplo es 31465
	
Si queremos reiniciar los pods::

	# kubectl scale deployment tomcat-project --replicas=0
	deployment.apps/tomcat-project scaled

	# kubectl get pods
	No resources found in default namespace.

	# kubectl scale deployment tomcat-project --replicas=3
	deployment.apps/tomcat-project scaled

	# kubectl get pods
	NAME                            READY   STATUS    RESTARTS   AGE
	tomcat-project-c9cd95999-6wql5   1/1     Running   0          2s
	tomcat-project-c9cd95999-dschd   1/1     Running   0          2s
	tomcat-project-c9cd95999-zfqfb   1/1     Running   0          2s

Para depurar lo instalado
+++++++++++++++++++++++++++++++

Borramos el service y el deploy::

	# kubectl delete service tomcat-project
	service "tomcat-project" deleted

	# kubectl delete deploy tomcat-project
	deployment.apps "tomcat-project" deleted
	
También podemos utilizar el archivo yaml::

	# kubectl delete -f tomcat.yaml
	service "tomcat-project" deleted
	deployment.apps "tomcat-project" deleted

