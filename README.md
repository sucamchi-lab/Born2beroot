*Este proyecto ha sido creado como parte del currículo de 42 por scamlett.*

# Born2beroot

## Descripción


## Información del sistema

- **OS:** Debian 
- **Virtualización:** VirtualBox 
- **Versión:** Debian 13, for 64-bit PC (amd64) debian-13.5.0-amd64-netinst.iso.
- **Hostname:** scamlett42
- **Username:** scamlett
- **Sudo policy:** 
- **Password policy:** 
- **UFW/Firewall rules:** 
- **SSH port:** 4242
- **Disk layout:** 

## Decisiones de arquitectura


### Debian vs ~~Rocky~~ :

Elegí Debian por la amplitud de su documentación, que acelera la resolución de problemas. Debian es también la base de muchas otras distros, lo que hace que el conocimiento sea transferible. 


### AppArmor vs ~~SELinux~~ :

AppArmor viene integrado con Debian y sus perfiles son legibles directamente como texto en `/etc/apparmor.d/`.


### UFW vs ~~firewalld~~ :

UFW (Uncomplicated Firewall) es el frontend estándar en Debian. Su sintaxis es explícita y sencilla, lo que facilita entender las reglas sin documentación adicional.

### VirtualBox vs ~~UTM~~ :

Al estar trabajando con Linux, la decisión es sencilla. UTM está diseñado para MacOS.


## Instalación

1. Base installation
2. Partitioning/LVM
3. Users/groups/sudo
4. SSH and firewall
5. Monitoring script/cron


## Recursos


- Se ha usado la IA para dar formato al README y resolver dudas.
