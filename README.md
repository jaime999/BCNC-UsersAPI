# BCNC-UsersAPI

Este es un proyecto en el que se va a presentar un desarrollo de API ficticia en Swagger de una gestión de usuarios y viviendas con sus respectivos CRUD usando OpenAPI. El código se ha desarrollado en Swagger Editor, y se ha dividido en distintas campos. En este documento no se va a ir de arriba a abajo en el código, si no que se va a explicar el desarrollo de forma que sea entendible.

## Introducción

Para la primera parte se ha especificado la versión de OpenAPI que se va a utilizar (3.0.3), además del título y una pequeña descripción de las funciones de la API.

En esta parte también se ha definido el servidor que usará la API. Como es un caso de prueba, se ha indicado el servidor ficticio https://bcncGroup.swagger.io/api/v3 .

## Respuestas, estructuras, seguridad...

A continuación se va a explicar todas las estructuras y demás elementos que se han utilizado de base para el desarrollo de los _endpoints_. 

### Seguridad

Para garantizar la seguridad de la API, se van a desarrollar dos tipos de autenticación. Estos tipos de autenticación responden a casos distintos, y se utilizan de manera independiente.

- apiKeyAuth: para utilizar la API se necesita una key. Por defecto es la autenticación que se va a necesitar para utilizar cualquier _endpoint_, y tiene que venir en la cabecera de cada llamada.
- OAuth2: esta es una autorización más específica, ya que se va a necesitar para algunas operaciones. Se han definido dos alcances, uno para la escritura en los _endpoints_ de usuarios, y otro para la escritura en los de asignación de viviendas. Para autenticarse, se tendrá que acceder a la URL ficticia https://bcncGroup.swagger.io/oauth/authorize .

### Respuestas

Primero se han implementado los textos que van a aparecer en varias de las respuestas de los _endpoints_. Estos son de distintos tipos, y aparecen junto a sus códigos de estado correspondientes.

- ResourceNotFound: Esta respuesta aparece cuando el usuario o la vivienda a la que se está intentando acceder no se encuentra en el sistema. Se asocia con el código 404.
- InvalidResource: Este aparece cuando el nombre de usuario o el ID de la vivienda no son correctos, ya sea por la presencia de caracteres especiales, como por cualquier otro caso. Se asocia con el código 400.
- InvalidInput: Aparece si los datos introducidos de algún recurso no son correctos por la razón que sea. También se asocia con el código 400.
- OperationNotAuthorized: Esta respuesta aparece cuando el usuario no tiene permisos suficientes en las operaciones de escritura, cuando la autenticación por OAuth2 del usuario no es satisfactoria. Se asocia con el código 401.
- UserModified: El mensaje que aparece cuando se ha modificado un usuario correctamente. Además, devuelve la información del usuario actualizado.
- HouseModified: El mismo caso, pero para la actualización de las viviendas de un usuario.

### Esquemas

En este apartado se van a ver los distintos parámetros utilizados en las distintas operaciones en los _endpoints_. Para el diseño de estas estructuras, cabe destacar que se ha partido de unas propiedas básicas de los usuarios y de las viviendas.

- UserBase: Los parámetros que puede tener cada usuario. Los campos van a ser **id,username,password,firstName,lastName,email,phone**.
- User: Este esquema utiliza UserBase, pero indica los parámetros obligatorios a la hora de hacer un POST o PUT.
- HouseBase: Los parámetros de cada vivienda. Los campos son **id,country,city,address,username**
- AddressBase: Los parámetros asociados a la propiedas "address" de HouseBase. Sus campos son **street,zip,door**
- House: Utiliza HouseBase con los parámetros obligatorios para el POST y PUT.
- Address: Utiliza AddressBase pero con los parámetros obligatorios para POST y PUT.

### _RequestBodies_

Por último, se han especificado la forma que van a tener los cuerpos en las llamadas POST, PUT y PATCH. Aquí se va a ver la diferencia entre **PUT** (actualización total del recurso) y **PATCH** (actualización parcial). Mientras que con PUT se tiene que enviar el recurso completo como se hace en un POST, PATCH permite enviar únicamente aquellos campos que van a ser modificados. Personalmente, yo únicamente hubiera creado el PATCH, ya que OpenAPI lo acepta, y así solo habría que pasar los campos a actualizar. Sin embargo, entiendo que en algunos contextos usar PUT es útil, ya que te permite eliminar directamente los campos que no estén presentes, en vez de tener que eliminar expresamente el contenido del campo. Por lo tanto, estos son los _bodies_ que se han utilizado en la API. Se pueden enviar en formato JSON, XML o "form-urlencoded":

- UserPatch: este _body_ utiliza el esquema "UserBase" para indicar los campos que se pueden actualizar del usuario en una operación PATCH
- UserBody: este es el _body_ por defecto para POST o PUT, que utiliza el esquema "User", ya que indica los parámetros que son obligatorios enviar.
- HousePatch: este _body_ utiliza el esquema "HouseBase" para indicar los campos que se pueden actualizar de la vivienda de un usuario en una operación PATCH
- HouseBody: este es el _body_ por defecto para POST o PUT, que utiliza el esquema "House", ya que indica los parámetros que son obligatorios enviar.

## Endpoints

Para la creación de los _endpoints_, primero se ha comprobado los tipos distintos que pueden haber en el ejercicio. Se ha decantado por diferenciar entre dos tipos: "Users" para todas las operaciones relacionadas con los usuarios, y "Houses" para las operaciones relacionados con las viviendas de los usuarios.

### Users

