# 1. Configuración inicial de las máquinas

## 1.1. Máquinas destino

- Hay que configurar la red, las máquinas destino tienen que tener una IP conocida y accesible desde la máquina ansible.
Vamos a suponer que tenemos dos servidores destino, srv01 (192.168.39.101) y srv02 (192.168.39.102).

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/9ee97c09-eb46-440b-a28c-f3a1827a8f18)

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/861e06bd-c968-47d9-8516-5e772d57f8df)


- En ambas máquinas destino, hay que crear los usuarios que van a conectarse y ejecutar acciones mediante ansible. En nuestro caso, crearemos un usuario llamado "ansible":
```sh
adduser ansible
```

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/4e18ea61-53fd-48cc-9034-894bf9d21e13)

- Incluiremos a este usuario en /etc/sudoers, y permitiremos que pueda ejecutar sudo sin que te pida contraseña:
```sh
nano /etc/sudoers
```
```sh
....
#/etc/sudoers
#Añadir una línea por usuario:
ansible ALL=(ALL) NOPASSWD:ALL
...
```
  
- Instalamos SSH y habilitamos acceso al usuario que va a conectarse y ejecutar acciones mediante ansible (o deshabilitamos el acceso a los que no se vayan a conectar).

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/12b6942a-dd12-4992-b8bb-259e65939fc9)

## 1.2. Máquina ansible
- Vamos a suponer que la máquina desde la que se va a trabajar se llama ansiblesrv y que su IP es 192.168.39.1

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/e7a769a3-e9fb-455e-b5f4-bc66c88a565a)

### 1.2.1. SSH mediante clave pública

- Probamos que podemos conectarnos a las máquinas destino mediante SSH, utilizando el usuario "ansible":

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/f1246e32-4717-4c66-ad84-957911acac72)


- Volvemos a la máquina ansiblesrv:

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/ba095762-4bdb-4476-b28a-4f749544ee8b)


- Vamos a habilitar el acceso SSH a las máquinas srv01 y srv02 desde ansiblesrv por clave pública y que no solicite password. Para ello, primero generamos el par de claves. La clave pública se almacena en /root/.ssh/id_rsa.pub. (Esto significa que el usuario que va a tener que ejecutar las acciones en ansiblesrv es root):
```sh
ssh-keygen
```

- Y copiamos la clave pública en los hosts de destino utilizando el usuario que hemos credo en las máquinas destino y que hemos configurado en el archivo /etc/ansible/ansible.cfg. En nuestro caso, el usuario "ansible" (la clave pública se almacena en el archivo /home/ansible/.ssh/authorized_keys de la máquina destino), lo que significa que las acciones que se ejecuten en srv01 y srv02 se van a ejecutar con el usuario ansible:
```sh
ssh-copy-id -i /root/.ssh/id_rsa.pub ansible@192.168.39.101
ssh-copy-id -i /root/.ssh/id_rsa.pub ansible@192.168.39.102
```

- Podemos probar que ahora la conexión SSH es posible y que no nos pide password:

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/e05c472f-12e5-4249-b10b-30c23f6acd11)

### 1.2.2. Ansible: instalación y configuración

- La versión de Ansible actual de los repositorios de Debian (ansible 2.10.8) es bastante antigua, por lo que la instalaremos con el rpm de Ubuntu ([aquí]([url](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-debian)) el artículo oficial). Primero hay que instalar wget y gpg:
```sh
apt install wget gpg
```
- Y después, añadimos el repositorio necesario de Ubuntu e instalamos ansible:
```sh
UBUNTU_CODENAME=focal

wget -O- "https://keyserver.ubuntu.com/pks/lookup?fingerprint=on&op=get&search=0x6125E2A8C77F2818FB7BD15B93C4A3FD7BB9C367" | sudo gpg --dearmour -o /usr/share/keyrings/ansible-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/ansible-archive-keyring.gpg] http://ppa.launchpad.net/ansible/ansible/ubuntu $UBUNTU_CODENAME main" | sudo tee /etc/apt/sources.list.d/ansible.list

apt update && apt upgrade

apt install ansible
```


- Comprobamos la versión y las rutas de los archivos de configuración y listamos el contenido de /etc/ansible:
```sh
ansible --version

ls -la /etc/ansible/
```
![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/a8e8c83d-4b49-4319-bab9-8542701874d4)


