

# Máquinas destino

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

# Máquina ansible
- Vamos a suponer que la máquina desde la que se va a trabajar se llama ansiblesrv y que su IP es 192.168.39.1

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/e7a769a3-e9fb-455e-b5f4-bc66c88a565a)

- Probamos que podemos conectarnos a las máquinas destino mediante SSH, utilizando el usuario "ansible":

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/f1246e32-4717-4c66-ad84-957911acac72)


- Volvemos a la máquina ansiblesrv:

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/ba095762-4bdb-4476-b28a-4f749544ee8b)

- Instalamos ansible:
```sh
apt install ansible
```

- Configuración inicial:
```sh
mkdir /etc/ansible
```

```sh
nano /etc/ansible/ansible.cfg
```

- Podríamos configurarlo para ejecutar acciones en las máquinas remotas directamente como root:
```sh
[defaults]

inventory=/etc/ansible/hosts
remote_user=root
host_key_checking=False
become=True
become_user=root
become_ask_pass=False
```

- Pero en este caso , lo vamos a configurar para ejecutar acciones en las máquinas remotas como otro usuario (por ejemplo, utilizando el usuario "ansible"):
```sh
[defaults]

inventory=/etc/ansible/hosts
remote_user=ansible
host_key_checking=False
become=True
become_user=root
become_ask_pass=False
```

- Aplicamos la configuración:
```sh
export ANSIBLE_CONFIG=/root/ansible/ansible.cfg
echo "export ANSIBLE_CONFIG=/root/ansible/ansible.cfg" >> ~/.profile
source ~/.profile
```
- Comprobamos la versión y las rutas de los archivos de configuración:
```sh
ansible --version
```

```sh
ansible 2.10.8
config file = /etc/ansible/ansible.cfg
configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
ansible python module location = /usr/lib/python3/dist-packages/ansible
executable location = /usr/bin/ansible
python version = 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
```

- Para habilitar el acceso SSH por clave pública y que no solicite password, primero generamos el par de claves:
```sh
ssh-keygen
```
- Y copiamos la clave pública en los hosts de destino, podríamos hacerlo como root (la clave pública se almacena en el archivo /root/.ssh/authorized_keys de la máquina destino):
```sh
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.39.101
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.39.102
```

- Pero nosotros lo haremos como otro usuario, por ejemplo, "ansible" (la clave pública se almacena en el archivo /home/ansible/.ssh/authorized_keys de la máquina destino):
```sh
ssh-copy-id -i /root/.ssh/id_rsa.pub ansible@192.168.39.101
ssh-copy-id -i /root/.ssh/id_rsa.pub ansible@192.168.39.102
```

- Ahora podemos crear un inventario de máquinas:
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

O puede estar en formato YAML:
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
nano /etc/ansible/ping_all.yml
```

```sh
#/etc/ansible/ping_all.yml
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
ansible-playbook ping_all.yml
```
![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/66c442ee-5c51-40f8-99a4-82e6b832a350)



```sh

```

```sh

```

```sh

```
# Instalar servidor web apache2



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
 name: Install Apache
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


```sh

```

```sh

```

```sh

```

```sh

```

```sh

```

```sh

```
