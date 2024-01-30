

# Máquinas destino

- Hay que configurar la red, las máquinas destino tienen que tener una IP conocida y accesible desde la máquina ansible.
Vamos a suponer que tenemos dos servidores destino, srv01 (192.168.39.101) y srv02 (192.168.39.102).
![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/9ee97c09-eb46-440b-a28c-f3a1827a8f18)

![imagen](https://github.com/andergl/SpainSkills2024-public/assets/52236484/861e06bd-c968-47d9-8516-5e772d57f8df)


- Hay que crear los usuarios que van a conectarse y ejecutar acciones mediante ansible.
```sh
adduser ansible
```

- Si es necesario, se puede incluir a estos usuarios en /etc/sudoers, y permitir que puedan ejecutar sudo sin que te pida contraseña:
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
  
- Instalar SSH y habilitar acceso a los usuarios que van a conectarse y ejecutar acciones mediante ansible (o deshabilitar acceso a los que no se vayan a conectar).


# Máquina ansible


Instalación:
```sh
apt install ansible
```

Configuración inicial:
```sh
mkdir /etc/ansible
```

```sh
nano /etc/ansible/ansible.cfg
```

Para ejecutar acciones en las máquinas remotas directamente como root:
```sh
[defaults]

inventory=/etc/ansible/hosts
remote_user=root
host_key_checking=False
become=True
become_user=root
become_ask_pass=False
```

Para ejecutar acciones en las máquinas remotas como otro usuario (por ejemplo, utilizando el usuario "ansible"):
```sh
[defaults]

inventory=/etc/ansible/hosts
remote_user=ansible
host_key_checking=False
become=True
become_user=root
become_ask_pass=False
```

Aplicar la configuración:
```sh
export ANSIBLE_CONFIG=/root/ansible/ansible.cfg
echo "export ANSIBLE_CONFIG=/root/ansible/ansible.cfg" >> ~/.profile
source ~/.profile
```
Comprobar la versión y las rutas de los archivos de configuración:
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

Habilitar acceso SSH por clave pública. Primero generamos el par de claves:
```sh
ssh-keygen
```
Y copiamos la clave pública en los hosts de destino, bien como root (se almacena en el archivo /root/.ssh/authorized_keys de la máquina destino):
```sh
ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.0.2.25
ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.0.2.26
ssh-copy-id -i /root/.ssh/id_rsa.pub root@10.0.2.27
```

O bien como otro usuario, por ejemplo, "ansible" (se almacena en el archivo /home/ansible/.ssh/authorized_keys de la máquina destino):
```sh
ssh-copy-id -i /root/.ssh/id_rsa.pub ansible@10.0.2.25
ssh-copy-id -i /root/.ssh/id_rsa.pub ansible@10.0.2.26
ssh-copy-id -i /root/.ssh/id_rsa.pub ansible@10.0.2.27
```


Podemos crear un inventario:
```sh
nano /etc/ansible/hosts
```
Este inventario puede estar en formato INI:
```sh
#Se puede definir un grupo
[servers]
#Alias y dirección IP
server01 ansible_hostname=server01 ansible_host=10.0.2.25
server02 ansible_hostname=server02 ansible_host=10.0.2.26
server03 ansible_hostname=server03 ansible_host=10.0.2.27

[otros]
#También se pueden poner IPs directamente
10.0.2.30
#O nombres de dominio
client07.skills.org
```

O puede estar en formato YAML:
```sh
all:
  #Se definen las máquinas
  hosts:
    server01:
      ansible_host: 10.0.2.25
      hostname: "server01"
    server02:
      ansible_host: 10.0.2.26
      hostname: "server02"
    server03:
      ansible_host: 10.0.2.27
      hostname: "server03"
  children:
    #Se puede definir un grupo
    servers:
      hosts:
        server01:
        server02:
        server03:
```

Y ya se pueden enviar ordenes, como por ejemplo, una orden a un equipo:
```sh
ansible -m shell -a "cat /etc/hosts" server01
```
![](images/ansible01.png)

Una orden con el comando sudo:
```sh
ansible -m shell -a "sudo cat /etc/shadow" server01

O una orden a varios equipos a la vez:
```sh
ansible -m shell -a "ip -c a show dev enp0s3" servers
```
![](images/ansible02.png)



![](images/ansible03.png)



![](images/ansible04.png)

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

```sh

```

```sh

```

```sh

```
