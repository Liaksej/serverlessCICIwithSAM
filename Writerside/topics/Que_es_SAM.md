# ¿Qué es SAM?

## AWS SAM

AWS SAM es un framework de código abierto que puedes usar para construir tus aplicaciones _serverless_. 
Te proporciona una sintaxis abreviada para expresar las funciones, APIs, bases de datos y mapeos de fuentes de eventos.

Durante las implementaciones, SAM luego transforma y amplía la sintaxis de SAM en una sintaxis de AWS CloudFormation. 
CloudFormation puede entonces aprovisionar tus recursos con capacidades confiables de implementación, haciendo más 
simple la implementación de los aplicación serverless.

### Plantillas SAM

Las plantillas de AWS SAM son una extensión de las plantillas de AWS CloudFormation, con algunos componentes adicionales 
que las hacen más fáciles de usar. Algunos de estos componentes adicionales incluyen los siguientes:

* Cree plantillas compatibles con AWS CloudFormation utilizando la sintaxis abreviada.
* Utiliza infraestructura como código para definir tus funciones Lambda, APIs de API Getaway, 
aplicación serverless desde el Repositorio de Aplicaciones Serverless de AWS y tablas de DynamoDB.
* Si se detectan errores durante la implementación de la plantilla, AWS CloudFormation la revertirá y 
eliminará cualquier recurso que se haya creado, dejando su entorno exactamente como estaba antes de la implementación.

### Ejemplo de una plantilla SAM

AWS SAM requiere el uso de la directiva transform y un bloque de recurso con un tipo correspondiente. 
La directiva transform toma un template completo escrito en la sintaxis de AWS SAM y lo transforma y expande 
en un template compatible de AWS CloudFormation. También puedes opcionalmente incluir cualquier recurso 
en un template de SAM.

![example-template.png](example-template.png)

### Despliegue plantillas SAM con la SAM CLI

Este diagrama resume el proceso más simple de desarrollo con AWS SAM. Comienzas escribiendo el código de
tu función Lambda y definiendo todos tus recursos _serverless_ dentro de una plantilla de AWS SAM. Puedes usar 
la CLI de SAM para emular el entorno Lambda y realizar pruebas locales en tus funciones Lambda. Después de que 
el código y las plantillas sean validados, puedes utilizar el comando de paquete SAM para crear un paquete 
de implementación, que es en esencia un archivo .zip que SAM guarda en Amazon S3. Después de eso, el comando 
de implementación SAM instruye a AWS CloudFormation para implementar el archivo .zip y crear recursos dentro 
de tu consola de AWS.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/FRRRFN5zNMSL3EE2_cxjqTIMYuVN7kQ_n.png)