- Vemos que tenemos la version ansible 2.12.10. El archivo principal de configuración es /etc/ansible/ansible.cfg y vemos que existe el archivo /etc/ansible/hosts.

- Vamos a editar el archivo /etc/ansible/ansible.cfg para la configuración inicial:
```sh
nano /etc/ansible/ansible.cfg
```

- Lo vamos a configurar para ejecutar acciones en las máquinas remotas como un usuario (por ejemplo, utilizando el usuario "ansible"):
```sh
[defaults]

inventory=/etc/ansible/hosts
remote_user=ansible
host_key_checking=False
become=True
become_user=root
become_ask_pass=False
```


- Ahora podemos crear un inventario de máquinas en el archivo /etc/ansible/hosts:
```sh
nano /etc/ansible/hosts
```
- Este inventario puede estar en formato INI:
```sh
#Se puede definir un grupo
[servers]
#Alias y dirección IP
srv01 ansible_hostname=srv01 ansible_host=192.168.39.101
srv02 ansible_hostname=srv02 ansible_host=192.168.39.102

[otros]
#También se pueden poner IPs directamente
192.168.39.105
#O nombres de dominio
client07.skills.org
```

O puede estar en formato YAML (en este ejemplo sólo se añaden los equipos del grupo servers):
```sh
all:
  #Se definen las máquinas
  hosts:
    srv01:
      ansible_host: 192.168.39.101
      hostname: "srv01"
    srv02:
      ansible_host: 192.168.39.102
      hostname: "srv02"
  children:
    #Se puede definir un grupo
    servers:
      hosts:
        srv01:
        srv02:
```

### 1.2.3. Prueba funcional

- Y ya se pueden enviar ordenes, como por ejemplo, una orden a un equipo:
```sh
ansible -m shell -a "ip -c a show enp0s3" srv01
```
![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/21fadc87-0b49-4a99-bcb7-cf67d2be5841)

- Una orden con el comando sudo:
```sh
ansible -m shell -a "sudo cat /etc/shadow | grep ansi" srv02
```
![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/4f35f5ec-7f66-46a0-96ca-55517523aa5b)

- O una orden a varios equipos a la vez:
```sh
ansible -m shell -a "ip -c a show enp0s3" servers
```
![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/8c10e53c-d9bc-4a6f-adc7-326c2f23aa10)

- Y podemos crear un playbook para probar conectividad (ping) con las másquinas en formato YAML:

```sh
mkdir /etc/ansible/playbooks

nano /etc/ansible/playbooks/ping_all.yml
```

```sh
#/etc/ansible/playbooks/ping_all.yml
- name: Testing
  hosts: servers
  tasks:
  - name: ping
    ping:
  - name: output
    debug:
      msg: "{{ hostname }}"
```
- Y podemos ejecutarlo con el comando "ansible-playbook" desde el directorio en el que está ubicado el archivo:
```sh
cd /etc/ansible/playbooks/

ansible-playbook ping_all.yml
```
![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/beba4204-b30a-4dca-9831-a402ab18f1df)

### 1.2.4. Entorno de desarrollo
- El entorno de desarrollo que del que se va a disponer está descrito en [https://github.com/andergl/SpainSkills2024-public/blob/main/Module%20A/ansible/ansible-dev-tools.md](https://github.com/andergl/SpainSkills2024-public/blob/main/Module%20A/ansible/ansible-dev-tools.md)

# 2. Tareas de ejemplo
## 2.1. Cambiar hostname
- Para cambiar el hostname de varios equipos y , podemos usar el siguiente playbook:

```sh
nano hostname.yml
```

- El playbook renombra cada equipo con el nombre {{ hostname }} que tiene definido en el archivo /etc/ansible/hosts. Esto modifica el contenido del archivo /etc/hostname de las máquinas destino. No es obligatorio, pero como el cambio no se refleja en el prompt hasta el siguiente reinicio, reinicia las máquinas:

```sh
- name: "Change hostname"
  hosts: "servers"

  tasks:

    - name: Set hostname
      hostname:
        name: "{{ hostname }}"
      become: true

    - name: "Reinicio"
      reboot:
      become: true
```

## 2.2. Instalar servidor web apache2



- En la máquina ansiblesrv, nos ubicamos en el directorio en el que van a estar los playbooks, en nuestro caso, /etc/ansible:
```sh
cd /etc/ansible
```

- Creamos un playbook llamado apache-install.yml en el directorio /etc/ansible:

```sh
nano apache-install.yml
```
- Con el siguiente contenido

```sh
- name: Install Apache
  hosts: servers
  become: true

  tasks:
  - name: Install Apache
    apt:
      name: apache2
      state: present

  - name: Start Apache
    service:
      name: apache2
      state: started
      enabled: yes
```
- Y lo ejecutamos:
```sh
ansible-playbook apache-install.yml
```

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/35bc7446-ab29-4ba3-bf28-26a4b326afa9)


