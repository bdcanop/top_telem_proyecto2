# st0263-Proyecto-2

## Instructivo Paso a Paso:

### Creación y configuración inicial de instancias.
Primero, cree 4 máquinas virtuales en AWS. Llame a una de ellas Master, y a otras dos Worker 1 y Worker 2, respectivamente, y finalmente a otra instancia llamada NFS. 

Para las cuatro máquinas utilice la imagen de Ubuntu

![image](https://github.com/user-attachments/assets/c67b8339-e820-480b-a90c-836dc261d831)

Tipo de instancia t2.medium

![image](https://github.com/user-attachments/assets/2786bad5-76de-44f2-8eea-069e4042947d)

Par de claves la que quiera asignar (en nuestro caso vockey)

![image](https://github.com/user-attachments/assets/6eb142d1-3844-465a-a958-31b52af9a09d)

Cree un nuevo grupo de seguridad despues habilitar las siguientes reglas

![image](https://github.com/user-attachments/assets/711d530b-b065-41b3-befe-2417a2fc2633)
![image](https://github.com/user-attachments/assets/5af4ad3f-8828-4b38-8f01-3703f3ab020c)

Por ultimo el almacenamiento de 20 GiB.

![image](https://github.com/user-attachments/assets/c9afdc7a-687d-45a9-865c-02147041c70e)

### Configuración inicial de instancia master, worker 1, worker 2 y NFS.
- Conéctese a la instancia de master, worker 1, worker 2 y NFS a través de SSH.
- Para cada una de las instancias ejecute estos comandos
  
- Ejecute el comando:

```
sudo apt update
sudo snap install microk8s --classic
microk8s status --wait-ready
```
Saldrá entonces un mensaje como el siguiente:

![image](https://github.com/user-attachments/assets/3623e45d-bd60-44f1-9e15-be02cae33282)

- Proceda a ejecutar individualmente los comandos que le solicitan ahí.
  
```
sudo usermod -a -G microk8s USUARIO
mkdir .kube
sudo chown -R USUARIO ~/.kube
newgrp microk8s
```

- Vuelva a ejecutar el comando:
  
```
microk8s status --wait-ready
```
Le aparecerá ahora sí un mensaje que dice que microk8s está ejecutándose.

- Ahora, ejecute los comandos:
  
```
microk8s enable dashboard
```
```
microk8s enable dns
```
```
microk8s enable registry
```
```
microk8s enable istio
```
```
microk8s enable ingress
```
- Este ultimo comando #SOLO ejecutarlo en la instancia Master
```
microk8s enable cert-manager
```
Con estos comandos podrá instalar los plugins necesarios para microk8s.

A partir de acá será necesario usar comandos que tienen el prefijo ```microk8s kubectl```. Sin embargo, esto puede ser largo y tornarse ineficiente en términos de escritura. Por lo tanto, usaremos un alias para evitar tener que escribirlo completo. 
Para ello, ejecute el comando:

```
alias kubectl="microk8s kubectl"
```
Así, ya solo debera usar el prefijo ```kubectl```.


### Configuración NFS.
- Ejecuto estos comandos solamente en la instancia NFS

- Proceda a instalar el servidor NFS con el siguiente comando:

```
sudo apt update
sudo apt install nfs-kernel-server
```
- Ahora, cree el directorio que se compartirá:
```
sudo mkdir -p /mnt/wordpress
```
- Cambie el propietario del directorio:
```
sudo chown nobody:nogroup /mnt/wordpress
```
- Después, cambie los permisos del directorio:
```
sudo chmod 777 /mnt/wordpress
```
- Edite el archivo /etc/exports:
```
sudo nano /etc/exports
```
- Deberá añadir entonces la siguiente línea en el archivo recién creado y abierto:
```
/mnt/wordpress *(rw,sync,no_subtree_check)
```
![image](https://github.com/user-attachments/assets/ce1da635-7180-4637-85ae-16050940a1e6)

- Guarde los cambios hechos en el archivo, y después reinicie el servicio de NFS:
```
sudo systemctl restart nfs-kernel-server
```

### Creación del clúster.

#### Configuración workers.
En la instancia Master ejecute el comando:

```
microk8s add-node
```

Le mostrará entonces un comando que debe ejecutar en la instancia ```Worker 1```. Ejecútelo allí. (seleccione el comando que termine con '--worker')

![image](https://github.com/user-attachments/assets/6bb9a7b3-53ff-40c3-9122-817822a3ad1b)

Una vez le diga que se conectó al cluster satisfactoriamente ejecute nuevamente el mismo comando en la instancia master. Nuevamente, ejecute el comando que se le muestre en la consola pero esta vez en la instancia ```Worker 2```. 

Cuando ejecute el comando 
```
kubectl get nodes
``` 
en la instancia master, después de haber hecho lo anterior, podrá ver los dos nodos workers recién añadidos al clúster.

![image](https://github.com/user-attachments/assets/04766c7c-1dea-44f5-8d92-7e445c358d60)

#### Configuración NFS.

Conectado a la instancia Master y realice las siguientes acciones:

- Primero, habilite Helm3 y añada el repositorio de CSI Driver para NFS:
```
microk8s enable helm3
microk8s helm3 repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
microk8s helm3 repo update
```

- Después, instale el driver para NFS:
  
```
microk8s helm3 install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
 --namespace kube-system \
 --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet
```

- Cree el StorageClass con el siguiente comando:
```
nano nfs-csi.yaml
```
- En ese archivo creado y abierto pegue lo siguiente:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: <IP-PRIVADA-NFS-SERVER>
  share: /mnt/wordpress
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
```

**Nota:** deberá cambiar el valor <IP-PRIVADA-NFS-SERVER> por la IP privada de su servidor NFS.

- Ahora, aplique el archivo nfs-csi.yaml:
```
kubectl apply -f nfs-csi.yaml
```
- Después de ello, cree el PersistentVolumeClaim con el siguiente comando:
```
nano nfs-pvc.yaml
```
- Después, en ese archivo recién creado y abierto coloque el siguiente contenido:
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  storageClassName: nfs-csi
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```
    
- Finalmente, aplique el archivo nfs-pvc.yaml:
```
kubectl apply -f nfs-pvc.yaml
```

#### Configuración Base de Datos y WordPress.

Conectado a la instancia microk8s-master, realice lo siguiente:

- Para la creación de un kustomization file, con el fin de almacenar un Secret con información de la base de datos, ejecute el siguiente comando cambiando el valor **YOUR_PASSWORD** por la contraseña que desee colocar para la base de datos:
```
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=YOUR_PASSWORD
EOF
```

- Proceda a aplicar dichos archivos con el comando a continuación:
```
kubectl apply -k ./
```
- Ahora ejecute este comando para verificar la creacion del secret
```
kubectl get secret
```
- Alli le aparecera algo como la siguiente imagen (copie y guarde el nombre del secret, en nuestro caso es "mysql-pass-2kt4dbgtk8")

![image](https://github.com/user-attachments/assets/aec9e71e-0750-40dc-aeea-4e028e663edb)

- Cree un archivo llamado ```mysql-pv-pvc.yaml```, el cual corresponderá al PersistentVolume y al PersistentVolumeClaim para MySQL, con el siguiente comando:
```
nano mysql-pv-pvc.yaml
```
- Coloque dentro del archivo recién creado y abierto el siguiente contenido:
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: nfs-csi
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: nfs-csi
  accessModes:
    - ReadWriteOnce
  volumeName: mysql-pv-volume
  resources:
    requests:
      storage: 5Gi
```

- Después, cree otro archivo llamado ```mysql-deployment.yaml```, el cual corresponderá al deployment y al service para MySQL y la base de datos deseada, con el siguiente comando:
```
nano mysql-deployment.yaml
```
- Coloque dentro del archivo recién creado y abierto el siguiente contenido. Ademas cambie el nombre de la secretKeyRef por el nombre del secret de MySQL que guardo anteriormente:

![image](https://github.com/user-attachments/assets/b014c589-28e7-4b0b-9cd0-07f1af8e5939)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:8.0
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass-2kt4dbgtk8
              key: password
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass-2kt4dbgtk8
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
  - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
```

- Después, cree otro archivo llamado ```wordpress-deployment.yaml```, el cual corresponderá al deployment, al service, y al PersistentVolumeClaim para WordPress, con el siguiente comando:
```
nano wordpress-deployment.yaml
```
- Coloque dentro del archivo recién creado y abierto el siguiente contenido. Ademas cambie el nombre de la secretKeyRef por el nombre del secret de MySQL que guardo anteriormente:

![image](https://github.com/user-attachments/assets/1fa995dc-f03f-4d99-8eec-031f49920a49)

```
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-csi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass-2kt4dbgtk8
              key: password
        - name: WORDPRESS_DB_USER
          value: wordpress
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

- Después de crear los archivos anteriores, proceda a aplicar el manifiesto del archivo ```mysql-pv-pvc.yaml```, con el siguiente comando:
```
kubectl apply -f mysql-pv-pvc.yaml
```
- Ahora, ejecute el siguiente comando para añadir los deployments previamente creados al archivo kustomization:
```
cat <<EOF >>./kustomization.yaml
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
EOF
```
- Proceda a aplicar dichos archivos con el comando a continuación:
```
kubectl apply -k ./
```

De esta forma, ya tiene WordPress y MySQL configurados y conectados correctamente. 

#### Configuración Load Balancer.
Continuando con la conexión a la instancia Master, realice las siguientes acciones:

- Primero, con el fin de redirigir las requests al dominio correspondiente al proyecto, diríjase al panel de su proveedor de dominio (en este caso, My No-Ip), y modifica la ip por la ip publica de su instancia Master:

![image](https://github.com/user-attachments/assets/56e822a1-430a-400b-b48f-653fc58efe6e)

- Ahora, proceda a crear un archivo llamado ```cluster-issuer.yaml``` para configurar una cuenta de correo con Let's Encrypt, con el comando a continuación:
```
nano cluster-issuer.yaml
```
- Coloque el siguiente contenido en el archivo recién creado y abierto:
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: lets-encrypt
spec:
  acme:
    email: <EMAIL-ADDRESS>
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: lets-encrypt-private-key
    solvers:
    - http01:
        ingress:
          class: public
```
**Nota:** deberá cambiar el valor <EMAIL-ADDRESS> por el correo con el cual desea cofigurar Let's Ecrypt.

- Proceda a aplicar dicho archivo con el comando:
```
kubectl apply -f cluster-issuer.yaml
```

- Ahora, pase a configurar el Ingress, para lo cual será necesario crear un archivo llamado ```nginx-ingress.yaml```, con el siguiente comando:
```
nano nginx-ingress.yaml
```
- Coloque dentro de ese archivo recién creado y abierto el siguiente contenido:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wordpress-ingress
  annotations:
    cert-manager.io/cluster-issuer: lets-encrypt
spec:
  tls:
  - hosts:
    - <YOUR-DOMAIN>
    secretName: wordpress-ingress-tls
  rules:
  - host: <YOUR-DOMAIN>
    http:
      paths:
      - backend:
          service:
            name: wordpress
            port:
              number: 80
        path: /
        pathType: Exact
```
**Nota:** deberá cambiar el valor <YOUR-DOMAIN> por su dominio correspondiente.
- Proceda a aplicar ese archivo con el comando:
```
kubectl apply -f nginx-ingress.yaml
```
**Nota:** deberá esperar unos 15 minutos aproximadamente para que se creen los certificados TLS correspondientes y se almacenen en las carpetas respectivas. 

#### Verificación del funcionamiento de la aplicación.

- Ejecute estos comandos:
```
kubectl get pods
```
```
kubectl get svc
```
```
kubectl get pv
```
```
kubectl get pvc
```
```
kubectl get ingress
```
```
kubectl get certificate
```
- Despues de ejecutar todos los comandos verifique lo siguiente
- Los pods estan en estado ```Running```
- Los svc estan como en la imagen
- Los pv estan en estado ```Bound```
- Los pvc estan en estado ```Bound```
- Los ingress tienen el host que usted asigno y que tenga una IP en address (como en la imagen)
- Los certificate estan en estado ```True```

![image](https://github.com/user-attachments/assets/448774a8-8861-487d-af9b-e7373f2d8030)

- Si tiene todo correcto podra acceder a su dominio y vera la pagina de wordpress funcionando
