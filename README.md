# ST0263 - Tópicos Especiales de Telemática

## Integrantes:
•	Simon Betancur Bedoya
•	Esteban Sierra Patiño
•	Daniel Cano Perez
•	Miguel Angel Cabrera Osorio

## Profesor
- **Nombre:** Edwin Nelson Montoya Munera

# Proyecto 2
#
# 1. breve descripción de la actividad
#
Se desplegó un CMS Wordpress en un clúster de alta disponibilidad en Kubernetes desplegado como IaaS (Infrastructure as a Service) montado en diferentes máquinas virtuales, con la distribución microk8s. Se consideraron además los volúmenes compartidos y la capa de acceso al clúster, Ingress.

Para el Load Balancer se instaló nginx y se configuró con un objeto Ingress que ayuda a administrar el acceso externo proporcionando reglas de enrutamiento http/s a los servicios dentro del clúster. Además están las dos instancias de WordPress y la de la base de datos, en este caso MySQL, y finalmente se implementó un servicio de almacenamiento distribuido NFS (Network File System).

## 1.1. Que aspectos cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)
Clúster instalado en nube con mikrok8s.
Almacenamiento en la capa de datos haciendo uso de una base de datos MySQL.
Almacenamiento de archivos distribuido implementado haciendo uso del NFS dentro del mismo clúster.
Conexion de servicios de wordpress con el servicio NFS.
Certificado SSL para habilitar la conexión cifrada (HTTPS).

## 1.2. Que aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor (requerimientos funcionales y no funcionales)


# 2. información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas.

# 3. Descripción del ambiente de desarrollo y técnico: lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.

## como se compila y ejecuta.
## detalles del desarrollo.
## detalles técnicos
## descripción y como se configura los parámetros del proyecto (ej: ip, puertos, conexión a bases de datos, variables de ambiente, parámetros, etc)
## opcional - detalles de la organización del código por carpetas o descripción de algún archivo. (ESTRUCTURA DE DIRECTORIOS Y ARCHIVOS IMPORTANTE DEL PROYECTO, comando 'tree' de linux)
## 
## opcionalmente - si quiere mostrar resultados o pantallazos 

# 4. Descripción del ambiente de EJECUCIÓN (en producción) lenguaje de programación, librerias, paquetes, etc, con sus numeros de versiones.

# IP o nombres de dominio en nube o en la máquina servidor.

## descripción y como se configura los parámetros del proyecto (ej: ip, puertos, conexión a bases de datos, variables de ambiente, parámetros, etc)

## como se lanza el servidor.

## una mini guia de como un usuario utilizaría el software o la aplicación

## opcionalmente - si quiere mostrar resultados o pantallazos 
