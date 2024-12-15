

<h1>Despliegue y configuración de Servidor Web en VM Ubuntu IaaS: Virtual Hosts, HTTPS, y redirección</h1>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734104985246/0c6a5072-f3af-44a9-97e9-ecef2afbcf0a.png)
<h4>Aprende a desplegar un servidor web en VM Ubuntu con virtual hosts, HTTPS y redirección para alojar múltiples sitios de manera segura y eficiente</h4>
<h4>Puedes leerlo aquí: https://blog.alexisabel.com/virtual-host</h4>
Cuando se alojan múltiples sitios web o servicios en un solo servidor, los virtual hosts son una herramienta fundamental. Un virtual host es una configuración que permite a un servidor web, como Apache, servir contenido diferente según el nombre de dominio o la dirección IP de la solicitud entrante. Este enfoque es especialmente útil para gestionar varios sitios web en la misma máquina o para proporcionar contenido personalizado según el método de acceso (por ejemplo, por IP o dominio).

En este artículo, demostraremos cómo configurar virtual hosts en una máquina virtual IaaS con **Ubuntu 22.04**. También habilitaremos HTTPS para conexiones seguras y usaremos archivos `.htaccess` para implementar reglas de redirección. Ya seas desarrollador, administrador de sistemas o estudiante, esta guía te llevará paso a paso a implementar un servidor web de nivel profesional.

**Apartados que se van a realizar:**

**Apartado 1: Configurar Virtual Host**

* Al acceder la URL `vhost.<dominio>.com` se mostrará:
    

```xml
Has accedido a la página del examen
Soy [nombre].
```

* Al acceder por la ip de la MV mostrará
    

```xml
No se puede acceder por IP
```

**Apartado 2: Permitir acceso HTTPS al nombre de dominio**

* Permitir acceso HTTPS para el dominio del apartado anterior
    

**Apartado 3: Configuración de redirección con .htaccess**

* Al intentar acceder al recurso /redirección te redirige a la web del instituto. Se debe resolver con .htaccess
    

---

## Configuración previa

Accedemos al siguiente enlace: [Create a virtual machine - Microsoft Azure](https://portal.azure.com/#create/microsoft.freeaccountvirtualmachine-linux) donde podremos crear una VM Linux gratuita.

Le daremos un nombre a la máquina, seleccionamos la región y el sistema a utilizar, en nuestro caso usaremos la versión más estable de Ubuntu, 22.04 LTS x64.

Respecto al tamaño seleccionado, aunque su costo mensual es de aproximadamente **US$8.76**, se encuentra dentro de los **servicios gratuitos elegibles** de Azure, por lo que no se me cobrará durante el periodo de uso

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734090392985/0e537d7d-00b5-458b-8e14-23c571ab00af.png )

La configuración de acceso a la VM por defecto será la que usemos, al ser la más segura. El acceso se realizará por SSH, de modo que proporcionaremos un nombre de usuario y Azure nos genera una pareja de claves. Azure se queda con la clave pública y en el último paso nosotros nos descargamos la privada.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734090562566/6764fcd0-b5d1-44bc-a7d8-268a55c6d048.png )

Por último, configuramos los puertos que queremos habilitar, en nuestro caso nos bastará con el 80 para HTTP, el 443 para conectarnos de forma segura con HTTPS y el 22 para la conexión SSH a la máquina.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734090676619/a39d55b9-4490-44b1-ad0b-c3f14ee50c8f.png )

Una vez configurado todo, como se ha dicho antes, para conectarnos necesitamos una clave privada que Azure nos genera. Nos la descargamos y la guardamos en un lugar seguro.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734090889630/9dbe62f0-b6b0-4f05-bafd-ffec6e104f9f.png )

Revisamos toda la información y creamos la máquina, lo que puede tardar algunos minutos.

Una vez creada la máquina, podremos conectarnos a ella de varias maneras, la más común utilizando clientes como [Putty](https://www.putty.org/).

Necesitaremos conocer nuestra IP; para ello, iremos a la página de inicio de Azure, donde podremos ver nuestra máquina en la sección “Recientes”.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734091447130/206ebdc5-cafb-4b11-863b-5affb4d491f5.png )

Una vez accedemos al recurso, podemos ver la Dashboard con todas las configuraciones. En el centro de la pantalla podemos ver la dirección IP.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734091895002/891f9361-b4a0-41a6-9d94-1e0483e8dc09.png )

