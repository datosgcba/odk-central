ODK Central
===========

Central es el servidor [ODK](https://getodk.org/). Administra las cuentas y los permisos de los usuarios, almacena las definiciones de los formularios y permite que los clientes de recopilación de datos, como ODK Collect, se conecten a él para descargar y enviar formularios.

Nuestro objetivo con Central es crear un servidor moderno que sea fácil de instalar, fácil de usar y extensible con nuevas características y funcionalidades tanto directamente en el software como con el uso de nuestras API programáticas REST, OpenRosa y OData.

Este repositorio sirve como un paraguas para el proyecto Central en su conjunto:

* Repositorio de operaciones para empaquetar el servidor y el cliente en una aplicación Docker Compose.
* Repositorio de lanzamiento para la publicación de artefactos binarios. Las notas de la versión se pueden encontrar en este repositorio: consulte las [versiones](https://github.com/getodk/central/releases).

Si está buscando ayuda, consulte el [sitio web de documentación](https://docs.getodk.org/central-intro/). Si eso no resuelve su problema, diríjase al [Foro ODK](https://forum.getodk.org) y haga una búsqueda para ver si alguien más ha tenido el mismo problema. Si ha identificado un nuevo problema o tiene una solicitud de función, publique en el foro. Preferimos las publicaciones en el foro a los problemas de GitHub porque la mayor parte de la comunidad está en el foro.

Contributing
============

Nos encantaría su contribución a Central. Si tienes pensamientos o sugerencias, compártelos con nosotros en el [Panel de funciones](https://forum.getodk.org/c/features) en el Foro ODK. Si desea contribuir con el código, tiene la opción de trabajar en el servidor Backend ([guía de contribución](https://github.com/getodk/central-backend/blob/master/CONTRIBUTING.md)), el sitio web de Frontend ([guía de contribución](https://github.com/getodk/central-frontend/blob/master/CONTRIBUTING.md)), o ambos. Para configurar un entorno de desarrollo, primero siga las [Instrucciones de backend](https://github.com/getodk/central-backend#setting-up-a-development-environment) y luego, opcionalmente, las [Instrucciones de frontend](https://github.com/getodk/central-frontend#setting-up-your-development-environment).

Además del Backend y el Frontend, Central implementa servicios:

* Central se basa en [pyxform-http](https://github.com/getodk/pyxform-http) para convertir formularios desde XLSForm. Por lo general, no debería ser necesario en desarrollo, pero se puede ejecutar localmente.
* Central se basa en [Enketo](https://github.com/enketo/enketo-express) para la funcionalidad de formulario web. Enketo puede ejecutarse localmente y configurarse para trabajar con Frontend y Backend en desarrollo siguiendo estas [instrucciones](https://github.com/getodk/central-frontend/blob/master/docs/enketo.md).

Operaciones
==========

Este repositorio cumple funciones administrativas, pero también contiene el código de Docker para crear y ejecutar una pila central de producción.

## Obtener y configurar ODK Central

Ahora deberá descargar el software. En la consola del servidor, ejecute:

	git clone https://repositorio-asi.buenosaires.gob.ar/ssppbe_usig/odk-central.git


Hay que pensar un rato y descargar muchas cosas. Luego, escriba:

	cd central


Ahora tiene el marco del software del servidor, pero faltan algunos componentes. Ejecuta: 

	git submodule update -i 


A continuación, debe actualizar algunas configuraciones. Primero, copie el archivo de plantilla de configuración para poder editarlo:

	mv .env.template .env 


Luego, edite el archivo escribiendo:

	nano .env 


Esto iniciará una aplicación de edición de texto.

Cambie el valor de `DOMAIN` para que este tenga el nombre de dominio que registró anteriormente. Por ejemplo: 

`DOMAIN=MyOdkCollectionServer.com. No incluya nada con http://.`

Cambie el valor de `SYSADMIN_EMAIL` para que este tenga su propia dirección de correo electrónico. El servicio `letsencrypt` utilizará esta dirección solo para notificarle si hay algún problema con su certificado de seguridad. De ser un dominio local, establezca el parámetro `SSL_TYPE=selfsign`.

El resto de la configuración no se toca. 

Presione `Ctrl + x` para salir del editor de texto. Presione `y` para indicar que desea guardar el archivo y luego presione `Enter` para confirmar el nombre del archivo. No cambie el nombre del archivo.

Ahora, agrupamos todo en un servidor. Ejecute:

	docker-compose build 


Esto tomará un tiempo y generará una gran cantidad de logs. No se preocupe si parece detenerse sin decir nada durante un rato. Cuando termine, debería mostrar el texto `Successfully built` y recuperar la línea de comandos. Cuando eso suceda, ejecuta:

	docker-compose up --no-start


Una vez que esté completo, ¡felicitaciones! Has instalado el ODK Central. A continuación, debemos enseñarle al servidor cómo iniciarlo.


## Corriendo ODK Central

Ahora, ejecute:

	docker-compose up -d 

La primera vez que lo inicie, tardará un poco en configurarse. Una vez que le dé unos minutos y vuelva a tener la línea de comandos, querrá ver si todo está funcionando correctamente.
Para ver si la herramienta ha terminado de cargarse, ejecute:

	docker-compose ps

En la columna State, del contenedor `central_nginx`, podrás ver el texto `Up o Up (healthy)`. Si ve `Up (health: starting)`, espere unos minutos. Si ve algún otro texto, algo salió mal. Es normal ver `Exit 0` en el contenedor `central_secrets`.
Si el dominio está activo, puedes visitarlo en un navegador web para verificar que obtiene el sitio web.

¡Ya casi terminas! Todo lo que tiene que hacer es crear una cuenta de administrador para poder iniciar sesión en ODK Central.

Nota: en caso de que el build de error de certificado, actualice la configuración del certbot.


### Actualizar Certbot

Entrar al contenedor de nginx:

	docker exec -it central_nginx_1 /bin/bash


Instalar editor de texto de preferencia (vim/nano):

	apt-get update
	apt-get install vim nano


Editar script que configura el certbot:

	nano /scripts/util.sh


En la línea 62, cambiar el valor de `PRODUCTION_URL`:

	PRODUCTION_URL:https://acme-v02.api.letsencrypt.org/directory


Guardar y salir del editor como se explico anteriormente, y salir del contenedor con `Ctrl + d`.
Reiniciar contenedor de nginx desde la ruta en donde esta el `docker-compose.yml`:

	docker-compose restart central_nginx_1


## Iniciar sesión en ODK Central

Asegúrese de estar en la carpeta `central/` del servidor. Si no ha cerrado sesión de consola desde antes, debería estar bien.
Luego, escriba:

	docker-compose exec service odk-cmd --email mail@ejemplo.com user-create

Sustituyendo la dirección de correo electrónico según corresponda. Al presionar `Enter` se le pedirá una contraseña para esta nueva cuenta.

El paso anterior creó una cuenta, pero no la convirtió en administrador. Para hacerlo, ejecute:

	docker-compose exec service odk-cmd --email mail@ejemplo.com user-promote


Por ahora ha terminado, pero si alguna vez pierde la pista de su contraseña, siempre puede restablecerla ejecutando:

	docker-compose exec service odk-cmd --email mail@ejemplo.com user-set-password

Al igual que con la creación de una cuenta, se le pedirá una nueva contraseña después de presionar `Enter`.


Una vez que tenga una cuenta, no tiene que volver a pasar por este proceso para futuras cuentas. Puede iniciar sesión en el sitio web con su nueva cuenta y crear directamente nuevos usuarios desde el gestor de usuarios del ODK Central.

Licencia
=======

Todo ODK Central tiene licencia de [Apache 2.0](https://raw.githubusercontent.com/getodk/central/master/LICENSE) License.
