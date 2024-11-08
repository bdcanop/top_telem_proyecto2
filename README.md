# ST0263 - Tópicos Especiales de Telemática

## Integrantes:
- Simon Betancur Bedoya
- Esteban Sierra Patiño
- Daniel Cano Perez
- Miguel Angel Cabrera Osorio

## Profesor
- **Nombre:** Edwin Nelson Montoya Munera

# Proyecto 2
#
# 1. breve descripción de la actividad
#
Se desplegó un CMS Wordpress en un clúster de alta disponibilidad en Kubernetes desplegado como IaaS (Infrastructure as a Service) montado en diferentes máquinas virtuales, con la distribución microk8s. Se consideraron además los volúmenes compartidos y la capa de acceso al clúster, Ingress.

Para el Load Balancer se instaló nginx y se configuró con un objeto Ingress que ayuda a administrar el acceso externo proporcionando reglas de enrutamiento http/s a los servicios dentro del clúster. Además están las dos instancias de WordPress y la de la base de datos, en este caso MySQL, y finalmente se implementó un servicio de almacenamiento distribuido NFS (Network File System).

## 1.1. Que aspectos cumplió o desarrolló de la actividad propuesta por el profesor
- Clúster instalado en nube con mikrok8s.
- Almacenamiento en la capa de datos haciendo uso de una base de datos MySQL.
- Almacenamiento de archivos distribuido implementado haciendo uso del NFS dentro del mismo clúster.
- Conexion de servicios de wordpress con el servicio NFS.

## 1.2. Que aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor
- Certificado SSL para habilitar la conexión cifrada (HTTPS).
- Redireccionamiento del trafico de red hacia la pagina


# 2. información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas.

![WhatsApp Image 2024-11-05 at 23 11 21_ff463125](https://github.com/user-attachments/assets/29006bde-8b9e-4721-9e29-677a5b0259f9)

# 3. Descripción del ambiente de desarrollo y técnico.

IPs Privadas:

Instancias:
- Master: 172.31.0.110
- Worker 1: 172.31.4.44
- Worker 2: 172.31.13.74
- NFS: 172.31.10.138

IPs Publicas:

Instancias:
- Master: 54.210.188.206
- Worker 1: 3.87.170.154
- Worker 2: 100.26.204.112
- NFS: 3.81.14.117

# 4. Guía paso a paso de instalación y configuración

La guía paso a paso de este proyecto se encuentra en el archivo ```PasoAPaso.md``` subido en este repositorio. De igual forma, se puede acceder a este desde el enlace a continuación:

https://github.com/bdcanop/top_telem_proyecto2/blob/main/PasoAPaso.md

# 5. Video de Sustentación
https://eafit-my.sharepoint.com/:v:/g/personal/sbetancurb_eafit_edu_co/ETxzjydut6ZFggexEK9OeAgBsiQG3skwr7jE1B9mUKWraQ?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0%3D&e=GYwrZm
