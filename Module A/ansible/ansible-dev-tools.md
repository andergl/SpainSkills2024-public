# Entorno de desarrollo de Ansible

Para empezar a desarrollar nuestros playbooks de una manera sencilla vamos a hacer uso de algunas herramientas que nos ayuden a escribir código y comprobar que este sea de calidad.

- Una interfaz de desarrollo sencilla (vscode) con plugins específicos
- Una herramienta de calidad de código (ansible-lint)
- Las collecciones que necesitemos para nuestro código

## Visual Studio Code

Visual Studio Code (también llamado VS Code) es un editor de código fuente desarrollado por Microsoft para Windows, Linux, macOS y Web. Incluye soporte para la depuración, control integrado de Git, resaltado de sintaxis, finalización inteligente de código, fragmentos y refactorización de código. También es personalizable, por lo que los usuarios pueden cambiar el tema del editor, los atajos de teclado y las preferencias. Es gratuito y de código abierto,1​2​ aunque la descarga oficial está bajo software privativo e incluye características personalizadas por Microsoft. Podemos descargar el paquete que se ajuste a nuestro sistema operativo en la [web oficial][vscode pkg].

![](https://user-images.githubusercontent.com/35271042/118224532-3842c400-b438-11eb-923d-a5f66fa6785a.png)

Una vez lo tengamos instalado vamos a añadir los plugins necesarios para poder trabajar de una manera sencilla con la máquina de ansible.

Para ello vamos la pestaña de "Extensiones" en la parte derecha, con el símbolo [imagen_extensiones] y buscamos las siguientes extensiones.

[vscode pkg]: https://code.visualstudio.com/download



#### Ansible

Esta extensión añade soporte al lenguaje de Ansible a VSCode. La extensión funciona con asociación de los documentos con formato .yml o .yaml

Una vez instalada la extensión nos ayudará a la hora de crear nuestros playbooks, autorrellenando los nombres de los módulos, mostrando la documentación de los mismos o sugiriendo las opciones posibles. Podemos consultar toda la [documentación][vscode ansible] del módulo en el marketplace de VS Code.

![](https://github.com/ansible/vscode-ansible/raw/HEAD/images/smart-completions.gif)

> [!TIP]
> El atajo de teclado para activar la opción de sugerencia es "ctrl+space".

También podéis echarle un ojo a este [vídeo][ansible video] donde se explica todo su funcionamiento por parte de unos de sus autores.

[ansible video]: https://www.youtube.com/watch?app=desktop&v=iI6cSvL87xY
[vscode ansible]: https://marketplace.visualstudio.com/items?itemName=redhat.ansible

#### Remote SSH

Si estamos usando una máquina externa en vez de nuestra estación de trabajo para desarrollar, podemos conectarnos a ella y escribir el código directamente en ella, de manera que todo el software necesario para ejecutar los playbooks estará en la máquina destino.

La extensión te permite utilizar cualquier máquina remota con un servidor SSH como entorno de desarrollo. 

![](https://microsoft.github.io/vscode-remote-release/images/ssh-readme.gif)

[vscode ssh]: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh

## Ansible Lint

Ansible Lint es una herramienta de línea de comandos para crear playbooks, roles y colecciones dirigidas a cualquier usuario de Ansible. Su objetivo principal es promover prácticas, patrones y comportamientos probados y, al mismo tiempo, evitar errores comunes que pueden provocar errores o dificultar el mantenimiento del código.

También se supone que Ansible lint ayuda a los usuarios a actualizar su código para que funcione con versiones más nuevas de Ansible. Por este motivo recomendamos utilizarlo con la versión más reciente de Ansible, incluso si la versión utilizada en producción puede ser anterior. Sus reglas son el resultado de las contribuciones de la comunidad y siempre se pueden desactivar individualmente o por categoría por cada usuario.

Se integra con VSCode, de esta manera cuando guardamos un fichero ejecutará las reglas del linter de manera automática.

![](https://github.com/ansible/vscode-ansible/raw/HEAD/images/ansible-lint.gif)

El proyecto Ansible Galaxy utiliza este linter para calcular puntuaciones de calidad para el contenido aportado por Galaxy Hub. Esto no significa que esta herramienta solo esté dirigida a aquellos que quieran compartir su código. Archivos como galaxy.yml o secciones como galaxy_info dentro de meta.yml ayudan con la documentación y el mantenimiento, incluso para roles o colecciones no publicadas.

El proyecto fue iniciado originalmente por @willthames y desde entonces ha sido adoptado por el equipo de Ansible Community. Su desarrollo es puramente impulsado por la comunidad manteniendo comunicaciones permanentes con otros equipos de Ansible.

#### Instalación 

Puede instalar la versión más reciente de Ansible-lint con el administrador de paquetes Python pip3 o pipx. Utilice pipx para aislar Ansible-lint de su entorno Python actual como alternativa a la creación de un entorno virtual.

```sh
# Esto también instala ansible-core si todavía no está instalado
pip3 install ansible-lint
```

Puedes consultar la [documentación oficial][lint doc] para más información sobre como configurarlo.

[lint doc]: https://ansible.readthedocs.io/projects/lint/

## Colecciones

Todo el código de Ansible se basa en la ejecución de instrucciones que se cargan en forma de módulos. Estos módulos están desarrollados en Python y empaquetados en forma de collecciones, las cuales incluyen el código y las dependencias necesarias para que se ejecute, así como roles o plugins que pueden simplificar su uso.

El repositorio donde se guardan las colecciones desarrolladas por la comunidad se llama [Ansible Galaxy][galaxy url] y en él encontraremos las colecciones ordenadas por nombre o utilidad. 

Una de las más útiles, y que puede que necesitéis durante la competición, están bajo el [_namespace Community_][comm ns]. Aquí encontraremos collecciones genericas, de bases de datos o de Windows, por poner algunos ejemplos.

Cada colección tiene su propia documentanción de uso, con todas las opciones disponibles y algunos ejemplos.

Para instalar una colección haremos uso del comando _ansible-galaxy_.

```sh
ansible-galaxy collection install community.general
```

Una vez instalada la coleccion en nuestra máquina de desarrollo podremos usar los módulos incluidos en ella en nuestro playbook mediante su FQCN o haciendo un include de la colección.

```yml
---
- hosts: localhost
  collections: 
    - community.general
  tasks:
    - name: Create a zip archive of /path/to/foo
      community.general.archive:
        path: /path/to/foo
        format: zip
```

> [!NOTE]
> El FQCN (Fully Qualified Collection Name) es recomendable para evitar posibles duplicidades con el nombre del módulo. Por ejemplo, existe un módulo llamado _file_ en la colección incluída en ansible-core (ansible.builtin) pero también uno en la coleción de Cisco o en la de F5.

[galaxy url]: https://galaxy.ansible.com/
[comm ns]: https://galaxy.ansible.com/ui/namespaces/community/

## Otros recursos

Hay algunos [laboratorios interactivos][ansible labs] creados por Red Hat para poder practicar el desarrollo de Ansible y otros conceptos avanzados.

[ansible labs]: https://www.redhat.com/en/interactive-labs/ansible