<details data-node-type="hn-details-summary"><summary>IP estática</summary><div data-type="detailsContent">La IP que nos genera Azure por defecto es dinámica, de modo que si reiniciamos la máquina ésta cambiará. Para evitar esto, hacemos click en la IP y nos llevará a la configuración de IP, donde podremos cambiar a IP estática para tener una reservada para nosotros. Le damos a guardar (es posible que se reinicie la máquina)</div></details>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734092053487/901d86f8-f25a-4a17-b4dc-7e809581583a.png )

Una vez tenemos nuestra IP, ya podemos abrir Putty para conectarnos a la máquina. Pero antes, tendremos que poner nuestra clave privada para autenticarnos.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734092218014/dc669560-00da-4134-9d0e-f28857a8a761.png )

*⚠️Putty no soporta de forma nativa el formato .PEM, entonces lo primero que haremos sera convertir el archivo a formato .PPK (Putty Private Key). Para hacerlo utilizaremos la herramienta PUTTYgen que se debió instalar al instalar el paquete de Putty.*

En la ventana de PUTTYgen pulsamos el boton «Load» y seleccionamos nuestro archivo .PEM

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Si no ves el archivo .pem, asegúrate de que estás configurando el explorador para que muestre todos los archivos. Esto lo puedes hacer normalmente en el menú de la esquina inferior derecha</div>
</div>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734092765542/df65ed35-b34b-4f36-beb7-ea6c4604a620.png )

Una vez importada, hacemos click en “Save private key” y le damos un nombre con la extensión .ppk

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734092864649/7e9e177c-5d67-49af-9ced-53e7fbf0ee45.png )

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Aunque conectarse de este modo parezca engorroso, es infinitamente más seguro que utilizar una contraseña creada por nosotros.</div>
</div>

Ahora que ya tenemos nuestra clave, volvemos a Putty.

En el menú de la izquierda, accedemos a Connection → SSH → Auth → Credentials y ponemos la clave privada que nos hemos descargado anteriormente.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734092294040/5b1d1d79-5c1f-4eaa-b721-7bf6e7e94b0b.png )

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Es recomendable volver al menú principal de Putty y guardar la sesión para no tener que configurar todo cuando queramos conectarnos en otro momento</div>
</div>

Una vez tengamos la conexión preparada, le damos a “Open”. Es probable que aparezca una alerta de seguridad, si confías en la máquina, puedes aceptarla y continuar la conexión.

Cuando se abra la terminal, nos pedirá introducir el nombre de usuario que hemos configurado en Azure.

Si hemos hecho todo correctamente podremos ver la linea de comando de la sesion SSH. Ya estamos dentro de nuestra máquina.

Antes de comenzar con el primer ejercicio, es importante configurar el **firewall interno** de nuestra máquina para asegurar que solo los puertos necesarios estén abiertos.

El ***firewall*** interno controla el tráfico que puede entrar o salir de la máquina, protegiéndola de accesos no autorizados. Vamos a configurar **UFW** (Uncomplicated Firewall) para permitir solo los puertos **22** (SSH), **80** (HTTP) y **443** (HTTPS), y bloquear todos los demás.

Para ello, vamos a introducir los siguientes comandos:

```bash
sudo ufw allow http
sudo ufw allow ssh
sudo ufw allow https
```

Comprobaremos el estado del firewall utilizando “sudo bash” (para obtener priviliegios de administrador) y con el comando “ufw status”. Aparecerá desactivado, de modo que debemos activarlo con:

