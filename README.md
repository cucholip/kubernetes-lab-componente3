# Componente 3: Orquestación con Kubernetes

## Información General

| Campo | Detalle |
|-------|---------|
| **Asignatura** | Sistemas Operativos |
| **Componente** | 3 — Orquestación con Kubernetes (20%) |
| **Herramientas** | Minikube v1.38.1, kubectl, Docker, Ubuntu 26.04 |
| **Fecha** | Junio 2026 |


## Descripción

En este laboratorio se implementó un cluster local de Kubernetes usando Minikube sobre una máquina virtual Ubuntu. Se desplegó un servidor web Nginx mediante manifiestos YAML, se expuso el servicio con NodePort y se demostró el escalado manual de réplicas.


## Marco Teórico

**Kubernetes** es una plataforma de orquestación de contenedores que automatiza el despliegue, escalado y gestión de aplicaciones en contenedores. Permite declarar el estado deseado del sistema mediante archivos YAML y se encarga de mantenerlo.

**Minikube** es una herramienta que levanta un cluster de Kubernetes de un solo nodo en una máquina local. Es ideal para desarrollo y aprendizaje sin necesidad de infraestructura en la nube.

**kubectl** es la herramienta de línea de comandos oficial para interactuar con clusters de Kubernetes. Permite aplicar manifiestos, consultar el estado del cluster, escalar deployments y mucho más.

**Docker** actúa como el motor de contenedores que Minikube usa internamente para simular los nodos del cluster y ejecutar los pods.

**Deployment** es un recurso de Kubernetes que define qué imagen de contenedor desplegar, cuántas réplicas mantener activas y cómo actualizarlas. Garantiza que el número deseado de pods esté siempre corriendo.

**Service** es un recurso que expone los pods como un punto de acceso estable. Sin un Service, los pods no son accesibles desde fuera del cluster. El tipo `NodePort` asigna un puerto fijo en el nodo para acceso externo.


## Actividades Realizadas

### 1. Instalación y verificación de Minikube

Se verificó la instalación de Minikube, kubectl y Docker en la distribución gráfica de Ubuntu. Minikube requiere Docker como motor de contenedores para poder simular el cluster localmente.

minikube version
# minikube version: v1.38.1

imagen: minikube-version.png


### 2. Creación de Manifiestos YAML

Se crearon dos archivos de configuración que describen el estado deseado del sistema. En Kubernetes todo se define de forma declarativa mediante estos archivos.

**deployment.yaml**

Define el Deployment de Nginx con 2 réplicas. Las réplicas garantizan alta disponibilidad: si un pod falla, el otro sigue respondiendo. Se usa la imagen oficial nginx:latest expuesta en el puerto 80 (HTTP estándar).

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

**service.yaml**

Define el Service de tipo NodePort. El Deployment crea los pods pero no los hace accesibles desde fuera del cluster. El Service actúa como puerta de entrada y con NodePort asigna el puerto fijo 30080 en la IP del nodo para acceso externo.

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080


### 3. Inicio del Cluster y Despliegue

Se inició el cluster con minikube start, lo que enciende el nodo de Kubernetes. Sin este paso kubectl no tiene a dónde conectarse. Luego se aplicaron los manifiestos con kubectl apply, que envía las instrucciones al cluster para que las ejecute.

minikube start

kubectl apply -f deployment.yaml
# deployment.apps/nginx-deployment created

kubectl apply -f service.yaml
# service/nginx-service created

imagen: kubectl-apply.png


### 4. Verificación del Despliegue

Se verificó el estado del cluster con los comandos de consulta de kubectl. get pods muestra los contenedores individuales, get deployments muestra el estado general del despliegue y get services muestra los puntos de acceso disponibles.

kubectl get pods
kubectl get deployments
kubectl get services

Los 2 pods quedaron en estado Running con 1/1 READY y el servicio nginx-service quedó expuesto como NodePort en el puerto 30080.

imagen: pods-running.png


### 5. Acceso desde el Navegador

Se accedió al servicio usando minikube service, que detecta automáticamente la IP del nodo y abre el navegador apuntando al puerto del servicio. Esto confirma que Nginx está respondiendo correctamente desde dentro del cluster.

minikube service nginx-service

Nginx respondió correctamente en http://192.168.49.2:30080, mostrando la página de bienvenida oficial.

imagen: nginx-navegador.png


### 6. Escalado Manual a 3 Réplicas

Se escaló el deployment de 2 a 3 réplicas usando kubectl scale. Kubernetes crea el pod adicional automáticamente sin apagar los existentes, demostrando una de sus ventajas principales: escalar sin tiempo de inactividad del servicio.
kubectl scale deployment nginx-deployment --replicas=3
kubectl get pods

Los 3 pods quedaron en estado Running con 1/1 READY.

imagen: escalado-3replicas.png



## Resultados

| Actividad | Resultado |
|-----------|-----------|
| Minikube instalado y funcionando | ✅ |
| Manifiestos YAML creados (Deployment y Service) | ✅ |
| Nginx desplegado con 2 réplicas | ✅ |
| Servicio expuesto con NodePort (puerto 30080) | ✅ |
| Acceso desde navegador verificado | ✅ |
| Escalado manual a 3 réplicas | ✅ |


## Conclusiones

- Kubernetes permite gestionar contenedores de forma declarativa mediante archivos YAML, lo que facilita la reproducibilidad del despliegue.
- Minikube es una herramienta útil para practicar Kubernetes en entornos locales sin necesidad de infraestructura en la nube.
- El tipo de servicio NodePort permite exponer aplicaciones dentro del cluster hacia el exterior usando un puerto fijo.
- El escalado horizontal en Kubernetes es inmediato y no requiere tiempo de inactividad del servicio.
- La separación entre Deployment y Service en Kubernetes refleja el principio de responsabilidad única: uno gestiona los pods y el otro gestiona el acceso a ellos.
