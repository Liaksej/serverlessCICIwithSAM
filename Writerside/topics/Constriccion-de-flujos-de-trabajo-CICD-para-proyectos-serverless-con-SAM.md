# Construcción de flujos de trabajo CI/CD para proyectos serverless con SAM

* [Creación del flujo de trabajo CI/CD](Creacion-del-flujo-de-trabajo-CI-CD-con-SAM.md)
* [Configuración de despliegue Canary y Lineal](Configuracion-de-Canary-y-Lineal-despliegue-con-SAM.md)

## AWS SAM Pipelines

AWS SAM Pipelines es una feature de AWS SAM que automatiza el proceso de creación de un pipeline de entrega continua. 
AWS SAM Pipelines proporciona plantillas para sistemas de CI/CD populares, como AWS CodePipeline, Jenkins, 
GitHub Actions, Bitbucket Pipelines y GitLab CI/CD. Las plantillas de pipeline incluyen las mejores prácticas 
de implementación de AWS para ayudar con implementaciones en múltiples cuentas y regiones. Entornos de AWS como 
dev y producción típicamente existen en diferentes cuentas de AWS. Los equipos de desarrollo pueden configurar 
pipelines de implementación seguros, sin realizar cambios no deseados en la infraestructura. También puedes suministrar 
tus propias plantillas de pipeline personalizadas para ayudar a estandarizar los pipelines a lo largo de 
los equipos de desarrollo. 

AWS SAM Pipelines está compuesto por dos comandos:

1. La orden de configuración **`sam pipeline bootstrap`**, que crea los recursos y permisos de AWS necesarios para 
desplegar artefactos de aplicaciones desde tu repositorio de código en tus entornos de AWS. 
2. **`sam pipeline init`**, un comando de inicialización que genera un archivo de configuración de pipeline 
que el sistema CI/CD puede usar para implementar aplicaciones sin servidor utilizando AWS SAM.

Con dos comandos separados, puedes gestionar las credenciales para operadores y desarrolladores por separado. 
Los operadores pueden usar el comando `sam pipeline bootstrap` para aprovisionar recursos de pipeline de AWS. 
Esto puede reducir el riesgo de errores en producción y los costos operativos. Luego, los desarrolladores pueden 
centrarse en construir sin tener que configurar la infraestructura del pipeline ejecutando el comando `sam pipeline init`.

Se puede combinar también estos dos comandos ejecutando **`sam pipeline init --bootstrap`**. Esto te guiará a través 
de todo el proceso de arranque y de inicialización.

SAM Pipelines crea archivos de configuración adecuados para el proveedor de CI/CD que elijas. Por ejemplo, al usar 
AWS CodePipeline, SAM sintetizará un archivo de plantilla AWS CloudFormation llamado `codepipeline.yaml`. Esta plantilla 
define múltiples recursos de AWS que trabajan juntos para implementar automáticamente una aplicación serverless.

> SAM Pipelines te ahorra el trabajo de configurar tus pipelines desde cero. Sin embargo, las configuraciones que 
> SAM Pipelines crea son simplemente una conveniencia para que puedas empezar. Eres libre de editar estos archivos 
> de configuración de CI/CD después de que SAM los haya creado. 

### Despliegue plantillas SAM con un solo comando

Anteriormente, desplegar aplicaciones a través del SAM CLI requería varios pasos y que proporcionaras un bucket 
de Amazon S3 para el paquete de despliegue de Lambda. Si deseas usar tu propio bucket de S3 para el paquete 
de despliegue de Lambda, aún puedes seguir utilizando los comandos **`sam package`** y **`sam deploy`** para desplegar 
tu plantilla SAM. Sin embargo, ahora puedes usar un único comando, `sam deploy`, para desplegar tus aplicaciones _serverless_, 
y el SAM CLI creará y gestionará este bucket de S3 por ti. El siguiente diagrama es un ejemplo de la salida en 
la terminal cuando se ejecuta el comando **`sam deploy`**. 

![La salida en la terminal](la-salida-en-la-terminal.jpg)

![Final de la salida en la terminal](final-de-la-salida-en-la-terminal.png)

La orden **`sam deploy`** también incluye un modo interactivo guiado (**`sam deploy --guided`**).
Este modo te guía a través de los parámetros necesarios para el despliegue, ofrece opciones predeterminadas y guarda tu
entrada para la aplicación indicada. También puedes ver los cambios realizados en el stack de la aplicación que se desplegará a través
de la salida de la orden **`sam deploy`** y configurar la orden para pedir confirmación de cambios antes de desplegar.
