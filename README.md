# RaspberryPiServer
Configurar el raspberry como ambiente de producción para una aplicación en Play Framework

**Abriendo puertos**

Lo primero que debemos realizar es la apertura de los puertos logicos, necesitaremos tener acceso desde el exterior hacia nuestro servidor RPi.

Podemos hacerlo de dos formas,la primera es haciendo Port forwarding o la segunda colocando la ip de nuestro RPi en el DMZ del router.

Si desconocemos la ip de nuestro RPi podemos usar nmap. Ejecutamos en consola el comando *nmap -sP iplocal/24* y nos listara las conexiones actvas de nuestro router con su respectivo nombre.

Una vez realizado el Port forwarding o activado el DMZ debemos reiniciar el RPi.

**Instalando sbt y puesta en Producción**

*Para instalar sbt en RPi usaremos las siguientes lineas en consola:*

echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
sudo apt-get update
sudo apt-get install sbt

Luego de instalar sbt bajamos nuestro proyecto desde git o el repositorio que estemos usando.

En el archvio *plugins.sbt* del proyecto agregamos las siguientes lineas:

addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "x.y.z")

Y en nuestro *build.sbt* debemos habilitar el archetype.

enablePlugins(JavaAppPackaging)

Teniendo esto, para probar si nuestra app corre en modo de Produccion podemos ejecutar el comando *sbt testProd*.

Ahora para crear nuestro paquete debemos ejecutar el comando *sbt stage* en la base del proyecto, este generara un paquete en una ruta especifica que mostrara por defecto en los logs de sbt, en nuestro caso el log se ve de la siguiente manera:

[warn] Executing in batch mode.
[warn]   For better performance, hit [ENTER] to switch to interactive mode, or
[warn]   consider launching sbt without any commands, or explicitly passing 'shell'
[info] Loading project definition from /home/daniel/.sbt/0.13/staging/2a2bead7ebd26837197a/backend/project
[info] Set current project to consultorioback (in build file:/home/daniel/Proyectos/consultorio/backend/)
[info] Wrote /home/daniel/.sbt/0.13/staging/2a2bead7ebd26837197a/backend/target/scala-2.11/consultorioback_2.11-1.0.pom
[success] Total time: 12 s, completed 2/06/2018 01:15:58 AM

El paquete se creo en la ruta */home/daniel/.sbt/0.13/staging/2a2bead7ebd26837197a/backend/target/.*

Dentro de la carpeta target en la ruta anterior se debio crear la ruta de carpetas */universal/stage/bin/<npmbre-app>*
  
(<nombre-app> sera el que tengamos configurado en *build.sbt* como name := "nombreapp")
  
Para correr nuestra app en mod producción vamos a ejecutar la siguiente linea:

/home/daniel/.sbt/0.13/staging/2a2bead7ebd26837197a/backend/target/universal/stage/bin/<nombre-app> -Dplay.http.secret.key=abcdefghijk.
  
Nos saldra el siguiente log indicandonos que ya tenemos corriendo nuestra app en Modo producción:

[warn] application - Logger configuration in conf files is deprecated and has no effect. Use a logback configuration file instead.
[info] application - Creating Pool for datasource 'default'
[info] p.a.d.DefaultDBApi - Database [default] connected at jdbc:mysql://xxx.xxx.xxx.xxx:3306/consulta
[warn] application - application.langs is deprecated, use play.i18n.langs instead
[warn] application - application.conf @ file:/home/daniel/.sbt/0.13/staging/2a2bead7ebd26837197a/backend/target/universal/stage/conf/application.conf: 22: application.secret is deprecated, use play.crypto.secret instead
[info] play.api.Play - Application started (Prod)
[info] p.c.s.NettyServer - Listening for HTTP on /0:0:0:0:0:0:0:0:9000.

Podemos usar algunos otras opciones para el envio a producción con -Ddb por ejemplo, si no tenemos especificado la base de datos en el application.conf, o podemos usar -Dconfig.resource si tenemos un .config para producción, podemos ver mas configuraciones en la pagina https://www.playframework.com/documentation/2.6.x/Production.


  
  