```bash
ufw enable
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734093386612/21191fd5-b872-4c63-a4cc-681a9fe2cb25.png )

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Asegúrate de permitir el puerto <strong>22 (SSH)</strong> antes de habilitar <strong>UFW</strong> para evitar bloquearte fuera de la máquina. Si no lo configuras primero, perderás el acceso remoto y no podrás conectarte nuevamente.</div>
</div>

Ya tenemos el firewall configurado, ahora podemos empezar con el primer ejercicio.

---

## **Apartado 1: Configurar Virtual Host**

En este primer apartado, configuraremos dos Virtual Hosts en Apache. Al acceder a `vhost.<dominio>.com`, se mostrará el mensaje "Has accedido a la página del examen, Soy \[nombre\]". Al intentar acceder directamente mediante la IP de la máquina virtual, se mostrará el mensaje "No se puede acceder por IP".

Vamos a instalar **Apache2**, un servidor web ampliamente utilizado, para poder servir las páginas web en nuestra máquina virtual. Apache2 nos permitirá configurar los Virtual Hosts, gestionar el tráfico HTTP y servir el contenido web a través de los puertos adecuados (80 y 443).

La instalación se realizará mediante el comando `apt` en Ubuntu, y luego habilitaremos el servicio para que se inicie automáticamente.

```bash
sudo apt install apache2
```

Aceptamos y esperamos a que se instale. Para habilitar **Apache2** y asegurarnos de que se inicie automáticamente al arrancar la máquina, utilizamos el siguiente comando:

```bash
sudo systemctl enable apache2
```

Además, para iniciar el servicio inmediatamente después de la instalación, utilizamos:

```bash
sudo systemctl start apache2
```

### Primera parte: Configuración de Virtual Host

### **Paso 1:** Crear la carpeta `htmlIP`

La carpeta `htmlIP` es donde vamos a almacenar los archivos que se mostrarán cuando alguien acceda a nuestra máquina virtual **por su IP**. Normalmente, Apache almacena los archivos web en `/var/www/html`, pero en este caso queremos tener contenido diferente para las visitas que accedan usando la IP, por lo que creamos una nueva carpeta específica para ese propósito.

```bash
sudo mkdir /var/www/htmlIP
```

### Paso 2: Crear los archivos `index.html`

1. **Para el dominio (**[**vhost.&lt;dominio&gt;.com**](http://vhost.examen538406.tridentemarketingsolutions.com)**):**
    

* Apache, por defecto, sirve archivos desde la carpeta `/var/www/html`. Vamos a configurar esta carpeta para que muestre el mensaje adecuado cuando accedamos mediante el dominio.
    
* Ejecuta el siguiente comando para eliminar el archivo `index.html` (tiene ya contenido que no queremos):
    
* ```bash
        sudo rm /var/www/html/index.html
    ```
    
* Luego, creamos un nuevo archivo `index.html` vacío en la misma ubicación:
    
    ```bash
    sudo nano /var/www/html/index.html
    ```
    
* Dentro de este archivo, escribimos el siguiente mensaje:
    
* ```xml
        <h1>Has accedido a la página del examen, Soy [nombre] </h1>
    ```
    

Para guardar el archivo y salir de **nano**, presiona:

* **Ctrl + O** (para guardar).
    
* **Enter** (para confirmar).
    
* **Ctrl + X** (para salir).
    

Este archivo se mostrará cuando accedamos al dominio [`vhost.<dominio>.com`](http://vhost.examen538406.tridentemarketingsolutions.com).

2. **Para la IP (en la carpeta** `htmlIP`):
    
    * Ahora, queremos que las visitas que accedan a nuestra máquina usando su **IP** vean un mensaje diferente. Para esto, creamos un archivo `index.html` dentro de la carpeta `htmlIP` que hemos creado previamente.
        
    * ```bash
            sudo nano /var/www/htmlIP/index.html
        ```
        
    * Y agregamos el siguiente mensaje:
        
        ```xml
        <h1>No se puede acceder por IP</h1>
        ```
        
        Guardamos igual que antes. Este archivo será mostrado cuando alguien intente acceder a la máquina mediante su dirección IP.
        
    
    ### **Paso 3: Configurar los archivos** `.conf` **de Apache**
    
3. **Copiar el archivo de configuración predeterminado**:
    
    * Apache utiliza archivos de configuración `.conf` para definir cómo debe comportarse el servidor web. Para configurar los dos Virtual Hosts (uno para el dominio y otro para la IP), primero vamos a copiar el archivo de configuración predeterminado `000-default.conf` a un nuevo archivo llamado `001-ip.conf`:
        
    * ```bash
            sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/001-ip.conf
        ```
        
4. **Editar el archivo para el dominio**:
    

* Ahora vamos a configurar el primer Virtual Host para que cuando accedamos a través del dominio [`vhost.<dominio>.com`](http://vhost.examen538406.tridentemarketingsolutions.com), se sirvan los archivos desde `/var/www/html`.
    
* Editamos el archivo `000-default.conf` para configurar el Virtual Host:
    
* ```bash
        sudo nano /etc/apache2/sites-available/000-default.conf
    ```
    
    Dentro del archivo, buscamos la siguiente sección:
    
    ```yaml
    <VirtualHost *:80>
        DocumentRoot /var/www/html
    </VirtualHost>
    ```
    
    Edita o añade el `ServerName` para que esté correctamente configurado para el dominio [`vhost.<dominio>.com`](http://vhost.examen538406.tridentemarketingsolutions.com):
    
    ```yaml
    <VirtualHost *:80> 
        ServerName vhost.<dominio>.com 
        DocumentRoot /var/www/html 
    </VirtualHost>
    ```
    

2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734094806451/a0441212-71d4-407d-8379-a2649b7163c9.png )
    
3. **Editar el archivo para la IP**:
    
    * Ahora, editamos el archivo `001-ip.conf` para configurar el Virtual Host que se activará cuando se acceda a la máquina mediante su IP. Este Virtual Host debe servir los archivos desde la carpeta `htmlIP` que hemos creado.
        
        ```bash
        sudo nano /etc/apache2/sites-available/001-ip.conf
        ```
        
    * <div data-node-type="callout">
        <div data-node-type="callout-emoji">💡</div>
        <div data-node-type="callout-text">En este archivo, el <code>DocumentRoot</code> debe apuntar a <code>/var/www/htmlIP</code> para que, al acceder por la IP (ServerName), se sirva el contenido de esa carpeta.</div>
        </div>
        
    * Asegúrate de que el archivo incluya esto: (deben coincidir mayúsculas)
        
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734096166397/3d869a83-f8eb-441f-aa16-2d71d7c246a6.png )
        
    
    Al usar la IP como `ServerName`, Apache responde sirviendo el contenido ubicado en la carpeta especificada en el `DocumentRoot`. Esto ocurre siempre que la solicitud llega a través de esa IP.
    

### Paso 4: Activar los sitios y recargar Apache

1. **Activar los Virtual Hosts**:
    

* Ahora que hemos configurado los dos archivos `.conf`, debemos habilitarlos para que Apache los use. Para esto, utilizamos el comando `a2ensite`:
    
    ```bash
    sudo a2ensite 000-default.conf
    sudo a2ensite 001-ip.conf
    ```
    

2. **Recargar Apache**:
    
    * Después de habilitar los sitios, necesitamos recargar Apache para que los cambios tomen efecto. Usamos el siguiente comando:
        
        ```bash
        sudo systemctl reload apache2
        ```
        

Con esto, hemos configurado correctamente los Virtual Hosts para que:

* Al acceder a [`vhost.<dominio>.com`](http://vhost.examen538406.tridentemarketingsolutions.com), se muestre : Has accedido a la página del examen. Soy \[nombre\].
    
* Al acceder directamente por la IP de la máquina, se muestre el mensaje: **"No se puede acceder por IP"**.
    

### Segunda parte: Configurar el dominio

En esta parte, vamos a configurar el dominio para que apunte a la dirección IP pública de nuestra máquina virtual. En mi caso utilizaré **Porkbun** pues es mi proveedor de dominio, aunque el proceso es similar en otros proveedores.

Iremos a la gestión de dominios y a los ajustes DNS del dominio que tenemos contratado.

Allí podremos crear un registro de tipo A (registro que asocia un nombre de dominio con una dirección IP).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734095799914/63602e6c-0934-419a-abca-329838bab5cd.png )

Una vez creado es probable que debamos esperar hasta varios minutos para que se propague el registro.

Ahora, podremos entrar en vhost.&lt;dominio&gt;.com y veremos el mensaje.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734103666023/844c4a37-bc61-4df4-8139-ca09162999df.png )

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Las tildes no se muestran correctamente porque el archivo no está guardado en codificación <strong>UTF-8</strong>.</div>
</div>

Del mismo modo, si probamos a acceder mediante la IP:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734096580220/8854362d-96f9-4fb9-9238-d951a95632b4.png )

---

## **Apartado 2:** Permitir acceso HTTPS al nombre de dominio

En esta segunda parte del ejercicio, lo que vamos a hacer es configurar HTTPS (acceso seguro a través de SSL) para el dominio que ya configuramos en el apartado anterior. HTTPS es un protocolo que asegura la comunicación entre el navegador del usuario y tu servidor, cifrando la información transmitida. Para ello, necesitamos generar un certificado SSL y configurar Apache para que use este certificado y pueda servir páginas a través de HTTPS.

1. #### **Habilitar el módulo SSL en Apache**
    
    Apache necesita tener habilitado el módulo SSL para manejar conexiones HTTPS. Este módulo es el que permite cifrar la comunicación entre el servidor y el cliente.
    
    Para habilitar el módulo SSL (y reiniciar Apache), utilizamos los siguientes comandos:
    
    ```bash
    sudo a2enmod ssl
    sudo systemctl restart apache2
    ```
    
2. **Generar el certificado SSL**
    
    Ahora que Apache puede manejar conexiones HTTPS, necesitamos un certificado SSL para poder cifrar la comunicación. Para generar un certificado SSL autofirmado (es decir, no emitido por una entidad certificadora externa), usamos el siguiente comando:
    
    ```bash
    sudo make-ssl-cert /usr/share/ssl-cert/ssleay.cnf /etc/ssl/private/examen.crt
    ```
    
    <div data-node-type="callout">
    <div data-node-type="callout-emoji">💡</div>
    <div data-node-type="callout-text">Si prefieres, puedes usar <code>openssl</code> en lugar de <code>make-ssl-cert</code> para generar el certificado SSL, lo cual es más común y compatible. En este caso, al estar en un entorno de pruebas, con este nos valdrá.</div>
    </div>
    
    El comando `make-ssl-cert` genera un certificado SSL autofirmado. El archivo `examen.crt` es el archivo del certificado que usaremos para habilitar HTTPS en nuestro servidor. En este caso, he utilizado el nombre de dominio [`vhost.<dominio>.com`](http://examen538406.tridentemarketingsolutions.com), pero puedes ajustar el nombre del archivo según sea necesario. (Si nos pide nombre alternativo, no hace falta poner nada)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734097123107/bf2eadd0-d5e6-4d70-81f8-c530be10cebd.png )
    
    <div data-node-type="callout">
    <div data-node-type="callout-emoji">💡</div>
    <div data-node-type="callout-text">Un certificado autofirmado no es reconocido por navegadores como "confiable" por defecto, ya que no es emitido por una autoridad certificadora (CA) oficial. Sin embargo, para este ejercicio, nos sirve para poder configurar y probar HTTPS en el servidor.</div>
    </div>
    
    <div data-node-type="callout">
    <div data-node-type="callout-emoji">💡</div>
    <div data-node-type="callout-text">Si necesitas que tu página sea accesible de forma segura y sin advertencias en los navegadores, puedes usar Let's Encrypt para obtener un certificado SSL gratuito. Puede serte útil <a target="_self" rel="noopener noreferrer nofollow" href="https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04-es" style="pointer-events: none">este tutorial de Digital Ocean</a>.</div>
    </div>
    
    Para asegurarnos de que el certificado se ha generado correctamente, vamos a la carpeta donde se almacenan los certificados en el sistema y verificamos que el archivo `examen.crt` exista.
    
    ```bash
    cd /etc/ssl/private
    ls -l
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734097275759/bd3410ae-7419-4f28-bcb7-da7605dc0ac1.png )
    
    `b008283e.0 -> examen.crt`: Este es un enlace simbólico que apunta al archivo `examen.crt`.
    
