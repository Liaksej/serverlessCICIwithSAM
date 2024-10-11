# Requisitos previos

### Cuenta de AWS 

Para llevar a cabo las tareas especificadas en la documentación, se proponen tres enfoques:

#### El primer enfoque - para los solitarios
Si estás trabajando en un proyecto solo o si no hay posibilidad de asignación de roles. Para llevar a cabo todas las acciones descritas en la documentación, necesitarás el rol de administrador. Por ejemplo, un desarrollador puede obtener permisos temporales de administrador en la cuenta de desarrollo, realizar el desarrollo del proyecto en ella, y luego el equipo de DevOps o el administrador responsable desplegarán el proyecto en producción.

#### El segundo enfoque - para los equipos
Si un grupo de desarroladores que inclue por ejemplo también un DevOps con permisos de administrador o amplios derechos de usuario trabaja en el proyecto, el último puede ayudar 
a completar las etapas que definen la implementación y configuración de pipeline, mientras que los desarrolladores 
con acceso a los servicios necesarios para que la aplicación funcione pueden preparar una plantilla de los servicios 
y permisos necesarios, que luego se implementarán a través del pipeline creado.

#### El tercer enfoque - crear un rol temporal
Se puede crear un rol temporal para el usuario responsable de configurar el pipeline y el entorno de desarrollo durante 
la fase activa de desarrollo, incluyendo en este rol todos los servicios necesarios descritos a continuación, y según sea necesario, complementar este rol con nuevas políticas y derechos a medida que avanza el proyecto.

Como umbral mínimo se recomienda tener los siguientes derechos:

* cloudformation:*
* cloudwatch:*
* lambda:*
* s3:*"
* codebuilder:*
* codepipeline:*
* codedeploy:*
* crear Rol IAM
* crear y aplicar IAM Politicas
* otros recursos/servicios utilizados en el proyecto:*

Después de completar la fase activa de desarrollo y poner en marcha el proyecto, es necesario retirar este rol al usuario, 
siguiendo las mejores prácticas de seguridad de AWS. 

En adición a cada uno de los enfoques, se recomienda utilizar AWS IAM Access Analyzer. Esto permitirá realizar una configuración detallada del rol (el servicio es de pago).

### Recursos adicionales:

* [Guía del usuario de IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
* [Taller de aprendizaje de políticas IAM](https://catalog.us-east-1.prod.workshops.aws/workshops/d1531d0a-79fd-45af-b198-d81e349ee660/es-ES)
* [Taller de resolución de problemas de IAM](https://catalog.us-east-1.prod.workshops.aws/workshops/a9661c42-97f6-400a-8dee-a8396e8d418f/en-US)
* [Taller de Refinamiento de Permisos IAM como un Experto](https://catalog.workshops.aws/refining-iam-permissions-like-a-pro/en-US)
* [Taller de evaluación de políticas de IAM](https://catalog.us-east-1.prod.workshops.aws/workshops/6dc3124a-6bd4-46eb-b5c4-be438a82ba3d/en-US)
* [Mejores Prácticas de IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
