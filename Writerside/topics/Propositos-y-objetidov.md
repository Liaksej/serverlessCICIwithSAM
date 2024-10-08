# Propositos y objetidov

### Descripción del proyecto 

La configuración del ciclo CI/CD para aplicaciones serverless difiere de la configuración de ciclos similares para aplicaciones de servidor o sitios estáticos. La falta de infraestructura de servidor o contenedor bajo el control del usuario, propietario de las aplicaciones serverless, requiere la integración de herramientas de compilación y despliegue habituales con herramientas y técnicas específicas.

Esta documentación proporciona instrucciones detalladas sobre una de las aproximaciones para organizar el pipeline de CI/CD para aplicaciones serverless en AWS utilizando SAM (Modelo de Aplicación Serverless) - una herramienta de desarrollo de código abierto que simplifica y mejora la experiencia de construir y ejecutar aplicaciones serverless en AWS.

Como runtime para las funciones Lambda se utilizó Node.js 20.x, y como lenguaje TypeScript. Este enfoque fue elegido específicamente para mostrar las capacidades y configuraciones especiales de SAM con compiladores y la creación de capas de dependencias. Sin embargo, la documentación será útil para ser aplicada con cualquier runtime en el que funcione AWS Lambda, ya que los enfoques para trabajar con SAM y el pipeline CI/CD se basan principalmente en el manejo de plantillas de descripción de infraestructura y servicios SAM, que son un caso especial del lenguaje CloudFormation.

La documentación será útil tanto para los profesionales de DevOps como para los desarrolladores y los operadores de sistemas, que trabajan con AWS.

### Objetivos de la documentación

* Obtener una comprensión del ciclo básico de CI/CD para aplicaciones sin servidor.
* Evaluar qué capacidades tiene SAM y cómo estas capacidades ayudan a simplificar y acelerar el desarrollo de aplicaciones sin servidor en AWS.
* Entender cómo iniciar un proyecto de aplicación serverless utilizando SAM.
* Entender las diferencias entre la plantilla SAM y la plantilla CloudFormation, y cuáles son las ventajas principales de usar la plantilla SAM para el desarrollo serverless.
* Entender la configuración de los roles de las funciones lambda en SAM y el entorno de ejecución de Node.js 20.x, así como las particularidades de compilar código TypeScript en lambda. 
* Entender cómo crear un canal CI/CD para una aplicación serverless y cómo SAM puede ayudar en esto.
* Entender cómo configurar paso a paso un pipeline de integración continua/entrega continua de dos etapas (dev-prod) en un proyecto específico, que incluya etapas de prueba, compilación con el uso de las herramientas AWS SAM, AWS CodePipeline, AWS Build, AWS Deploy.
* Analizar cómo SAM ayuda a configurar el monitoreo del pipeline de CI/CD directamente desde la caja.
* Aprender a configurar un despliegue canary o linear de un proyecto utilizando AWS SAM, AWS CodePipeline y AWS Deploy.
* Ofrecer las mejores prácticas para el uso de tuberías CI/CD en aplicaciones serverless.