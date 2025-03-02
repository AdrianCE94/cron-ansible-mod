# Estudio del Módulo cron de Ansible

## 1. Nombre del módulo: `cron`

### Descripción
El módulo `cron` de Ansible permite gestionar tareas programadas (cron jobs) en sistemas Unix/Linux. Con este módulo puedes crear, modificar o eliminar entradas en el crontab de los usuarios del sistema, facilitando la automatización de tareas periódicas como:

- Respaldos
- Limpieza de archivos temporales
- Actualizaciones
- Monitoreo
- Ejecución de scripts en horarios específicos

## 2. Ejemplos de funcionamiento

### Ejemplo 1: Crear una tarea cron básica

**Playbook**: `ejemplo1-cron-basico.yml`

```yaml
---
# ejemplo1-cron-basico.yml
# Este playbook crea una tarea cron básica que ejecuta un script de limpieza diariamente

- name: Configuración de tarea cron básica
  hosts: all
  become: yes
  
  tasks:
    - name: Crear directorio para scripts
      file:
        path: /usr/local/scripts
        state: directory
        mode: '0755'
        
    - name: Crear script de limpieza
      copy:
        content: |
          #!/bin/bash
          echo "Iniciando limpieza de archivos temporales..."
          find /tmp -type f -mtime +7 -delete
          echo "Limpieza completada a las $(date)" >> /var/log/limpieza.log
        dest: /usr/local/scripts/limpieza_tmp.sh
        mode: '0755'
        
    - name: Configurar tarea cron para ejecutar el script diariamente a las 2 AM
      cron:
        name: "Limpieza diaria de archivos temporales"
        hour: "2"
        minute: "0"
        job: "/usr/local/scripts/limpieza_tmp.sh"
        state: present
```

**Resultado esperado**:

### Ejemplo 2: Gestión avanzada de tareas cron

**Playbook**: `ejemplo2-cron-avanzado.yml`

```yaml
---
# ejemplo2-cron-avanzado.yml
# Este playbook muestra características avanzadas del módulo cron:
# - Crear tareas para un usuario específico
# - Configurar variables de entorno
# - Usar expresiones crontab completas
# - Eliminar tareas existentes

- name: Gestión avanzada de tareas cron
  hosts: all
  become: yes
  
  tasks:
    - name: Crear usuario para tareas programadas
      user:
        name: cronuser
        comment: "Usuario para tareas cron"
        shell: /bin/bash
        state: present
        
    - name: Crear directorio para logs
      file:
        path: /var/log/cron-tasks
        state: directory
        owner: cronuser
        group: cronuser
        mode: '0755'
        
    - name: Establecer tarea cron con variables de entorno
      cron:
        name: "Backup semanal"
        user: cronuser
        weekday: "0"    # Domingo
        hour: "3"
        minute: "30"
        job: "/usr/local/scripts/backup.sh /home /backup"
        cron_file: weekly-backup
        env:
          BACKUP_OPTS: "--compress --exclude-vcs"
          LOG_LEVEL: "info"
        state: present
        
    - name: Añadir tarea con expresión crontab completa
      cron:
        name: "Monitoreo de recursos"
        user: cronuser
        cron_file: monitoring
        job: "/usr/local/scripts/monitor-resources.sh > /var/log/cron-tasks/monitor.log 2>&1"
        minute: "*/15"    # Cada 15 minutos
        state: present
        
    - name: Eliminar tarea antigua que ya no es necesaria
      cron:
        name: "Tarea obsoleta"
        state: absent
```

**Resultado esperado**:


### Ejemplo 3: Tareas cron especiales

**Playbook**: `ejemplo3-cron-reboot.yml`