Aquí se han implementado los _endpoints_ de creación, actualización, borrado y recuperación de datos de los usuarios. Estas son las rutas que se han utilizado:

- /users (GET): Esta ruta permite obtener todos los usuarios del sistema. Como no hay que enviar ningún parámetro en la consulta, solo se contempla la respuesta con código 200, que devuelve una lista con todos los usuarios, siguiendo el esquema que se ha definido.
- /users (POST): Con este método se va a realizar la creación de un usuario. Para ello se va a utilizar el parámetro "requestBody", pues en OpenAPI es la forma estándar para mandar toda la información de un recurso, ya que permite no mandar todos los parámetros en una _query_, reutilizarlo en otras operaciones (como se ha hecho en este ejercicio), indicar los parámetros obligatorios u opcionales, etc. Para las respuestas se han especificado el código 201 en caso de que la creación sea correcta (y devolver el usuario que se ha creado), el 400 en caso de que haya algún error con los valores introducidos, o el 401 en caso de que no tengas permisos para realizar la acción. Estas dos últimas respuestas usan dos mensajes estandarizados que se han creado para utilizarse en los distintos _endpoints_.

A continuación se van a definir las operaciones que se realizan para un usuario específico. Todos los métodos utilizan el parámetro "username", que se trata de un _pathParam_", ya que se envía en la propia ruta de cada método. Se ha mandado de esta forma para que sea obligatorio para todos los métodos, y para diferenciarse de las rutas anteriores pero siguiendo una estructura común.

- /users/{username} (GET): Se obtiene la información del usuario que se corresponde con ese _username_. Además del código 200, puede devolver un 400 en caso de que el nombre de usuario no sea correcto, o un 404 en caso de que no se haya encontrado el usuario.
- /users/{username} (PATCH): Se actualiza parcialmente la información del usuario que se corresponde con ese _username_. Se utiliza el _body_ "UserPatch" para que no sea obligatorio el envío de ningún parámetro. Las respuestas son la 200 en caso de que todo haya ido bien, la 400 en caso de que haya algún error con los valores introducidos, la 401 si no hay permisos, o la 404 si no se encuentra el usuario.
- /users/{username} (PUT): Se actualiza totalmente la información del usuario que se corresponde con ese _username_. Se utiliza el _body_ "UserBody" para indicar los parámetros obligatorios. Las respuestas son la 200 en caso de que todo haya ido bien, la 400 en caso de que haya algún error con los valores introducidos, la 401 si no hay permisos, o la 404 si no se encuentra el usuario.
- /users/{username} (DELETE): Se elimina el usuario _username_ del sistema. Se devuelve un 200 en caso de eliminación correcta, la 400 si el usuario no es correcto, la 401 si no hay permisos, la 404 si no se encuentra el usuario, **o la 409 en caso de que el usuario aun tenga viviendas asignadas**

### Houses

Este es el _endpoint_ donde se encuentran las operaciones relacionadas con las viviendas de las que dispone un usuario. Para esta ruta, se ha seguido el mismo patrón de /users/{username} del apartado anterior, pero añadiendo la ruta /houses para indicar que se van a realizar operaciones con las casas de estos usuarios.

- /users/{username}/houses (GET): este método obtiene las viviendas asignadas a un usuario. El usuario puede filtrar por ciudad, país y dirección. Para seguir usando la misma estructura en otras operaciones, estos parámetros se pasan por "queryParams", además de que permite que sean opcionales. Para la dirección, se va a utilizar el esquema "AddressBase", ya que tiene las propiedades de una dirección y no son obligatorias. Devuelve un 200 y la información de las casas si no ha habido problemas, un 400 si el usuario no es correcto, un 401 si no hay permisos, o un 404 si no se encuentra el usuario.
- /users/{username}/houses (POST): este método asigna una vivienda al usuario. Utiliza el _body_ "HouseBody" para enviar la información de la vivienda y asignarla a un usuario con un determinado nombre de usuario. Devuelve un 201 en caso de que la creación haya tenido éxito, un 400 en caso de que haya algún error con los valores introducidos, un 401 si no hay permisos, o un 404 si no se encuentra el usuario.

Para los siguientes _endpoints_, además del nombre de usuario, también se debe de indicar cuál es la vivienda que se quiere modificar. Para ello se sigue la misma estructura que hasta ahora, pasando por "pathParams" el ID de la vivienda sobre la que se quiere realizar las operaciones.

- /users/{username}/houses (PATCH): actualiza parcialmente la información de una vivienda de un usuario. Utiliza como _body_ "HousePatch" con los posibles parámetros a actualizar. Devuelve un 200 en caso de que la vivienda se haya modificado con éxito, un 400 en caso de que haya algún error con los valores introducidos, un 401 si no hay permisos, o un 404 si no se encuentra el usuario o la vivienda.
- /users/{username}/houses (PUT): actualiza totalmente la información de una vivienda. Utiliza como _body_ "HouseBody". Devuelve un 200 en caso de que la vivienda se haya modificado con éxito, un 400 en caso de que haya algún error con los valores introducidos, un 401 si no hay permisos, o un 404 si no se encuentra el usuario o la vivienda.
- /users/{username}/houses (DELETE): elimina una vivienda asignada a un usuario. Devuelve un 200 si el borrado ha sido correcto, un 400 si el usuario o la vivienda no son correctos, un 401 si no hay permisos, o un 404 si no se encuentra el usuario o la vivienda.

Este sería el ejemplo de API que se ha desarrollado para la prueba técnica. Se ha implementado un diseño básico, que puede mejorarse en un futuro.
