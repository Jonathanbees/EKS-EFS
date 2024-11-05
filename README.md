# Despliegue de Moodle utilizando EKS-EFS (Reto 2)

## Estudiante(s): 
Jonathan Betancur, jbetancur3@eafit.edu.co  
Esteban Vergara Giraldo, evergarag@eafit.edu.co  

## Profesor: 
Alvaro Enrique Ospina SanJuan, aeospinas@eafit.edu.co  

---

## 1. Breve descripción

En el proyecto, desplegamos Moodle en un clúster Kubernetes de alta disponibilidad en la nube, utilizando la imagen de Moodle en un entorno que incluye autoescalado y un balanceador de carga NGINX. Configuramos un dominio personalizado con certificado SSL y añadimos servidores dedicados para la base de datos y el almacenamiento de archivos (NFS). Este despliegue en AWS EKS asegura escalabilidad y rendimiento óptimo para el LMS.

Gracias por compartir las imágenes. Tras revisarlas, se puede hacer una interpretación general del estado del despliegue y cumplir con los puntos solicitados. Aquí está el resumen adaptado a tu plantilla:

### 1.1. Aspectos cumplidos o desarrollados:

- Se implementó el despliegue de Moodle en un clúster de alta disponibilidad en Kubernetes, utilizando la imagen de Moodle y configurando un dominio personalizado: [https://kub.reto2telematicakubernetes.online](https://kub.reto2telematicakubernetes.online).
- Configuración de un balanceador de carga NGINX para distribuir el tráfico a nivel de aplicación.
- Implementación de un sistema de almacenamiento distribuido mediante un servidor NFS, separado de la capa de aplicación, cumpliendo con los requisitos de almacenamiento externo.
- Configuración de una base de datos MariaDB externa en el clúster, asegurando la disponibilidad y rendimiento para las necesidades de Moodle.
- Se aplicó un certificado SSL para garantizar la seguridad en las conexiones al dominio.
- El sistema se implementó con un enfoque de autoescalado, permitiendo el ajuste dinámico de recursos según la demanda.

### 1.2. Aspectos NO cumplidos:

- Aunque se completaron los aspectos funcionales del despliegue, los archivos CSS de Moodle no se cargaron correctamente, afectando la apariencia visual de la interfaz.
- Nos gastamos todos los créditos de AWS, por lo que la página actualmente no está activa.
  
---

## 2. Información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas

- **Arquitectura distribuida:** Despliegue en un clúster Kubernetes de alta disponibilidad, distribuyendo las cargas de trabajo entre múltiples nodos y garantizando la resiliencia mediante réplicas en la base de datos y el sistema de archivos. Inspirada en arquitecturas de alta disponibilidad como GFS y HDFS.
- **Balanceador de carga NGINX:** Configurado en la capa de aplicación para distribuir el tráfico de manera uniforme entre las réplicas de Moodle, optimizando la escalabilidad y disponibilidad.
- **Separación de almacenamiento y base de datos:** Implementación de un servidor NFS para el almacenamiento de archivos y una base de datos externa MariaDB para el manejo de datos de Moodle, asegurando un manejo eficiente y seguro de los recursos de almacenamiento.
- **Certificado SSL y dominio personalizado:** Implementación de HTTPS mediante un certificado SSL para mejorar la seguridad en las conexiones, vinculado al dominio [https://kub.reto2telematicakubernetes.online](https://kub.reto2telematicakubernetes.online).
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
  - **AWS Certificate Manager (ACM):** Para gestionar el certificado SSL y asegurar las conexiones HTTPS.
  - **AWS Route 53:** Para la configuración del dominio personalizado y enrutamiento de DNS.
  
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

3. **Configuración de Certificados SSL con AWS Certificate Manager:**
   - Solicitar un certificado para el dominio y vincularlo con el Ingress en Kubernetes para habilitar HTTPS.

4. **Configuración del dominio en Route 53:**
   - Crear un registro A en Route 53 que apunte al balanceador de carga de AWS, asegurando el acceso externo al dominio configurado.

5. **Verificación del despliegue:**
   - Comprobar el estado de los pods y servicios en Kubernetes para confirmar que todos los componentes (Moodle, MariaDB, NGINX Ingress) estén en estado "Running."

   ```bash
   kubectl get pods
   kubectl get svc
   ```

---

## 4. Configuración y despliegue

---

## 5. Fotos de la ejecución del sistema:

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

## 6. Video:

(Lo grabaremos el jueves y te lo mandaremos por el correo institucional)

---

## Referencias:
- **Kubernetes Documentation:** [https://kubernetes.io/docs/](https://kubernetes.io/docs/)
- **NGINX Ingress Controller Documentation:** [https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)
- **AWS EKS Documentation:** [https://docs.aws.amazon.com/eks/](https://docs.aws.amazon.com/eks/)
- **AWS EFS Documentation:** [https://docs.aws.amazon.com/efs/](https://docs.aws.amazon.com/efs/)
- **AWS Certificate Manager Documentation:** [https://docs.aws.amazon.com/acm/](https://docs.aws.amazon.com/acm/)
- **AWS Route 53 Documentation:** [https://docs.aws.amazon.com/route53/](https://docs.aws.amazon.com/route53/)
- **Moodle Docker Documentation:** [https://hub.docker.com/r/bitnami/moodle/](https://hub.docker.com/r/bitnami/moodle/)
- **MariaDB Documentation:** [https://mariadb.com/kb/en/documentation/](https://mariadb.com/kb/en/documentation/)
- **Persistent Volumes in Kubernetes:** [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
