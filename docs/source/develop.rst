.. index::
  ambiente de desarrollo

**********************
Ambiente de Desarrollo
**********************

Requerimientos
==============

* Mínimos

  - NodeJs_ v9.5.0

  - Yarn_ v1.7.0

  - Mongo_ v3.6.3

  - direnv_ v2.4.0 o superior

  - redis_ v4

* Recomendaciones

  - Utilizar nvm_ para el versionado de NodeJs

* Opcionales

  - docker_

  - docker-compose_

  - mongodb-compass_

.. _NodeJs: https://nodejs.org/en
.. _Yarn: https://yarnpkg.com/lang/en
.. _Mongo: https://www.mongodb.com) 3.6.3
.. _direnv: https://direnv.net/
.. _redis: https://redis.io
.. _nvm: https://github.com/creationix/nvm
.. _docker: https://www.docker.com
.. _docker-compose: https://docs.docker.com/compose
.. _mongodb-compass: https://www.mongodb.com/products/compass

Instalación del ambiente
========================

*si le falta cumplir alguno de los requerimientos, seguir a la siguiente sección*

Instalación de los componentes necesarios utilizando :code:`yarn`:

.. code-block:: bash

    $ yarn
    $ yarn build:dev


Con esto, se crea la carpeta node-modules, y algunos archivos extra
de configuración.

Generación de las imágenes docker
---------------------------------

*Nota: Se recomienda agregar su usuario al grupo de docker para evitar la necesidad de ejecutar docker con sudo. Esto se puede hacer ejecutando:*

.. code-block:: bash

    $ sudo usermod -a -G docker $USER


Posteriormente, podemos seguir 2 diferentes caminos, usando docker y docker-compose, o usando
simplemente docker, llegando siempre al mismo resultado.

Docker + Docker-compose (recomendable)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Se utilizan los archvios :code:`Dockerfile` y :code:`docker-compose.yml` (para
producción) o :code:`Dockerfile-development` y :code:`docker-compose-development.yml` (para
desarrollo).

**Ambiente Desarrollo**

.. code-block:: bash

    $ docker-compose -f docker/docker-compose-development.yml build
    $ docker-compose -f docker/docker-compose-development.yml up


Con esto se genera la imagen docker, y corre tanto el servidor node, como mongodb.

**Ambiente Producción**

.. code-block:: bash

    $ docker-compose -f docker/docker-compose.yml build
    $ docker-compose -f docker/docker-compose.yml up


Usando yarn start
-----------------

Para esta opción, vamos a necesitar tener todos los servicios (mongo por
ejemplo), corriendo en nuestra maquina al momento de iniciar la aplicación,
ademas de contar con las variables de ambiente seteadas correctamente con la
dirección de host, puerto, y las credenciales necesarias para acceder al
servicio.

La idea es ejecutar la siguiente instrucción en la terminal,

.. code-block:: bash

    $ yarn start


Parametrización del ambiente
============================

Para evitar futuros problemas, se evita en todo momento escribir las
credenciales utilizadas para los diferentes servicios en los archivos de
configuración. En su lugar, utilizan variables de ambiente.

Para facilitar el uso de las mismas, se recomienda utilizar *direnv* exportando
todas las variables necesarias en el archivo *.envrc*.

Eso se hace de la siguiente manera:

.. code-block:: bash

    export TU_CREDENCIAL=CREDENCIAL


*Recordar nunca quitar el archivo .envrc del gitignore*

*Para el correcto funcionamiento del archivo .envrc es necesario contar con* **direnv_**

*Notar que se debe agregar la linea eval en el archivo de configuracion de su shell*

*En bash por ejemplo, se agrega la siguiente linea en el .bashrc*

.. code-block:: bash

    eval "$(direnv hook bash)"


*Para conocer más sobre que línea agregar y como configurarlo en las diferentes
shell, se recomienda leer el repositorio_ en github*

.. _repositorio: https://github.com/direnv/direnv

*Ademas,* **cada vez que se modifique** *se debe ejecutar:_*

.. code-block:: bash

    $ direnv allow


Visualización de los datos
==========================

Para visualizar los datos de una manera más cómoda, se recomienda la
utilización de Mongodb Compass, una app de los creadores de mongo, que permite
visualización de datos, estadísticas, entre otras funcionalidades interesantes.

Dependiendo como se haya levantado el ambiente, las configuraciones que se van a
tener que ingresar en Mongodb Compass.

Si el ambiente se levantó completamente local (es decir, instalando los
programas directamente en el SO), no va a ser necesario cambios, a menos que se
hayan cambiados las configuraciones por defecto de Mongodb.
Por defecto, son las siguientes:

-   Hostname: localhost
-   Port: 27017
-   Authentication: None
-   SSL: off
-   SSH Tunel: off

De utilizar el ambiente dockerizado, va a ser necesario especificar el hostname
y puerto especificado al momento de levantar el ambiente.
En los archivos docker-compose se utiliza la siguiente configuración:

-   Hostname: 172.17.0.1
-   Port: 27017
-   Authentication: none
-   SSL: off
-   SSH Tunel: off

Manejo del repositorio en desarrollo
====================================

Para el manejo del repositorio en etapas de desarrollo, se recomienda la
utilización de :code:`git-flow`, para una mayor organización en el desarrollo.

Además se recomienda también utilizar los :code:`git-flow-hooks`, para una mayor
comodidad. Además de agregar la versión actual del código al archivo "VERSION",
de no utilizar git-flow-hooks, mantener en **todo momento** actualizado dicho
archivo para poder mostrar dicha versión en la app funcionando en producción.

Extra
=====

Para que las consultas a las apis sean más amigables, se propone utilizar el
gist api_request_, el cual es un wrapper de curl, agregando un beautifier
para las respuestas en JSON.

.. _api_request: https://gist.github.com/lucasdc6/741972836ddff247551e5e8b52277541

Levantar ambiente de desarrollo
===============================

Para levantar el ambiente de desarrollo completamente local, hace falta contar
con un programa extra llamado ngrok_, o algún programa
similar para completar el funcionamiento del bot de telegram.

.. _ngrok: https://ngrok.com

En primera instancia es necesario compilar los js, utilizando :code:`npm` o :code:`yarn`.

.. code-block:: bash

    # con yarn
    $ yarn build:dev
    # o con npm
    $ npm run build:dev

Luego es necesario abrir un túnel al localhost para que el webhook del bot de
telegram se pueda comunicar con nuestra app localmente, para esto corremos el
siguiente comando:

.. code-block:: bash

    $ ngrok http localhost:8000

En este caso se utilizó el puerto utilizado por la app, tener en cuenta
cambiarlo en el comando si se utiliza otro.

Luego es necesario definir la variable :code:`URL_BASE` en la **misma** terminal que se
va a correr el servidor localmente, con la url asignada por ngrok, de la siguiente manera:

.. code-block:: bash

    $ export URL_BASE=<url https de ngrok>

Tener en cuenta que se debe utilizar la versión **https** provista por ngrok.

Luego solo hace falta correr el docker-compose de la siguiente manera:

.. code-block:: bash

    $ docker-compose -f docker/docker-compose-development.yml up --build

Con esto se debería contar con todo el ambiente levantado, listo para atender
consultas en el bot de telegram.
