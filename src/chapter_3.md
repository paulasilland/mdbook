# 3. Creació d'Usuaris

Per a crear els usuaris hem utilitzat un fitxer dit “create_users” el cual crearà els usuaris dits a un fitxer dit “users.txt” amb la seva contrasenya.

El primer que fará és crear-ne els usuaris amb la comanda:

<span style="color: yellow;">useradd -m -s /bin/bash “usuari”</span>

Un cop l’hagi creat el possarà al grup del wheel per a poder-ne’l otorgar certs privilegis d’usuari:

<span style="color: yellow;"> usermod "usuari" -aG wheel</span>

Després de crear-ne els usuaris, els assignarà una contrasenya aleatoria, que només serà d’un sol ús. Per temes de guardar-ne aquesta contrasenya temporal, l’afegirem amb el nom d’usuari a un fitxer que és dirà “temporal_passwd.txt”.

Primer crearà aquesta contrasenya i se l’assignarà a aquell usuari:

<span style="color: yellow;">  
password=$(openssl rand -base64 12) 
echo $user:$password | chpasswd  </span>

<br>Després mostrarà per pantalla el nom de l’usuari i l’afegirà al nostre fitxer de contrasenyes temporals:

<span style="color: yellow;">
echo "Usuari: usuari created"
echo -e "Usuari: usuari" >> scripts/temporal_passwd.txt</span>


<br>A continuació mostrarà la contrasenya temporal i tambè l’afegirà al fitxer per a una futura consulta:

<span style="color: yellow;"><br>
echo "Password created: contrasenya"<br>
echo -e "Password: contrasenya" >> scripts/temporal_passwd.txt</span>

Llavors li direm que aquesta contrasenya serà d’un sol ús, un cop es registri, només la podrà utilitzar-ne un cop, ja que quan es loguegi, li farà cambiar la contrasenya per una altra.

Important aclarar que aquesta contrasenya nova haurà de ser-hi lo més segura posible, degut a un nou filtre del PAM per a contrasenyes més segures. Es podría desabilitar però no ho farem perque volem mantenir una certa seguretat dels nostres usuaris.

La comanda per a forçar a que cambii la contrasenya que usarem és la següent:

<span style="color: yellow;">passwd -e "usuari"</span>

A més a més afegirem el nostre nou usuari al grup jupytherhub:

<span style="color: yellow;">usermod -a -G jupyterhub "usuari"</span>

# Codi utilitzat per a CREATE_USERS:

#!/bin/bash

<span style="color: yellow;">Creem els usuaris</span>  
while read user; do <br>
        useradd -m -s /bin/bash "$user"<br>
        usermod "$user" -aG wheel<br>
done < scripts/users.txt

<span style="color: yellow;">Assignem una password aleatoria</span>  
while read user; do<br>
    password=$(openssl rand -base64 12)<br>
    echo $user:$password | chpasswd<br>
    echo "Usuari: $user created"<br>
    echo -e "Usuari: $user" >> scripts temporal_passwd.txt<br>
    echo "Password created: $password"<br>
    echo -e "Password: $password" >> scripts/temporal_passwd.txt<br>
    passwd -e "$user"<br>
done < scripts/users.txt

<span style="color: yellow;">Assignem al grup jupyterhub</span>  

while read user; do<br>
    usermod -a -G jupyterhub "$user"<br>
done < scripts/users.txt

# Eliminació d'Usuaris

Per a eliminar els usuaris primer matarem tots els processos que encara tinguin oberts amb un “killall”, això ho hem de fer, perque si no, no ens deixarà eliminar-hi segons que usuaris que estiguin o hagin estat actius:

<span style="color: yellow;">killall -u "usuari"</span>

Ara procedirem a eliminar l’usuari per complert amb la comanda “userdel -r”:

<span style="color: yellow;">userdel -r "usuari"</span>

Per a mostrar-ne que ha sigut eliminat correctament l’usuari, mostrarem un missatge d’error amb “echo”:

<span style="color: yellow;">echo "usuari deleted"</span>

Finalment per a quan vulguem crear usuaris de nou i que no s’ompli després de les dades anteriors, utilitzarem la comanda “truncate -s 0” per a buidar el nostre fitxer “temporal_passwd.txt” i deixar-lo net:

<span style="color: yellow;">truncate -s 0 scripts/temporal_passwd.txt</span>

# Codi utilitzat per a DELETE_USERS:

#!/bin/bash

<span style="color: yellow;">Eliminem els usuaris</span>  
while read user; do<br>
    killall -u "$user";<br>
    userdel -r "$user";<br>
    echo "$user deleted";
done < scripts/users.txt

<span style="color: yellow;">Buidem el nostre fitxer completament</span>  
truncate -s 0 scripts/temporal_passwd.txt


