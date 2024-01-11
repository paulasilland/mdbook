# CREACIÓ D’USUARIS

Per a crear els usuaris hem utilitzat un fitxer dit “create_users” el cual crearà els usuaris dits a un fitxer dit “users.txt” amb la seva contrasenya.

El primer que fará és crear-ne els usuaris amb la comanda:

useradd -m -s /bin/bash “usuari”

Un cop l’hagi creat el possarà al grup del wheel per a poder-ne’l otorgar certs privilegis d’usuari:

usermod "usuari" -aG wheel

Després de crear-ne els usuaris, els assignarà una contrasenya aleatoria, que només serà d’un sol ús. Per temes de guardar-ne aquesta contrasenya temporal, l’afegirem amb el nom d’usuari a un fitxer que és dirà “temporal_passwd.txt”.

Primer crearà aquesta contrasenya i se l’assignarà a aquell usuari:

password=$(openssl rand -base64 12)
echo $user:$password | chpasswd

Després mostrarà per pantalla el nom de l’usuari i l’afegirà al nostre fitxer de contrasenyes temporals:

echo "Usuari: usuari created"
echo -e "Usuari: usuari" >> scripts/temporal_passwd.txt

A continuació mostrarà la contrasenya temporal i tambè l’afegirà al fitxer per a una futura consulta:

echo "Password created: contrasenya"
echo -e "Password: contrasenya" >> scripts/temporal_passwd.txt

Llavors li direm que aquesta contrasenya serà d’un sol ús, un cop es registri, només la podrà utilitzar-ne un cop, ja que quan es loguegi, li farà cambiar la contrasenya per una altra.

Important aclarar que aquesta contrasenya nova haurà de ser-hi lo més segura posible, degut a un nou filtre del PAM per a contrasenyes més segures. Es podría desabilitar però no ho farem perque volem mantenir una certa seguretat dels nostres usuaris.

La comanda per a forçar a que cambii la contrasenya que usarem és la següent:

passwd -e "usuari"

A més a més afegirem el nostre nou usuari al grup jupytherhub:

usermod -a -G jupyterhub "usuari"

CODI UTILITZAT PER A CREATE_USERS:

#!/bin/bash

# Creem els usuaris
while read user; do
        useradd -m -s /bin/bash "$user"
        usermod "$user" -aG wheel
done < scripts/users.txt

# Assignem una password aleatoria
while read user; do
    password=$(openssl rand -base64 12)
    echo $user:$password | chpasswd
    echo "Usuari: $user created"
    echo -e "Usuari: $user" >> scripts/temporal_passwd.txt
    echo "Password created: $password"
    echo -e "Password: $password" >> scripts/temporal_passwd.txt
    passwd -e "$user"
done < scripts/users.txt

# Assignem al grup jupyterhub
while read user; do
    usermod -a -G jupyterhub "$user"
done < scripts/users.txt

ELIMINACIÓ D’USUARIS

Per a eliminar els usuaris primer matarem tots els processos que encara tinguin oberts amb un “killall”, això ho hem de fer, perque si no, no ens deixarà eliminar-hi segons que usuaris que estiguin o hagin estat actius:

killall -u "usuari"

Ara procedirem a eliminar l’usuari per complert amb la comanda “userdel -r”:

userdel -r "usuari"

Per a mostrar-ne que ha sigut eliminat correctament l’usuari, mostrarem un missatge d’error amb “echo”:

echo "usuari deleted"

Finalment per a quan vulguem crear usuaris de nou i que no s’ompli després de les dades anteriors, utilitzarem la comanda “truncate -s 0” per a buidar el nostre fitxer “temporal_passwd.txt” i deixar-lo net:

truncate -s 0 scripts/temporal_passwd.txt

CODI UTILITZAT PER A DELETE_USERS:

#!/bin/bash

# Eliminem els usuaris
while read user; do
    killall -u "$user";
    userdel -r "$user";
    echo "$user deleted";
done < scripts/users.txt

# Buidem el nostre fitxer completament
truncate -s 0 scripts/temporal_passwd.txt