```yaml
---
# ejemplo3-cron-reboot.yml
# Este playbook muestra cómo configurar tareas especiales como @reboot
# y cómo trabajar con archivos crontab separados

- name: Configuración de tareas cron especiales
  hosts: all
  become: yes
  
  tasks:
    - name: Crear script de arranque
      copy:
        content: |
          #!/bin/bash
          echo "Iniciando servicios personalizados después del arranque..."
          date >> /var/log/startup-custom.log
          # Iniciar servicio personalizado
          systemctl start mi-servicio-personalizado || echo "Error al iniciar servicio" >> /var/log/startup-custom.log
        dest: /usr/local/scripts/startup.sh
        mode: '0755'
        
    - name: Configurar tarea que se ejecute al arranque del sistema
      cron:
        name: "Iniciar servicios después del arranque"
        special_time: reboot
        job: "/usr/local/scripts/startup.sh"
        state: present
        
    - name: Configurar tarea de mantenimiento mensual
      cron:
        name: "Mantenimiento mensual"
        special_time: monthly
        job: "/usr/local/scripts/monthly-maintenance.sh"
        cron_file: maintenance
        state: present
        
    - name: Configurar tarea que se ejecute diariamente
      cron:
        name: "Actualizar base de datos local"
        special_time: daily
        job: "/usr/local/scripts/update-local-db.sh"
        state: present
        
    - name: Listar tareas cron configuradas (solo funciona en entornos interactivos)
      shell: crontab -l
      register: crontab_content
      changed_when: false
      failed_when: false
      
    - name: Mostrar tareas cron configuradas
      debug:
        var: crontab_content.stdout_lines
      when: crontab_content.rc == 0
```

**Resultado esperado**:


## Inventario de Ansible

```ini
# inventory.ini
# Reemplaza la dirección IP por la de tu servidor de pruebas

[servers]
servidor ansible_host=192.168.1.100 ansible_user=tu_usuario

# Si necesitas usar contraseña en lugar de claves SSH, descomenta la siguiente línea
# servidor ansible_host=192.168.1.100 ansible_user=tu_usuario ansible_password=tu_contraseña

# Configuración global para todos los servidores
[all:vars]
ansible_connection=ssh
# ansible_ssh_private_key_file=/ruta/a/tu/clave_privada
ansible_become=yes
ansible_become_method=sudo
# ansible_become_pass=tu_contraseña_sudo
```

## 3. Referencias

- [Documentación oficial del módulo cron de Ansible](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/cron_module.html)
- [Explicación de la sintaxis crontab (crontab guru)](https://crontab.guru/)
- [Automating Linux with cron (Red Hat blog)](https://www.redhat.com/sysadmin/automate-linux-cron)
- [Guía de administración de sistemas Linux: Cron y Crontab](https://www.redhat.com/sysadmin/linux-cron-command)
- [Best Practices for Setting Up Cron Jobs](https://www.baeldung.com/linux/crontab-understanding)

## Tabla de parámetros principales del módulo cron

| Parámetro | Descripción |
|-----------|-------------|
| `name` | Nombre descriptivo para la tarea cron (requerido) |
| `user` | Nombre del usuario cuyo crontab se modificará (por defecto: root) |
| `job` | Comando a ejecutar (requerido a menos que state=absent) |
| `state` | Si la tarea debe estar presente o ausente (present/absent) |
| `minute` | Minuto (0-59, *, */2, etc.) |
| `hour` | Hora (0-23, *, */2, etc.) |
| `day` | Día del mes (1-31, *, */2, etc.) |
| `month` | Mes (1-12, *, */2, etc.) |
| `weekday` | Día de la semana (0-6 donde 0=domingo o nombres: mon,tue,wed,thu,fri,sat,sun) |
| `special_time` | Valores especiales: reboot, yearly, annually, monthly, weekly, daily, hourly |
| `cron_file` | Archivo en /etc/cron.d donde se añadirá la tarea |
| `backup` | Crear backup antes de modificar (yes/no) |
| `env` | Diccionario de variables de entorno para la tarea |

## Instrucciones para probar en tu servidor

1. Clona este repositorio en tu máquina local, encontrarás la carpeta files con los playbooks y los scripts usados.
2. Modifica el archivo `inventory.ini` con la información de tu servidor
3. Ejecuta alguno de los ejemplos:

```bash
# Asegúrate de tener Ansible instalado
ansible-playbook -i inventory.ini ejemplo1-cron-basico.yml
ansible-playbook -i inventory.ini ejemplo2-cron-avanzado.yml
ansible-playbook -i inventory.ini ejemplo3-cron-reboot.yml
```

4. Verifica los resultados en tu servidor:

```bash
# Ver tareas cron del usuario actual
crontab -l

# Ver tareas cron de un usuario específico
sudo crontab -u cronuser -l

# Ver archivos cron creados
ls -la /etc/cron.d/
```
