# Servidor web (mdbook)
En aquest apartat es descriu la configuració del servidor web basat en apache per a servir un llibre escrit en mdbook.

Teniem els següents objectius:
- Instal·lar i configurar un servidor web apache.
- Instal·lar i configurar un certificat SSL.
- Instal·lar i configurar mdbook.
- Fer una configuració segura del servidor web.
- Fer un desplegament automatitzat del llibre mdbook.

Vam instal.lar un servidor web apache com a servei web:

dnf install -y httpd  

Activar i inciar el servei d'Apache:

systemctl enable --now httpd  

Fer un còpia de seguretat de la configuració d'Apache:

cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.original  


Vam crear crear un fitxer index.html per a comprovar que el servidor web funciona correctament.

cat << EOF > /var/www/html/index.html
Hola món!
EOF  

# Configuració del tallafocs  

Instal·lar el paquet firewalld:

dnf install -y firewalld  

Activarem i iniciarem el servei de firewalld:

systemctl enable --now firewalld  

Afegirem el servei web al tallafocs:

firewall-cmd --add-service=http --permanent
Recarregarem la configuració del tallafocs:


Certificat SSL
Es important que el nostre servidor web utilitzi un certificat SSL per a que les connexions siguin segures. En el nostre cas que treballem en una intranet, utilitzarem un certificat autofirmat.

Crearem un directori per a guardar els certificats:

mkdir /etc/httpd/ssl
Generarem el certificat:

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/httpd/ssl/apache.key -out /etc/httpd/ssl/apache.crt  

Generating a RSA private key
...
Country Name (2 letter code) [XX]:ES
State or Province Name (full name) []:Lleida
Locality Name (eg, city) [Default City]:Lleida
Organization Name (eg, company) [Default Company Ltd]:UDL
Organizational Unit Name (eg, section) []:TC
Common Name (eg, your name or your server's hostname) []:mdbook.tc.udl.cat
Email Address []: tc@udl.cat
Observeu que generem un certificat amb una durada de 365 dies. Utilitzem una clau RSA de 2048 bits i el guardem en el directori /etc/httpd/ssl.

Instal·larem el paquet mod_ssl per a que Apache pugui utilitzar el certificat:

dnf install -y mod_ssl  

Editar /etc/httpd/conf.d/ssl.conf i afegir el següent contingut:

Comentar les linies següents i actualitzeu-les amb:  

SSLCertificateFile /etc/httpd/ssl/apache.crt
SSLCertificateKeyFile /etc/httpd/ssl/apache.key  

Reiniciarem el servei d'Apache:

systemctl restart httpd  

Obrirem el port 443 (https) al tallafocs:

firewall-cmd --add-service=https --permanent
firewall-cmd --reload  

Revisar que el vostre servidor web funciona correctament amb el protocol https.


Ens pot interessar que els nostres usuari sempre utilitzin el protocol https, configurarem apache per redirigir totes les peticions al port 80 al port 443. Per fer-ho editarem el fitxer /etc/httpd/conf.d/vhost.conf i afegirem el següent contingut:

<VirtualHost *:80>
    ServerName mdbook.tc.udl.cat
    Redirect permanent / https://192.168.101.X
</VirtualHost>

I comprovarem que les peticions al port 80 són redirigides al port 443.

Instal·lació de mdbook i git  

Per instal·lar mdbook, primer necessitarem instal·lar el paquet rust:

dnf install -y rust cargo   

Instal·larem el paquet git:

dnf install -y git  

Crearem un usuari especial per a mdbook  

Aquest usuari tindrà permisos per actualitzar el llibre mdbook que desplegarem amb apache a través de git i github. Però no podrà fer cap altra cosa.

Crearem un usuari amb el nom mdbook sense home directory:

useradd -m mdbook  

Crearem una clau ssh per a l'usuari mdbook:

su - mdbook  

ssh-keygen -t ed25519 -C "mdbook.tc.udl.cat"


Afegirem la clau ssh al nostre repositori de github.

Com a usuari root:  

mkdir /mdbook-source  
chown mdbook:apache /mdbook-source  

Nota: Necessitem que l'owner sigui l'usuari encarregat de compilar el llibre i el grup sigui apache per a que el servidor web pugui accedir al directori.


Instal·larem mdbook:

cargo install --locked mdbook --vers 0.4.34  

Afegireu el PATH de mdbook al PATH de l'usuari mdbook:

echo 'export PATH=$PATH:/home/mdbook/.cargo/bin'  >> /home/mdbook/.bashrc  
source /home/mdbook/.bashrc  

Generar el llibre:

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


Desplegament automatitzat del llibre  

Com el servidor es troba en una intranet, no podem utilitzar cap servei de CI/CD com github actions o gitlab CI. Tampoc podem utiltizar els webhooks de github per a que el servidor web es desplegui automàticament. La solució que us proposo es utilitzar un cronjob per a que cada dia es comprovi si hi ha hagut algun canvi en el repositori i en cas afirmatiu, es compili el llibre i es desplegui al servidor web.


# Build mdBook
mdbook build  

Donarem permisos d'execució al fitxer:

chmod +x /home/mdbook/update.sh  

Crearem un cronjob per a que s'executi cada dia a les 00:00:

crontab -e  

I afegirem la següent línia:


0 0 * * * /home/mdbook/update.sh