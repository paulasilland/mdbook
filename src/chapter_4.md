# 4. Ticketing

Per fer el ticketing, hem decidit fer servir osTicket.

PAS 1
- Descarregar mariaDB:  
<span style="color: yellow;">dnf install httpd mariadb mariadb-server -y</span>

- Instal·lar els repositoris EPEL i PHP Remi:  
<span style="color: yellow;">dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y  
dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm -y

- Deshabilitem el repositori PHP predeterminat i habiliteu el repositori PHP Remi:  
<span style="color: yellow;">
dnf module reset php  
dnf module install php:remi-7.4 -y </span>

- Instal·larem PHP amb una extensió diferent:  
<span style="color: yellow;">dnf install php php-pear php-cgi php-common php-curl php-gettext php-zip php-opcache php-apcu php-imap php-intl php-gd php-mysqli wget unzip -y</span>

- Després que tots els paquets estiguin instal·lats s'inicia i habilita els serveis Apache i MariaDB:  
<span style="color: yellow;">systemctl enable --now httpd
systemctl enable --now mariadb</span>

PAS 2

- Començarem per establir la contrasenya de MariaDB:
<span style="color: yellow;">mysql_secure_installation</span>

- Iniciem sessió al servidor:  
<span style="color: yellow;">mysql -u root -p</br>

- Creem una base de dades, un usuari per a osTicket i esborrem els privilegis:  
CREATE DATABASE osticketdb;<br>
GRANT ALL PRIVILEGES ON osticketdb.* TO osticketuser@localhost IDENTIFIED BY "securepassword";
FLUSH PRIVILEGES;<br>
EXIT;


PAS 3

- Descarreguem osTicket:  
<span style="color: yellow;">wget https://github.com/osTicket/osTicket/releases/download/v1.15.4/osTicket-v1.15.4.zip</span>

- Descomprimim l'arxiu:  
<span style="color: yellow;">unzip osTicket-v1.15.4.zip -d /var/www/html/osTicket</span>

- Copiem el fitxer de configuració de mostra de osTicket:  
<span style="color: yellow;">cp /var/www/html/osTicket/upload/include/ost-sampleconfig.php /var/www/html/osTicket/upload/include/ost-config.php</span>

- Canviem la propietat del directori osTicket a Apache:  
<span style="color: yellow;">chown -R apatxe:apache /var/www/html/osTicket</span>


PAS 4

- Descarreguem nano:  
<span style="color: yellow;">dnf install nano -y</span>

- Creem un fitxer de configuració de host virtual Apache per a osTicket:  
<span style="color: yellow;">nano /etc/httpd/conf.d/osticket.conf</span>

- Agreguem el següent:  
<VirtualHost *:80><br>
      ServerAdmin admin@example.com<br>
      DocumentRoot /var/www/html/osTicket/upload<br>
      ServerName osticket.example.com<br>
      <Directory /var/www/html/osTicket/><br>
           Options FollowSymlinks<br>
           AllowOverride All<br>
           Requereix all granted
      </Directory>

      ErrorLog /var/log/httpd/osticket_error.log
      CustomLog /var/log/httpd/osticket_access.log combined
</VirtualHost>

- Reiniciem el servei:  
<span style="color: yellow;">systemctl restart httpd</span>

- Veiem l'estat de l'Apache:  
<span style="color: yellow;">systemctl status httpd</span>




PAS 5

- Obrim al nostre navegador osTicket usant la IP de la nostra màquina
- Ens assegurem que totes les extensions php estan instal·ladeds.

- Haurem de canviar el nom del següent fitxer i eliminar el directori d'instal·lació:  
<span style="color: yellow;">chmod 0644 /var/www/html/osTicket/upload/include/ost-config.php
rm -rf /var/www/html/osTicket/upload/setup/<br>

- Haurem d'emplenar les dades.
- Tornem al navegador, donem el nostre nom d'usuari i contrasenya d'administrador.