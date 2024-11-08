# Despliegue de Moodle utilizando EKS-EFS (Reto 2)

## Estudiante(s): 
Jonathan Betancur, jbetancur3@eafit.edu.co  
Esteban Vergara Giraldo, evergarag@eafit.edu.co  

## Profesor: 
Alvaro Enrique Ospina SanJuan, aeospinas@eafit.edu.co  

---

## 1. Breve descripción

En el proyecto, desplegamos Moodle en un clúster Kubernetes de alta disponibilidad en la nube, utilizando la imagen de Moodle en un entorno que incluye autoescalado y un balanceador de carga NGINX. Configuramos un dominio personalizado con certificado SSL gestionado por Cert-Manager y añadimos servidores dedicados para la base de datos y el almacenamiento de archivos (NFS). Este despliegue en AWS EKS asegura escalabilidad y rendimiento óptimo para el LMS.

### 1.1. Aspectos cumplidos o desarrollados:

- Se implementó el despliegue de Moodle en un clúster de alta disponibilidad en Kubernetes, utilizando la imagen de Moodle y configurando un dominio personalizado: [https://kub.reto2telematicakubernetes.online](https://kub.reto2telematicakubernetes.online).
- Configuración de un balanceador de carga NGINX para distribuir el tráfico a nivel de aplicación.
- Implementación de un sistema de almacenamiento distribuido mediante un servidor NFS, separado de la capa de aplicación, cumpliendo con los requisitos de almacenamiento externo.
- Configuración de una base de datos MariaDB externa en el clúster, asegurando la disponibilidad y rendimiento para las necesidades de Moodle.
- Se aplicó un certificado SSL mediante Cert-Manager para garantizar la seguridad en las conexiones al dominio.
- El sistema se implementó con un enfoque de autoescalado, permitiendo el ajuste dinámico de recursos según la demanda.

### 1.2. Aspectos NO cumplidos:

- Aunque se completaron los aspectos funcionales del despliegue, los archivos CSS de Moodle no se cargaron correctamente, afectando la apariencia visual de la interfaz.
- Nos gastamos todos los créditos de AWS, por lo que la página actualmente no está activa.

---

## 2. Información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas

- **Arquitectura distribuida:** Despliegue en un clúster Kubernetes de alta disponibilidad, distribuyendo las cargas de trabajo entre múltiples nodos y garantizando la resiliencia mediante réplicas en la base de datos y el sistema de archivos. Inspirada en arquitecturas de alta disponibilidad como GFS y HDFS.
- **Balanceador de carga NGINX:** Configurado en la capa de aplicación para distribuir el tráfico de manera uniforme entre las réplicas de Moodle, optimizando la escalabilidad y disponibilidad.
- **Separación de almacenamiento y base de datos:** Implementación de un servidor NFS para el almacenamiento de archivos y una base de datos externa MariaDB para el manejo de datos de Moodle, asegurando un manejo eficiente y seguro de los recursos de almacenamiento.
- **Certificado SSL y dominio personalizado:** Implementación de HTTPS mediante Cert-Manager para la gestión de certificados SSL, asociado al dominio configurado en GoDaddy [https://kub.reto2telematicakubernetes.online](https://kub.reto2telematicakubernetes.online).
- **Autoescalado y monitoreo:** Configuración de autoescalado en el clúster para responder a picos de demanda, así como el uso de monitoreo en AWS para mantener el control de recursos y asegurar la disponibilidad.
- **Buenas prácticas de seguridad y redundancia:** Aplicación de prácticas de redundancia y replicación de datos en la base de datos y almacenamiento para minimizar la pérdida de información y garantizar la alta disponibilidad del sistema.

---

## 3. Descripción del ambiente de desarrollo y técnico

- **Lenguaje de programación y configuración:**  
  - YAML (para los manifiestos de Kubernetes y configuración de servicios).
  - Bash y AWS CLI (para la gestión y despliegue de infraestructura en AWS).

- **Herramientas y servicios en la nube:**  
  - **AWS EKS (Elastic Kubernetes Service):** Plataforma en la que se despliega el clúster Kubernetes, permitiendo la gestión automatizada de nodos y recursos.
  - **AWS EFS (Elastic File System):** Servicio de almacenamiento NFS utilizado para gestionar el almacenamiento persistente de archivos de Moodle.
  - **Cert-Manager:** Para gestionar el certificado SSL y asegurar las conexiones HTTPS.
  - **GoDaddy:** Para la configuración del dominio personalizado y enrutamiento de DNS.
  
- **Componentes en Kubernetes:**  
  - **NGINX Ingress Controller:** Actúa como balanceador de carga y gestiona el acceso externo al clúster a través de HTTP/HTTPS.
  - **MariaDB (Base de datos):** Base de datos desplegada en el clúster o en contenedores para almacenar los datos de Moodle.
  - **Persistent Volumes (PV) y Persistent Volume Claims (PVC):** Definidos para gestionar el almacenamiento de archivos de Moodle y la base de datos en el sistema de archivos EFS.

### Cómo se configura y despliega:

1. **Configuración del clúster EKS y nodos en AWS:**
   - Utilizar AWS CLI o la consola de AWS para crear el clúster EKS y los nodos de trabajo.
   - Configurar el acceso mediante `kubectl` para gestionar el clúster.

   ```bash
   aws eks update-kubeconfig --name nombre_del_clúster --region us-east-1
   ```

2. **Despliegue de los manifiestos de Kubernetes:**
   - Aplicar los manifiestos para la configuración de Moodle, la base de datos MariaDB, el Ingress NGINX y el servidor NFS.

   ```bash
   kubectl apply -f moodle-deployment.yaml
   kubectl apply -f mariadb-deployment.yaml
   kubectl apply -f ingress-nginx.yaml
   kubectl apply -f efs-pv.yaml
   ```

3. **Configuración de Certificados SSL con Cert-Manager:**
   - Configurar Cert-Manager para gestionar el certificado SSL automáticamente y vincularlo al servicio Ingress para habilitar HTTPS.

4. **Configuración del dominio en GoDaddy:**
   - Configurar el dominio en GoDaddy y crear un registro A que apunte al balanceador de carga de AWS, asegurando el acceso externo al dominio configurado.

5. **Verificación del despliegue:**
   - Comprobar el estado de los pods y servicios en Kubernetes para confirmar que todos los componentes (Moodle, MariaDB, NGINX Ingress) estén en estado "Running."

   ```bash
   kubectl get pods
   kubectl get svc
   ```
   
---

## 4. Configuración y despliegue en AWS EKS

### Configuración del entorno en AWS

#### 1. Configuración de la VPC y subredes
- Crear una VPC con 3 subredes (dos públicas y una privada) en distintas zonas de disponibilidad para alta disponibilidad.
- Configurar un Internet Gateway para las subredes públicas y una tabla de enrutamiento para habilitar la comunicación externa.
- Crear y configurar los Grupos de Seguridad necesarios para permitir el tráfico HTTP, HTTPS y los puertos requeridos por Kubernetes.

#### 2. Configuración de EFS (Elastic File System)
- Crear un sistema de archivos EFS y asociarlo a las subredes configuradas en la VPC.
- Configurar los puntos de montaje para el EFS en las subredes.
- Aplicar los archivos `efs-storageclass.yaml`, `efs-pv.yaml`, y `efs-pvc.yaml` para definir la clase de almacenamiento, el volumen persistente y el reclamo de volumen en Kubernetes.

```bash
kubectl apply -f /moodle-deployment/efs-storageclass.yaml
kubectl apply -f /moodle-deployment/efs-pv.yaml
kubectl apply -f /moodle-deployment/efs-pvc.yaml
```

### Configuración del clúster EKS

#### 1. Crear el clúster EKS y nodos
- Utilizar AWS CLI para crear el clúster EKS en la región deseada.
- Crear un grupo de nodos asociado al clúster EKS, configurando el autoescalado para gestionar la demanda de recursos.

```bash
aws eks create-nodegroup --cluster-name cluster-moodle --nodegroup-name nodes-moodle --node-role arn:aws:iam::<ACCOUNT_ID>:role/LabRole --subnets <SUBNET_IDS> --scaling-config minSize=1,maxSize=3,desiredSize=2 --instance-types t3.medium --region us-east-1
```

#### 2. Configurar `kubectl` para gestionar el clúster
```bash
aws eks update-kubeconfig --name cluster-moodle --region us-east-1
```

### Instalación de Helm y Cert-Manager en CloudShell

#### 1. Instalar Helm
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

#### 2. Instalar Cert-Manager para gestionar los certificados SSL
```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml
```

#### 3. Configurar el emisor de certificados Let's Encrypt
Aplicar el archivo `letsencrypt-issuer.yaml` para configurar el emisor de certificados SSL con Let's Encrypt.

```bash
kubectl apply -f /moodle-deployment/letsencrypt-issuer.yaml
```

### Despliegue de NGINX Ingress y servicio de base de datos MariaDB

#### 1. Desplegar NGINX para el balanceo de carga
- Aplicar los archivos `nginx-deployment.yaml` y `nginx-service.yaml` para configurar el despliegue y servicio de NGINX.

```bash
kubectl apply -f /moodle-deployment/nginx-deployment.yaml
kubectl apply -f /moodle-deployment/nginx-service.yaml
```

#### 2. Desplegar MariaDB para la base de datos de Moodle
- Inicialmente, intentamos configurar MySQL, pero surgieron problemas, por lo que se implementó MariaDB en su lugar.
- Aplicar los archivos `mariadb-deployment.yaml` y `mariadb-service.yaml` (anteriormente `mysql-deployment.yaml` y `mysql-service.yaml`) para configurar la base de datos MariaDB en Kubernetes.

```bash
kubectl apply -f /moodle-deployment/mariadb-deployment.yaml
kubectl apply -f /moodle-deployment/mariadb-service.yaml
```

### Despliegue de Moodle con Helm y archivos YAML

#### 1. Desplegar Moodle utilizando los archivos YAML
- Aplicar los archivos `moodle-deployment.yaml` y `moodle-service.yaml` para crear el despliegue y servicio de Moodle.

```bash
kubectl apply -f /moodle-deployment/moodle-deployment.yaml
kubectl apply -f /moodle-deployment/moodle-service.yaml
```

#### 2. Configurar el Ingress con Certificado SSL
- Aplicar el archivo `moodle-ingress.yaml` para gestionar el acceso HTTP/HTTPS al clúster mediante el balanceador de carga y Cert-Manager.

```bash
kubectl apply -f /moodle-deployment/moodle-ingress.yaml
```

#### 3. Crear y aplicar el certificado para el dominio de Moodle
- Aplicar el archivo `moodle-certificate.yaml` para crear el certificado SSL asociado al dominio de Moodle utilizando Cert-Manager.

```bash
kubectl apply -f /moodle-deployment/moodle-certificate.yaml
```

### Configuración y modificación del archivo `config.php` de Moodle

#### 1. Acceder al contenedor de Moodle
```bash
kubectl exec -it <POD_NAME> -- /bin/bash
```

#### 2. Copiar el archivo `config.php` para editarlo
```bash
kubectl cp <POD_NAME>:/bitnami/moodle/config.php ./moodle-deployment/config.php
```

#### 3. Modificar `config.php` para ajustar los valores de configuración de Moodle
- Cambiar las credenciales de la base de datos y la URL base (`wwwroot`) en el archivo `config.php`.

```php
<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mariadb';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'my-moodle-mariadb';
$CFG->dbname    = 'bitnami_moodle';
$CFG->dbuser    = 'bn_moodle';
$CFG->dbpass    = '<MARIADB_PASSWORD>';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '',
  'dbcollation' => 'utf8mb4_general_ci',
);

$CFG->wwwroot   = 'https://kub.reto2telematicakubernetes.online';
$CFG->dataroot  = '/bitnami/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 02775;

require_once(__DIR__ . '/lib/setup.php');
```

#### 4. Volver a copiar el archivo modificado al contenedor
```bash
kubectl cp ./moodle-deployment/config.php <POD_NAME>:/bitnami/moodle/config.php
```

### Comandos de gestión y monitoreo

#### 1. Comandos para verificar el estado del clúster y recursos
```bash
kubectl get pods
kubectl get svc
kubectl get pvc
kubectl describe pod <POD_NAME>
kubectl logs <POD_NAME>
```

#### 2. Comandos para gestionar el despliegue
- Eliminar despliegues, servicios, PVC, y pods cuando sea necesario.

```bash
kubectl delete deployment my-moodle
kubectl delete svc my-moodle
kubectl delete pvc --selector=app.kubernetes.io/name=moodle
kubectl delete pod <POD_NAME> --grace-period=0 --force
```

### Acceso a Moodle en Producción

- Una vez completada la configuración, Moodle estará accesible en el dominio [https://kub.reto2telematicakubernetes.online](https://kub.reto2telematicakubernetes.online) con un certificado SSL válido gestionado por Cert-Manager.

---

## 5. Descripción de archivos de configuración del ambiente (ubicados en `moodle-deployment`)

1. `/moodle-deployment/efs-storageclass.yaml` - Clase de almacenamiento para EFS.
2. `/moodle-deployment/letsencrypt-issuer.yaml` - Emisor de certificados Let's Encrypt para Cert-Manager.
3. `/moodle-deployment/moodle-certificate.yaml` - Certificado SSL para Moodle.
4. `/moodle-deployment/moodle-deployment.yaml` - Despliegue de Moodle.
5. `/moodle-deployment/moodle-ingress.yaml` - Configuración de Ingress para Moodle.
6. `/moodle-deployment/moodle-service.yaml` - Servicio de Moodle.
7. `/moodle-deployment/moodle-values.yaml` - Valores adicionales de configuración de Moodle.
8. `/moodle-deployment/config.php` - Archivo de configuración para Moodle (para ajustes manuales).
9. `/moodle-deployment/efs-pv.yaml` - Volumen persistente para EFS.
10. `/moodle-deployment/efs-pvc.yaml` - Reclamo de volumen persistente para EFS.
11. `/moodle-deployment/nginx-deployment.yaml` - Despliegue de NGINX para balanceo de carga.
12. `/moodle-deployment/nginx-service.yaml` - Servicio de NGINX.
13. `/moodle-deployment/mariadb-deployment.yaml` - Despliegue de MariaDB para la base de datos de Moodle.
14. `/moodle-deployment/mariadb-service.yaml` - Servicio de MariaDB.

---

## 6. Fotos de la ejecución del sistema:

### Distribución (Desde consola)

- Servicios  
![Imagen de WhatsApp 2024-11-01 a las 20 08 58_780a85bf](https://github.com/user-attachments/assets/0030fc2f-ad86-4b39-b1eb-ddf9a698ba37)

- Pods con replicación  
![Imagen de WhatsApp 2024-11-01 a las 20 08 58_b43b2bf4](https://github.com/user-attachments/assets/1f59c71f-9a16-41d1-991f-5eba9e987a47)

- PV  
![Imagen de WhatsApp 2024-11-01 a las 20 08 59_3ed5c25f](https://github.com/user-attachments/assets/7bd67452-f17c-48c8-85a4-a4fd01e7b956)

- PVC  
![Imagen de WhatsApp 2024-11-01 a las 20 08 59_6da87829](https://github.com/user-attachments/assets/b47f93ad-0446-4830-bb19-7f2cc06189a4)

- Acceso a moodle adentro de un pod  
![Imagen de WhatsApp 2024-11-01 a las 20 13 08_90d2df97](https://github.com/user-attachments/assets/c49a2b67-6a3e-48d3-9eaa-7036aab4ed3a)

### Distribución (Desde interfaz de AWS)

- Clúster: ReplicaSets (Réplicas de los pods)
![Imagen de WhatsApp 2024-11-01 a las 20 38 43_6ae9f0ae](https://github.com/user-attachments/assets/227f1504-ddf9-41ad-9ef1-218476c1efea)

- Nodos workers EC2 creados
![Imagen de WhatsApp 2024-11-01 a las 20 38 43_557d32d7](https://github.com/user-attachments/assets/d8ff2438-4a35-4d2b-8d2b-fbc10a6403fe)

- Clúster: Implementaciones
![Imagen de WhatsApp 2024-11-01 a las 20 38 44_3e02a94c](https://github.com/user-attachments/assets/a8c51f6f-f201-43a2-a6e9-025bb809f032)

- Pods
![Imagen de WhatsApp 2024-11-01 a las 20 38 44_58f884a4](https://github.com/user-attachments/assets/da3214c8-5c51-4521-a516-3c8c944d982e)

- Clúster: Daemons
![Imagen de WhatsApp 2024-11-01 a las 20 38 44_fb256642](https://github.com/user-attachments/assets/f44bb13c-af55-43c4-be65-72f0c473556b)

- PVCs
![Imagen de WhatsApp 2024-11-01 a las 20 38 44_7bb37b51](https://github.com/user-attachments/assets/a053c385-55c2-4bb4-b2ad-707137c8c0ea)

- PVs
![Imagen de WhatsApp 2024-11-01 a las 20 38 44_5f86a109](https://github.com/user-attachments/assets/232d6fd0-bccd-46a8-b452-4dd994123a72)

- Storage Classes (para conectar PVCs y PVs, conexión EKS-EFS)
![Imagen de WhatsApp 2024-11-01 a las 20 38 44_ee0bd674](https://github.com/user-attachments/assets/7a4b16a8-521c-43ba-ae99-507746d07b41)

- Nos acabamos los créditos :(
![Imagen de WhatsApp 2024-11-01 a las 20 39 41_7dad5ed5](https://github.com/user-attachments/assets/2b751d86-b172-49b0-a39a-44f0a19f8a3b)

### Moodle en funcionamiento

- Login
![Imagen de WhatsApp 2024-11-01 a las 20 17 41_e7a02df7](https://github.com/user-attachments/assets/02236d34-7e67-480a-afee-f4fd5bf910c7)

- Password reset
![Imagen de WhatsApp 2024-11-01 a las 20 17 41_486493ad](https://github.com/user-attachments/assets/240640c7-353c-4ecc-8468-d21b8792c9bc)

- Vista de usuario
![Imagen de WhatsApp 2024-11-01 a las 20 23 26_4e75ab2f](https://github.com/user-attachments/assets/46a40fae-d2b4-4ab0-ad30-d3e88718981c)

- Hi Admin
![Imagen de WhatsApp 2024-11-01 a las 20 25 11_fb295166](https://github.com/user-attachments/assets/964b3838-b268-4997-8687-9ec83a453b9a)

- Creación de curso
![Imagen de WhatsApp 2024-11-01 a las 20 27 06_0d14a7aa](https://github.com/user-attachments/assets/fcccebbd-2a72-434d-85ae-d51ca405ca41)

- Curso creado
![Imagen de WhatsApp 2024-11-01 a las 20 30 15_1322b66a](https://github.com/user-attachments/assets/d2e40d14-07b0-460d-bb98-8bbe1ec6fd6f)

- Secciones de curso  
![Imagen de WhatsApp 2024-11-01 a las 20 30 43_b3b3cec1](https://github.com/user-attachments/assets/a12dfaa3-70db-4545-95cd-ddc893267ef5)

---

## 7. Video:

TdeTelemática - Reto 2 - EKS-EFS - Moodle
[https://youtu.be/vtndBFc7ZzM](https://youtu.be/vtndBFc7ZzM)

---

## 8. Referencias

- **Kubernetes Documentation:** [https://kubernetes.io/docs/](https://kubernetes.io/docs/)
- **NGINX Ingress Controller Documentation:** [https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)
- **AWS EKS Documentation:** [https://docs.aws.amazon.com/eks/](https://docs.aws.amazon.com/eks/)
- **AWS EFS Documentation:** [https://docs.aws.amazon.com/efs/](https://docs.aws.amazon.com/efs/)
- **Cert-Manager Documentation:** [https://cert-manager.io/docs/](https://cert-manager.io/docs/)
- **GoDaddy Domain Management Documentation:** [https://www.godaddy.com/help/managing-dns-680](https://www.godaddy.com/help/managing-dns-680)
- **Moodle Docker Documentation:** [https://hub.docker.com/r/bitnami/moodle/](https://hub.docker.com/r/bitnami/moodle/)
- **MariaDB Documentation:** [https://mariadb.com/kb/en/documentation/](https://mariadb.com/kb/en/documentation/)
- **Persistent Volumes in Kubernetes:** [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