3. #### **Configurar el archivo de configuración de Apache para SSL**
    
    Ahora vamos a configurar Apache para que sirva contenido a través de HTTPS, mediante un archivo de configuración específico que tiene Apache para habilitar SSL. Vamos a copiar la configuración predeterminada para SSL y modificarla según nuestras necesidades.
    
    ```bash
    cd /etc/apache2/sites-available
    sudo cp -a default-ssl.conf https-ssl.conf
    ```
    
4. #### **Editar el archivo de configuración SSL**
    
    Ahora necesitamos editar el archivo de configuración para que Apache sepa qué certificado utilizar y para qué dominio configurarlo.
    
    ```bash
    sudo nano /etc/apache2/sites-available/https-ssl.conf
    ```
    
    Dentro del archivo, añadimos el ServerName con nuestro dominio:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734102064234/16986c24-574a-4723-b499-0524de9d4d9a.png )
    
    Además, añadimos la siguiente linea para especificar la ubicación del archivo del certificado SSL que generamos anteriormente.
    
    ```yaml
    SSLCertificateFile /etc/ssl/private/examen.crt
    ```
    
    La añadimos junto con el resto de configuraciones de SSL (IMPORTANTE AÑADIRLA DEBAJO)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734102052592/c21bab79-7251-4d44-a22d-2360eac44e9b.png )
    
    Después de hacer estos cambios, guardamos el archivo y salimos.
    
