Cambia la Ley ([english](/README.md))
============

'Cambia la Ley' es una propuesta para:
- Traducir la ley para ciudadanos no familiarizados con la jerga
  legislativa.
- Resumir a las secciones más comentadas de una ley.
- Ver el texto original de la ley, para que puedas sacar tus propias
  conclusiones.

##Dependencias
- Node.js >= v0.10.24
- NPM >= 1.3.10
- Sails.js = 0.10.5

Instala el resto de los paquetes listados en ``package.json`` ejecutando `sudo npm install` en el directorio del proyecto.

##Instalación
1. Instalar node.js ([Instrucciones](http://howtonode.org/how-to-install-nodejs))
2. Instalar sails `sudo npm -g install sails`
3. Instalar grunt `npm install grunt` y `npm install grunt-core`

##Configuración
El proyecto requiere de variables de entorno para utilizar la base de datos y la autenticación mediante Twitter.

###Base de datos
Por defecto, el proyecto utiliza PostgreSQL. La conexión a la base de datos se configura en ``config/connections.js``.
En este archivo se pueden tener varias configuraciones. Por ejemplo:
```
postgresql: {
  adapter   : 'sails-postgresql',
  host      : process.env.PG_HOST,
  port      : process.env.PG_PORT,
  user      : process.env.PG_USER,
  password  : process.env.PG_PASSWORD,
  database  : process.env.PG_DATABASE
}
```
El nombre de esta configuración es ``postgresql`` y utiliza el módulo  ``sails-postgresql`` (instalado anteriormente).
El resto de las variables deben de estar definidas en el entorno de su servidor. Por ejemplo, si el servidor es Heroku,
deberá de definir las variables ``PG_HOST``, ``PG_PORT``, etc. con los datos pertinentes de su base de datos, en la página de "Settings" en el apartado de "Config Vars" de su aplicación.

Para hacer uso de una de las múltiples configuraciones en ``config/connections.js``, el nombre de ésta se debe de especificar en ``config/models.js``, en el atributo ``connection``:
```
module.exports.models = { 
  connection: 'postgresql',
  migrate: 'safe'
};
```
IMPORTANTE: el atributo ``migrate`` puede tomar 3 valores:
- ``safe``: No habrá modificaciones a la estructura de la base de datos, aunque los modelos hayan sido modificados.
- ``alter``: La información de la base de datos se mantendrá. Su estructura se modificará de acuerdo a los modelos (función experimental).
- ``drop``: Cada que se reinicie el proyecto, la información en la base de datos se perderá. Su estructura se modificará de acuerdo a los modelos.
Se recomienda leer la [documentación oficial] (http://sailsjs.org/#/documentation/concepts/ORM/model-settings.html).

###Twitter OAuth
Para poder autenticar a los usuarios mediante Twitter, se debe de contar con una "app" registrada (https://apps.twitter.com).
El archivo ``config/globals.js`` declara una instancia del módulo ``node-twitter-api`` (instalado anteriormente),
el cual se encarga de negociar la autenticación con los servidores de Twitter:
```
var twitterApi = require('node-twitter-api');
var twitterApiInstance = new twitterApi({
  consumerKey: process.env.TWITTER_KEY,
  consumerSecret: process.env.TWITTER_SECRET,
  callback: process.env.TWITTER_CALLBACK_HOSTNAME + process.env.TWITTER_CALLBACK_FUNCTION
});
```
Esta instancia utiliza 4 variables de entorno:
- ``TWITTER_KEY``: Consumer key / API key (https://apps.twitter.com).
- ``TWITTER_SECRET``: Consumer secret / API secret (https://apps.twitter.com).
- ``TWITTER_CALLBACK_HOSTNAME``: La URL de su servidor a donde Twitter llevará a su usuario después de haberse autenticado. Por ejemplo ``http://explica.la``.
- ``TWITTER_CALLBACK_FUNCTION``: La función en su proyecto que creará la sesión de usuario si éste se autenticó correctamente. Por defecto ésta función debe de ser ``/user/twitterAuthCallback``.

###El archivo config/local.js
Sails.js le da mayor prioridad a ``config/local.js`` y sobreescribe lo que esté en otros archivos de configuración con lo que aquí se especifique. Este archivo es útil si desea probar el proyecto en su máquina local. Por ejemplo, si quiere cambiar las variables de la conexión ``postgresql`` creada anteriormente, tendría que hacer algo como lo siguiente en ``config/local.js``:
```
module.exports.connections = {
  postgresql: {
    adapter : 'sails-postgresql',
    host : 'db.example.org',
    port : '5432',
    user : 'mi_usuario',
    password : 'mi_pass',
    database : 'nombre_de_la_bdd'
  }
}
```
Para utilizar la autenticación de Twitter mientras el proyecto corre localmente, deberá de hacer algo como:
```
var twitterApi = require('node-twitter-api');
var mi_conf =  new twitterApi({
    consumerKey: 'mi_otra_llave',
    consumerSecret: 'mi_otro_secreto',
    callback: 'http://127.0.0.1:1337/user/twitterAuthCallback'
})
module.exports.globals = {
  twitterApi: mi_conf,
}
```

##Uso
Después de haber configurado al proyecto, navegue al directorio raíz de éste y ejecute ``sails lift``.

###Usuarios
Hay 3 roles de usuario:
- ``user``: puede crear, editar, destruir y votar por anotaciones.
- ``expert``: puede hacer todo lo que un usuario, pero sus anotaciones son de otro color.
- ``admin``: puede hacer todo lo que un experto, y crear artículos, leyes y "tags".


IMPORTANTE: por cuestiones de desarrollo, se "hardcodearon" varios integrantes del equipo de desarrolladores como ``admin`` en las políticas de privilegios del proyecto (``api/policies.js``). Asegúrese de borrar las líneas 3 a 11 de ``expertRole.js`` y ``adminRole.js`` si hizo fork antes de este [commit] (https://github.com/CodeandoMexico/cambia-la-ley/commit/e3641df6283b8291eb3b1ffb5eb4443e299e60b8).

El rol por defecto para todos los usuarios es ``user``. El otorgamiento de roles se hace directamente en la base de datos (ej. ``UPDATE "user" set role = 'admin' where id = 1;``).

IMPORTANTE: Si usa PostgreSQL, debido a que ``user`` es palabra reservada, y la tabla de usuarios se llama de esa manera, necesitará referirse a la tabla con comillas dobles en sus queries (ej: ``SELECT * FROM "user";``).

###Tags, Leyes y Artículos
- Un tag (o reforma) es una colección de leyes
- Una ley puede pertenecer únicamente a un tag
- Una ley es una colección de artículos
- Un artículo puede pertenecer únicamente a una ley

Crear:
- ``/tag/create``: Crea tags/reformas
- ``/law/create``: Crea leyes
- ``/article/create``: Crea artículos

Editar:
- Basta con navegar al tag/ley/artículo a editar y agregar ``/edit`` a la URL
- Requiere que el usuario tenga el rol de ``admin``

Borrar:
- Requiere interacción directa con la base de datos


##Demo
[Explica.la/ley](http://explica.la/ley)

##¿Preguntas o problemas?
Mantenemos la conversación del proyecto en nuestra página de problemas [issues] (https://github.com/CodeandoMexico/cambia-la-ley/issues). Si usted tiene cualquier otra pregunta, nos puede contactar por correo <equipo@codeandomexico.org>.

##Contribuye
Queremos que este proyecto sea el resultado de un esfuerzo de la comunidad. Usted puede colaborar con [código](https://github.com/CodeandoMexico/cambia-la-ley/pulls), [ideas](https://github.com/CodeandoMexico/cambia-la-ley/issues) and [bugs](https://github.com/CodeandoMexico/cambia-la-ley/issues). Lea nuestro archivo [CONTRIBUYE](/CONTRIBUYE.md).

##Licencia
Available under the license: GNU General Public License (GPL) v3.0. Read the document [LICENSE](/LICENSE) for more information

Creado por [Codeando México](http://www.codeandomexico.org), 2014.

![alt text](http://blog.codeandomexico.org/images/logo.png "Codeando México")
