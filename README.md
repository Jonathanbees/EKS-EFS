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
