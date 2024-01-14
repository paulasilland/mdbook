# Servidor web (mdbook)
En aquest apartat es descriu la configuració del servidor web basat en apache per a servir un llibre escrit en mdbook.

Teniem els següents objectius:
- Instal·lar i configurar un servidor web apache.
- Instal·lar i configurar un certificat SSL.
- Instal·lar i configurar mdbook.
- Fer una configuració segura del servidor web.
- Fer un desplegament automatitzat del llibre mdbook.

Es va instal.lar apache, i es va activar i iniciar com a servei web:

dnf install -y httpd  
systemctl enable --now httpd  

Es va realitzar una còpia de seguretat de la configuració original d'Apache

cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.original  


Vam crear crear un fitxer index.html per a comprovar que el servidor web funciona correctament.

cat << EOF > /var/www/html/index.html
Hola món!
EOF  

<span style="color: yellow;">Configuració del tallafocs </span> 

Es va instal·lar i activar el servei de firewalld, i es van afegir les regles necessàries.

dnf install -y firewalld  
systemctl enable --now firewalld  

Afegirem el servei web al tallafocs:

firewall-cmd --add-service=http --permanent
Recarregarem la configuració del tallafocs:


<span style="color: yellow;">Certificat SSL</span>  

Es important que el nostre servidor web utilitzi un certificat SSL per a que les connexions siguin segures. En el nostre cas que treballem en una intranet, utilitzarem un certificat autofirmat.

Crearem un directori per a guardar els certificats:

mkdir /etc/httpd/ssl

Es va generar i configurar un certificat SSL autofirmat per a l'ús intern. 


Es va configurar Apache per redirigir tot el tràfic de HTTP a HTTPS.

Instal·larem el paquet mod_ssl per a que Apache pugui utilitzar el certificat:

dnf install -y mod_ssl  

Vam editar /etc/httpd/conf.d/ssl.conf per afegir el següent contingut:

SSLCertificateFile /etc/httpd/ssl/apache.crt
SSLCertificateKeyFile /etc/httpd/ssl/apache.key  

Reiniciar el servei d'Apache i obrirem el port 443 (https) al tallafocs:

systemctl restart httpd  

firewall-cmd --add-service=https --permanent
firewall-cmd --reload  

I revisar que el servidor web funciona correctament amb el protocol https.


Ens pot interessar que els nostres usuari sempre utilitzin el protocol https, configurarem apache per redirigir totes les peticions al port 80 al port 443. Per fer-ho editarem el fitxer /etc/httpd/conf.d/vhost.conf i afegirem el següent contingut:

<VirtualHost *:80>
    ServerName mdbook.tc.udl.cat
    Redirect permanent / https://192.168.101.X
</VirtualHost>

I comprovarem que les peticions al port 80 són redirigides al port 443.

<span style="color: yellow;">Instal·lació de mdbook i git</span>

Per instal·lar mdbook, primer necessitarem instal·lar el paquet rust i el paquet git:

dnf install -y rust cargo   
dnf install -y git  

Crearem un usuari especial per a mdbook.  
Aquest usuari tindrà permisos per actualitzar el llibre mdbook que desplegarem amb apache a través de git i github. Però no podrà fer cap altra cosa.

Crearem un usuari amb el nom mdbook sense home directory i una clau ssh per a l'usuari mdbook:

useradd -m mdbook  

su - mdbook  

ssh-keygen -t ed25519 -C "mdbook.tc.udl.cat"


Afegirem la clau ssh al nostre repositori de github.

Com a usuari root:  

mkdir /mdbook-source  
chown mdbook:apache /mdbook-source  

Nota: Necessitem que l'owner sigui l'usuari encarregat de compilar el llibre i el grup sigui apache per a que el servidor web pugui accedir al directori.


<span style="color: yellow;">Instal·larem mdbook</span>

cargo install --locked mdbook --vers 0.4.34  

Afegirem el PATH de mdbook al PATH de l'usuari mdbook:

echo 'export PATH=$PATH:/home/mdbook/.cargo/bin'  >> /home/mdbook/.bashrc  
source /home/mdbook/.bashrc  

Generarem el llibre:

cd /mdbook-source/mdbook<br>
mdbook build  

Per desplegar el llibre, necessitarem moure el contingut del directori book al directori /var/www/html/.

Crearem un enllaç simbòlic al directori /var/www/html/mdbook:

ln -s  /mdbook-source/mdbook/book /var/www/html/
Actualitzarem SELinux per a que permeti a Apache accedir al directori.

Ara actualitzarem el nostre servidor web per servir el nostre llibre aprofitant tota la configuració anterior del servidor web. En aquest cas, haurem de crear un nou fitxer de configuració per aquest llibre. Crearem el fitxer /etc/httpd/conf.d/mdbook.conf amb el següent contingut:

<VirtualHost *:443>
    ServerName mdbook.tc.udl.cat
    DocumentRoot /var/www/html/book
    ErrorLog /var/log/httpd/mdbook-error.log
    CustomLog /var/log/httpd/mdbook-access.log combined
    SSLEngine on
    SSLCertificateFile /etc/httpd/ssl/apache.crt
    SSLCertificateKeyFile /etc/httpd/ssl/apache.key
</VirtualHost>


<span style="color: yellow;">Desplegament automatitzat del llibre</span> 

Utilitzarem un cronjob per a que cada dia es comprovi si hi ha hagut algun canvi en el repositori i en cas afirmatiu, es compili el llibre i es desplegui al servidor web.

mdbook build  

Donarem permisos d'execució al fitxer:

chmod +x /home/mdbook/update.sh  

I crearem un cronjob per a que s'executi cada dia a les 00:00:

crontab -e  

0 0 * * * /home/mdbook/update.sh