*Este proyecto ha sido creado como parte del currículo de 42 por scamlett.*

# Born2beroot

## Descripción

Este proyecto consiste en virtualizar un servidor desde cero, usando Virtualbox y una imagen `iso` de Debian (o Rocky) como base, y configurar los ajustes según las especificaciones estrictas del subject, usando únicamente la terminal y sin interfaz gráfica.

## Información del sistema

- **OS:** Debian 
- **Virtualización:** VirtualBox 
- **Versión:** Debian 13, for 64-bit PC (amd64) debian-13.5.0-amd64-netinst.iso.
- **Hostname:** scamlett42
- **Username:** scamlett
- **Formato del disco duro:** Crear al menos 2 particiones encriptadas usando LVM.
- **Reglas sudo:**
- Además del usuario de raíz, debo tener un usuario presente con mi nombre de login.
- El usuario debe pertenecer a los grupos `sudo` y `user42`. 
- Autenticación sudo debe estar limitada a 3 intentos en caso de contraseña incorrecta.
- Se debe mostrar un mensaje personalizado al introducir una contraseña incorrecta.
- Cada acción sudo debe logearse en el directorio `/var/log/sudo/`
- Se debe activar TTY (terminal que permite al usuario interactuar directamente con el sistema operativo)
- Restringir determinados directorios para que solo se puedan acceder mediante `sudo`.
- **Reglas password:**
- La contraseña debe caducar cada 30 días.
- Deben pasar un mínimo de 2 días entre cambios de contraseñas.
- El usuario debe ser avisado 7 días antes de que caduque su contraseña.
- La contraseña debe tener al menos 10 caracteres, una mayúscula, una minúscula y un número. 
- No debe contener 3 caracteres idénticos consecutivos.
- No puede incluir el nombre de usuario.
- (Sólo aplica a usuarios básicos, no al root) Debe contener al menos 7 caracteres que no formaban parte de la anterior contraseña.
- **UFW/Firewall rules:** Activar UFW y solo crear una excepción para el puerto 4242. No debe ser posible conectarse usando SSH como root. UFW debe estar activo al lanzar la máquina virtual.
- **SSH port:** 4242

## monitoring.sh

Así se debe configurar el script de bash para que muestre la información necesaria, formateado de forma idéntica al ejemplo en el subject:

```bash
#!/bin/bash

# arquitectura
arch=$(uname -a)

# CPU físicos
cpuf=$(grep "physical id" /proc/cpuinfo | wc -l)

# CPU virtuales
cpuv=$(grep "processor" /proc/cpuinfo | wc -l)

# RAM
ram_total=$(free --mega | awk '$1 == "Mem:" {print $2}')
ram_use=$(free --mega | awk '$1 == "Mem:" {print $3}')
ram_percent=$(free --mega | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')

# disco
disk_total=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_t += $2} END {printf ("%.1fGb\n"), disk_t/1024}')
disk_use=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} END {print disk_u}')
disk_percent=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} {disk_t+= $2} END {printf("%d"), disk_u/disk_t*100}')

# carga CPU
cpul=$(vmstat 1 2 | tail -1 | awk '{printf $15}')
cpu_op=$(expr 100 - $cpul)
cpu_fin=$(printf "%.1f" $cpu_op)

# úlimo boot
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')

# LVM si/no
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi)

# conexiones TCP
tcpc=$(ss -ta | grep ESTAB | wc -l)

# USER LOG
ulog=$(users | wc -w)

# NETWORK
ip=$(hostname -I)
mac=$(ip link | grep "link/ether" | awk '{print $2}')

# SUDO
cmnd=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

# display mostrado en pantalla
wall "	Architecture: $arch
	CPU physical: $cpuf
	vCPU: $cpuv
	Memory Usage: $ram_use/${ram_total}MB ($ram_percent%)
	Disk Usage: $disk_use/${disk_total} ($disk_percent%)
	CPU load: $cpu_fin%
	Last boot: $lb
	LVM use: $lvmu
	Connections TCP: $tcpc ESTABLISHED
	User log: $ulog
	Network: IP $ip ($mac)
	Sudo: $cmnd cmd"
```
Posteriormente, se configura `crontab` para mostrar la información cada 10 minutos, añadiendo al final la siguiente línea:
```bash
sudo crontab -u root -e
```

`*/10 * * * * sh /home/scamlett/monitoring.sh`

## Decisiones de arquitectura


### Debian vs ~~Rocky~~ :

Elegí Debian por la amplitud de su documentación, que acelera la resolución de problemas. Debian es también la base de muchas otras distros, lo que hace que el conocimiento sea transferible. 


### AppArmor vs ~~SELinux~~ :

AppArmor viene integrado con Debian y sus perfiles son legibles directamente como texto en `/etc/apparmor.d/`.


### UFW vs ~~firewalld~~ :

UFW (Uncomplicated Firewall) viene por defecto con Debian. Su sintaxis es explícita y sencilla.

### VirtualBox vs ~~UTM~~ :

Al estar trabajando con Linux, la decisión es sencilla. UTM está diseñado para MacOS.


## Instalación

1. Descargar el repositorio
2. Abrir Virtualbox y comprobar que `signature.txt` coincide con la firma de la máquina virtual evaluada.
3. Lanzar la máquina virtual con `Start` y seguir la hoja de evaluación del proyecto.

## Recursos
- https://github.com/Vikingu-del/Born2beRoot
- https://github.com/chlimous/42-born2beroot_guide
- Se ha usado la IA para dar formato al README y resolver dudas.

## Diferencia entre apt y aptitude:

Ambos son herramientas para manejar paquetes en Debian, pero tienen ciertas diferencias:

APT: Manejo paquetes desde la CLI, ligero y raṕido. Low level.

Aptitude: Interfaz más interactiva, muestra más información de los paquetes, más intuitivo a nivel de usuario. Higher level.

## ¿Qué es AppArmor?
AppArmor es un módulo de seguridad de Linux que proporciona seguridad de Control de Acceso Obligatorio (MAC). 

### Características principales:

- Restringe las capacidades de los programas y su acceso a los recursos del sistema.

- Usa perfiles de seguridad para definir a qué recursos pueden acceder las aplicaciones.

- Controla el acceso según las rutas de los archivos.

- Las aplicaciones solo pueden acceder a los recursos explícitamente permitidos.

- Funciona junto con los permisos de archivo estándar de Linux.

## ¿Qué es LVM?
LVM (Logical Volume Manager) es un framework de mapeo de dispositivos que proporciona gestión de volúmenes lógicos para el kernel de Linux. 

### Beneficios principales:

- Redimensionamiento dinámico de particiones sin desmontar
- Gestión sencilla del almacenamiento en múltiples dispositivos físicos
- Creación de snapshots para copias de seguridad.

### Estructura:

- Volúmenes Físicos (PV): Discos duros físicos o particiones.

- Grupos de Volúmenes (VG): Conjuntos de volúmenes físicos.

- Volúmenes Lógicos (LV): Particiones virtuales que pueden abarcar múltiples volúmenes físicos.

## Comandos durante la evaluación:
TODO


