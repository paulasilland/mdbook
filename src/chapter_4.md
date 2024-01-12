# 4. Backup

Hem fet un backup del nostre servidor per a poder tenir una copia de seguretat semanal sempre disponible per a poder recuperar les dades en cas de que es perguin o es corrompi la informació del servidor.  
Per a fer-ho, vam crear una maquina servidor amb una partició de 10gb per a poder utilitzar-la per a emmagatzemar les nostres copies de seguretat. Una vegada fet això, vam configurar un servidor Bacula per a que vagin fent les copies de seguretat.  

Vam tenir que enllaçar el servidor Bacula amb el nostre servidor i en aquest, vam tenir que instal·lar el client de Bacula per a poder enllaçar les dos maquines.  
Despres vam crear un "Job" que es el que s'executara cada setmana a la mateixa hora per a que realitze la copia de seguretat.  

Com podem veure en aquesta imatge, aquestes son les copies de seguretat que s'han fet fins ara i com podem comprobar el sistema de Backup funciona a la perfecció.
