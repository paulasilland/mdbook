# Servidor Web (Jupyter)

Es configura un servidor web amb JupyterHub en Rocky Linux 8 per proporcionar un entorn de desenvolupament Python flexible, amb accés per SSH i SFTP als directoris de treball dels usuaris.

<span style="color: yellow;">Objectius:</span>

- Implementar JupyterHub en un servidor web.  
- Establir un sistema de fitxers LVM per a JupyterHub.
- Assegurar JupyterHub i el servidor web.
- Configurar un desplegament basat en un servei de Linux.


<span style="color: yellow;">Configuració del Sistema de Fitxers</span>

Instal.lem el paquet lvm2:

dnf install lvm2 -y

Es configura LVM per a major flexibilitat i es creen volums lògics per a diferents necessitats de JupyterHub, com executables, entorns virtuals, fitxers temporals i logs.

<span style="color: yellow;">Configuració de JupyterHub</span>

Es realitza la instal·lació de Python3, pip, Node.js, npm i configurable-http-proxy. Es crea un entorn virtual per a JupyterHub i es configuren les dependències necessàries.

Crearem un entorn virtual per a JupyterHub i l'activarem:

cd /opt/jupyterhub
python3 -m venv venv

source /opt/jupyterhub/venv/bin/activate

<span style="color: yellow;">Seguretat i Servei</span>  

Es configura un usuari dedicat jupyterhub per executar el servei sense permisos de root. S'utilitza Sudospawner per a permetre l'elevació de permissos segura per als processos d'usuari.

<span style="color: yellow;">Gestió Automàtica de Comptes d'Usuari</span>

Es desenvolupen scripts per automatitzar la creació i eliminació d'usuaris, incloent la gestió de volums lògics i assignació de passwords aleatòries.

<span style="color: yellow;">Configuració del Tallafocs</span>

Es configura firewalld amb una zona específica per a JupyterHub i es permeten els serveis necessaris.

<span style="color: yellow;">Configuració Avançada de JupyterHub</span>

Es detalla la configuració de jupyterhub_config.py, incloent la configuració de spawner, directoris i logs. També es configura el servei systemd per iniciar JupyterHub.

<span style="color: yellow;">Aïllament d'Usuaris en JupyterHub</span>  

Es configura JupyterHub per a que cada usuari tingui el seu propi entorn virtual i kernel, garantint aïllament i personalització del seu espai de treball.