5. **Activar el sitio SSL** para que Apache lo sirva:
    
    ```bash
    sudo a2ensite https-ssl.conf
    sudo service apache2 restart
    ```
    
    Con esto ya tendríamos terminado el segundo apartado y podemos comprobarlo accediendo a nuestra web utilizando HTTPS.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734103705784/1adee3d3-6d1b-4438-a4af-ce83d3018047.png )

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">El certificado autofirmado no es emitido por una autoridad certificadora (CA) confiable, por lo que los navegadores no lo reconocen como seguro. Por eso aparece un mensaje de “No seguro”. Los certificados autofirmados solo garantizan la autenticidad del servidor, pero no su confianza pública.</div>
</div>

---

## **Apartado 3: Configuración de Redirección con .htaccess**

En este último apartado, configuramos una redirección con `.htaccess` para redirigir una ruta a cualquier página que queramos.

Hay varias formas de hacer redirecciones en un servidor web, como usar configuraciones en los archivos (`.conf`), o modificar las reglas del servidor mediante archivos `.htaccess` específicos en directorios. En este ejercicio, vamos a usar un archivo `.htaccess` para realizar una redirección sencilla.

1. **Crear la carpeta “redireccion”**:
    
    Primero, vamos a crear una carpeta llamada “redireccion” (o cualquiero otro nombre que queramos para la ruta) dentro del directorio `/var/www/html`.
    
    ```bash
    sudo mkdir /var/www/html/redireccion
    ```
    