## 2.3. Configurar servidor web apache2 (creación de un VirtualHost)

- En la máquina ansiblesrv, nos ubicamos en el directorio en el que van a estar los playbooks, en nuestro caso, /etc/ansible:
```sh
cd /etc/ansible/playbooks
```

- Creamos un un directorio que contenga los playbooks y los ficheros de configuración de apache:
```sh
mkdir apache
```

- Movemos el playbook de instalación a este directorio:
```sh
mv apache-install.yml apache
```
- Creamos la estructura que contendrá los ficheros de configuración del apache:
```sh
mkdir files
mkdir vars
```
- Creamos el fichero para las variables a utilizar en la configuración de nuestro sitio:
```sh
cd vars
nano default.yml
```

```sh
#/etc/ansible/playbooks/apache/vars/default.yml
---
app_user: "ansible"
http_host: "misite"
http_conf: "misite.conf"
http_port: "80"
disable_default: true
```
- Creamos el fichero de configuración de apache para crear el VirtualHost:
```sh
cd /etc/ansible/playbooks/apache/files
nano apache.conf
```

```sh
#/etc/ansible/playbooks/apache/files/apache.conf
<VirtualHost *:{{ http_port }}>
  ServerAdmin webmaster@localhost
  ServerName {{ http_host }}
  ServerAlias www.{{ http_host }}
  DocumentRoot /var/www/{{ http_host }}
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Creamos el index.html para nuestro VirtualHost:
```sh
cd /etc/ansible/playbooks/apache/files
nano index.html
```

```sh
#/etc/ansible/playbooks/apache/files/index.html
<html lang="es">
  <head>
    <title>Bienvenido a {{ http_host }}!!</title>
    <meta charset="UTF-8"/>
  </head>
  <body>
    <h1>CORRECTO! El {{ http_host }} virtual está funcionando en el servidor {{ hostname }}!</h1>
  </body>
</html>
```
- Creamos un playbook llamado apache-config.yml en el directorio /etc/ansible/playbooks/apache:

```sh
nano apache-config.yml
```
- Con el siguiente contenido

```sh
- name: Apache configuration
  hosts: servers
  become: true
  vars_files:
    - vars/default.yml

  tasks:
    - name: Create document root
      file:
        path: "/var/www/{{ http_host }}"
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'

    - name: Copy index test page
      template:
        src: "files/index.html"
        dest: "/var/www/{{ http_host }}/index.html" 

    - name: Set up Apache VirtualHost
      template:
        src: "files/apache.conf"
        dest: "/etc/apache2/sites-available/{{ http_conf }}"

    - name: Enable new site
      shell: /usr/sbin/a2ensite {{ http_conf }}
      notify: Reload Apache

    - name: Disable default Apache site
      shell: /usr/sbin/a2dissite 000-default.conf
      when: disable_default
      notify: Reload Apache

  handlers:
    - name: Reload Apache
      service:
        name: apache2
        state: reloaded

    - name: Restart Apache
      service:
        name: apache2
        state: restarted
```
- Y lo ejecutamos:
```sh
ansible-playbook apache-config.yml
```
<img width="630" alt="Captura de pantalla 2024-02-02 a las 13 01 56" src="https://github.com/andergl/SpainSkills2024-public/assets/15244568/15bb97ba-306b-4f23-ad32-b92d4eba2936">

- Probamos desde un navegador:
```sh
http://server_host o IP
```
<img width="811" alt="Captura de pantalla 2024-02-02 a las 13 20 59" src="https://github.com/andergl/SpainSkills2024-public/assets/15244568/fc55d01d-b750-4487-8ca3-9119e6cfd8ba">

<img width="811" alt="Captura de pantalla 2024-02-02 a las 13 21 19" src="https://github.com/andergl/SpainSkills2024-public/assets/15244568/9e6045fa-e698-4ce6-9995-fb7b0221b027">



