# Automatización y Despliegue de un Servicio Proxy y Caché Web (Squid) 

Este repositorio contiene la solución automatizada para el diseño, aprovisionamiento y despliegue de un servicio de infraestructura crítica enfocado en **Redes Comunitarias**. El proyecto se enfoca en el uso de metodologías modernas como la **Infraestructura como Código (IaC)**, **Gestión de Configuración** y **Contenedores**.

El servicio seleccionado corresponde a la **Categoría B (Servicios de Red): Proxy y Caché Web mediante Squid** , diseñado para optimizar la navegación web, filtrar contenido y asegurar un ahorro sustancial de ancho de banda en entornos con conectividad limitada.

---

## 🛠️ Tecnologías Obligatorias Utilizadas

* **Automatización:** Ansible para la gestión de configuración y creación de infraestructura.
* **Cloud Computing:** AWS EC2 (Capa Gratuita) para el entorno público.
* **Seguridad Cloud:** AWS Security Groups para el control estricto de puertos.
* **Herramientas de Control:** AWS CLI para la integración nativa con la API de AWS.
* **Contenedores:** Docker y Docker Compose para garantizar el aislamiento, portabilidad y reproducibilidad del servicio.
* **Infraestructura de Red Local:** Diseñado para su integración con entornos Linux y routers comunitarios OpenWrt o equipos Cisco.

---

## 📂 Estructura del Repositorio

La arquitectura de archivos del proyecto está organizada bajo la siguiente jerarquía estructural:

```text
├── ansible/
│   ├── despliegue.yml       # Playbook para configuración interna, Docker y Squid
│   ├── infraestructura.yml  # Playbook para el aprovisionamiento de recursos en AWS (EC2/SG)
│   └── inventario.ini       # Archivo de inventario estático (Cloud y Entorno Local)
├── docker/
│   ├── Dockerfile           # Definición de la imagen personalizada del contenedor Squid
│   ├── docker-compose.yml   # Orquestador del ciclo de vida del contenedor
│   └── squid.conf           # Configuración de políticas de caché y listas de control de acceso (ACL)
└── README.md                # Documentación técnica principal del proyecto
```

---

## 📋 Requisitos Previos en la Máquina Local

Antes de realizar cualquier ejecución, es obligatorio preparar el entorno de control local en tu máquina Linux Ubuntu con los siguientes comandos:

1. **Actualizar repositorios e instalar las herramientas base:**
   ```bash
   sudo apt update
   sudo apt install -y ansible python3-pip
   ```

2. **Instalar las dependencias de Python necesarias para la API de AWS:**
   ```bash
   sudo apt install -y python3-boto3 python3-botocore
   ```

3. **Instalar la colección oficial de AWS para Ansible:**
   ```bash
   ansible-galaxy collection install amazon.aws
   ```

4. **Configurar las credenciales de acceso local:**
   ```bash
   aws configure
   ```
   *(Ingresa tu Access Key, Secret Key y define la región por defecto, por ejemplo `us-east-1`)*.

---

## 🚀 Guía de Ejecución Paso a Paso

### [cite_start]🌐 Fase 1: Despliegue en Entorno Cloud (AWS) [cite: 49]

El aprovisionamiento y despliegue en la nube se realiza de forma 100% automatizada, dividiéndose en dos etapas controladas por los playbooks dentro de la carpeta `ansible/`:

1. **Creación de la Infraestructura:**
   Ejecuta el archivo encargado de interactuar con AWS para crear el grupo de seguridad e instanciar el servidor virtual EC2:
   ```bash
   ansible-playbook ansible/infraestructura.yml
   ```
   *Toma nota de la dirección IP pública que se imprima en la terminal al finalizar la tarea.*

2. **Configuración del Host e Inventario:**
   Abre el archivo `ansible/inventario.ini` y define los parámetros del host dentro del grupo cloud:
   ```ini
   [servidores_cloud]
   <IP_PUBLICA_AWS> ansible_user=ubuntu ansible_ssh_private_key_file=/ruta/a/tu/llave.pem
   ```

3. **Despliegue del Servicio:**
   Ejecuta el playbook principal para instalar de forma automatizada Docker, transferir los archivos de configuración y levantar el servicio proxy mediante Docker Compose:
   ```bash
   ansible-playbook -i ansible/inventario.ini ansible/despliegue.yml
   ```

###  Fase 2: Despliegue en Entorno Local (Red Comunitaria) 

Para dar cumplimiento a la replicación en infraestructuras autónomas y descentralizadas locales usando OpenWrt o Cisco:

1. Abre el archivo `ansible/inventario.ini`.
2. Comenta el bloque de `[servidores_cloud]` usando el carácter `#`.
3. Descomenta y edita la sección local con los datos de direccionamiento IP de tu servidor de laboratorio local:
   ```ini
   [servidores_locales]
   192.168.1.100 ansible_user=tu_usuario_local ansible_ssh_pass=tu_contraseña
   ```
4. Lanza de nuevo el despliegue automático enfocado en la red interna:
   ```bash
   ansible-playbook -i ansible/inventario.ini ansible/despliegue.yml
   ```

---

##  Verificación del Funcionamiento y Auditoría de la Caché 
Para comprobar el correcto flujo de conectividad, mapeo de puertos y validar las ventajas de la caché local en entornos comunitarios:

1. **Auditoría de Logs en Vivo:**
   Conéctate al servidor (Cloud o Local) a través de SSH y visualiza los registros de peticiones de navegación de Squid en tiempo real:
   ```bash
   sudo docker exec -it squid_proxy tail -f /var/log/squid/access.log
   ```

2. **Prueba de Tráfico desde un Cliente:**
   En una terminal independiente dentro de la red del cliente, fuerza peticiones HTTP a través del puerto del proxy (`3128`):
   ```bash
   curl -x http://<IP_DE_TU_SERVIDOR>:3128 -I [http://example.com](http://example.com)
   ```

3. **Interpretación de Códigos Técnicos:**
   * `TCP_DENIED/403`: Petición bloqueada por reglas de seguridad perimetral configuradas. Asegúrate de añadir tu IP cliente dentro de las ACL autorizadas en `docker/squid.conf`.
   * `TCP_MISS/200`: El recurso no estaba indexado en el almacenamiento del proxy local. Squid procesó la salida a la red WAN para obtenerlo por primera vez.
   * `TCP_HIT/200`: **¡Caché operativa con éxito!** El recurso se sirvió de forma local e instantánea, reduciendo a cero el consumo de ancho de banda externo en esta consulta.

---

##  Restricciones de Seguridad Críticas 

En cumplimiento estricto con los criterios de evaluación y penalizaciones del proyecto final:
* **PROHIBIDO** subir al repositorio público archivos con extensiones de llave privada (`.pem`).
* **PROHIBIDO** exponer de forma explícita credenciales fijas como *Access Keys*, *Secret Keys* o tokens temporales de AWS dentro de los playbooks.

---

## 👥 Integrantes
* Andrés Fernando Basto Bejarano
* Melissa Marian Martinez Corredor

*Administración de Redes — Universidad Católica de Colombia* 