2. **Accedemos a la ruta y creamos el archivo .htaccess**
    
    ```bash
    cd /var/www/html/redireccion
    sudo nano .htaccess
    ```
    
    Y dentro incluimos la siguiente línea para redirigir cualquier intento de acceder a `/redireccion` hacia la página que queramos, en mi caso usaré como ejemplo la página del IES Azarquiel.
    
    ```yaml
    Redirect 301 /redireccion http://www.ies-azarquiel.es
    ```
    
    <div data-node-type="callout">
    <div data-node-type="callout-emoji">💡</div>
    <div data-node-type="callout-text">El tipo de redirección <strong>301</strong> indica una redirección <strong>permanente</strong>. Otros códigos, como <strong>302</strong> y <strong>307</strong>, indican redirecciones <strong>temporales</strong>.</div>
    </div>
    
    Guardamos el archivo con Ctrl + O y Enter, y salimos con Ctrl + X.
    
3. **Habilitar .htaccess en Apache**
    
    Necesitamos asegurarnos de que Apache permita usar archivos `.htaccess` para configurar redirecciones. Para ello, debemos editar el archivo apache2.conf
    
    ```bash
    sudo nano /etc/apache2/apache2.conf
    ```
    
    Buscamos esta parte (bajando mucho) y editamos el archivo apache2.conf cambiando el “None” por “all” en AllowOverride
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734103324282/f64ec404-d3ca-4927-acda-a5ea9f7fe6b2.png)
    
    Esto permitirá que Apache lea y ejecute las reglas de `.htaccess` en los directorios dentro de`/var/www/`, como el que hemos creado.
    
    Para finalizar, reiniciamos Apache para que se apliquen los cambios:
    
    ```bash
    sudo systemctl restart apache2
    ```
    

Ahora podemos probar nuestra redirección entrando en [http://vhost.&lt;dominio&gt;.com/redireccion](http://vhost.alexisabel.com/redireccion), en mi caso he añadido un botón en el HTML de la página que nos lleva a la ruta:

```xml

<h1>Has accedido a la página del examen, Soy Alex</h1>
 <a href="/redireccion">
    <button>Redireccion</button>
  </a>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734104366409/462ce62b-920c-4376-935f-7ffc91c25666.gif)

# Conclusión

En este trabajo, hemos aprendido a desplegar y configurar un servidor web en Ubuntu, configurando Virtual Hosts para gestionar múltiples sitios, habilitando HTTPS para mayor seguridad, y utilizando redirecciones con .htaccess. Estos pasos nos permiten optimizar el acceso a nuestros servicios, y también nos ayudan a proporcionar un servicio más seguro.

Espero que esta guía te haya sido útil ;)
