# Workshop 02

## ¿Qué necesitamos para desplegar una aplicación web?

- Servidor 
- Dominio 
- IP
- Aplicación (backend, frontend, fullstack)
- Base de datos 
- Presupuesto 
- Seguridad 
    - Firewall
- Seo 
    - Analítica 

## Implementación de un servidor LAMP

1. **Iniciar la maquina:** Es importante que se inicialice donde se ubique el archivo "Vagrantfile".
```bash
cd ~/ISW811/VMs/webserver
vagrant up 
```
2. **Conectarse por SSH:** Nos conectamos a la maquina vagrant tipo GNU/Linux por medio de ssh. 
```bash
cd ~/ISW811/VMs/webserver
vagrant ssh
```
3. **Cambiar hostname de la maquina virtual:** El siguiente comando se debe ejecutar dentro de la maquina virtual para cambiarle el nombre del host que tenia predeterminado. Para poder ver el cambio se sale de la maquina virtual y se vuelve a ingresar.
```bash
sudo hostnamectl set-hostname webserver
exit
vagrant ssh
```
4. **Actualizar el hostname en el archivo "hosts":** Para completar el cambio hay que actualizar el nombre de la maquina en el archivo hosts
```bash
sudo nano /etc/hosts
```
 ![image archivo hosts](./images/archivo%20hosts.png "Archivo hosts")
 5. **Actualizar la lista de paquetes elegibles:** Antes de instalar cualquier paquete se debe actualizar la base de datos de paquetes de la maquina virtual. 
```bash
 sudo apt-get update
```
6. **Instalar paquetes:** Para instalar paquetes vim, curl, apache2, mysql, y php. 
```bash
sudo apt-get install vim vim-nox \
        curl git apache2 mariadb-server mariadb-client \
        php7.4 php7.4-bcmath php7.4-curl php7.4-json \
        php7.4-mbstring php7.4-mysql php7.4-xml
```
7. **Comprobar la IP del servidor:** desde la maquina anfitriona verificamos la ip definida en el parámetro private_network, en este caso es "192.168.33.10", al final le hacemos ping a la dirección ip. 
 ![image private network](./images/private%20network.png "private network")
 ```bash
ping 192.168.33.10
```
 ![image ping ip](./images/ping%20ip.png "ping")
 8. **Editar el archivo hosts:** desde la maquina anfitriona, ejecutamos la ruta windows/system32/drivers/etc como administrador, en el archivo host insertamos la
resolución del dominio deseado y lo abrimos en el navegador para ver que salga la pagina por defecto y todo funcione correctamente.
```bash
cd \
cd Windows\System32\drivers\etc
notepad hosts
192.168.33.10 michelle.isw811.xyz 
```
 ![image dominio](./images/dominio%20hosts.png "dominio")

9. **Habilitar módulos:** Verificamos el estado de apache, y se habilitan modulos para soportar host y certificados SSL para poder publicar el sitio HTTPS, al final reiniciamos apache.  
 ```bash
sudo systemctl status apache2.service
sudo a2enmod vhost_alias rewrite ssl
sudo systemctl restart apache2 
```
10. **Montar carpeta de sitios:** Para mejorar el flujo de trabajo vamos a crear un folder local y lo sincronizamos contra la ruta /home/vagrant/sites de la máquina virtual. Para esto editamos el archivo "VagrantFile"
 ```bash
config.vm.synced_folder "sites/", "/home/vagrant/sites", owner: "www-data", group: "www-data"
```
 ![image sites](./images/vagrant%20sites.png "sites")
 11. **Reiniciar maquina:** Luego de modificar el Vagrantfile debemos reiniciar la máquina para que los cambios surtan efectos
 ```bash
exit
vagrant halt
vagant up
vagrant ssh
```
12. **Crear el «conf» para el sitio:** Necesitaremos crear un archivo .conf para cada sitio que deseemos hospedar en el servidor web.
```bash
cd ~/ISW811/VMs/webserver
mkdir confs
cd confs
touch michelle.isw811.xyz.conf
code michelle.isw811.xyz.conf
```
Una vez creado el archivo .conf le agregamos el siguiente contenido: 
```bash
<VirtualHost *:80>
  ServerAdmin webmaster@michelle.isw811.xyz
  ServerName michelle.isw811.xyz

  # Indexes + Directory Root.
  DirectoryIndex index.php index.html
  DocumentRoot /home/vagrant/sites/michelle.isw811.xyz

  <Directory /home/vagrant/sites/michelle.isw811.xyz>
    DirectoryIndex index.php index.html
    AllowOverride All
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/michelle.isw811.xyz.error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/michelle.isw811.xyz.access.log combined
</VirtualHost>
```
15. **Copiar "conf" a "sites-available".** Desde la máquina virtual se va a copiar el archivo.conf a lo que viene siendo la ruta de sitios disponibles de Apache2. Al final reiniciamos Apache.
```bash
    sudo cp /vagrant/confs/michelle.isw811.xyz.conf /etc/apache2/sites-available
    sudo apache2ctl -t
```
 ![image status](./images/syntax%20ok.png "status")
 17. **Configurar el parámetro "ServerName"** Si al probar la configuración de Apache obtenemos el error Could not reliably determine the server's fully qualified domain name, debemos ejecutar el siguiente comando, para agregar la directiva «SeverName» al archivo de configuración general de Apache, usando el siguiente comando:
```bash
echo "ServerName webserver" | sudo tee -a /etc/apache2/apache2.conf
```
18. **Habilitar el nuevo sitio:** Si ya no se encuentran errores, habilitamos el sitio y reiniciamos apache nuevamente. 
```bash
sudo a2ensite michelle.isw811.xyz.conf
sudo systemctl restart apache2.service
```
19. **Verificar el nuevo sitio:** Ingresamos la URL http://michelle.isw811.xyz.conf
 ![image website](./images/final%20site.png "website")