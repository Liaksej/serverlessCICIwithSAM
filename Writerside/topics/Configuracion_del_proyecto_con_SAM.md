# Configuración del proyecto

## Inicializar proyecto

AWS SAM te proporciona una herramienta de línea de comandos, el AWS SAM CLI, que te facilita la creación y gestión 
de aplicaciones serverless. En particular, la creación de un nuevo proyecto se simplifica al crear el esqueleto 
inicial de una aplicación a partir del cual puedes seguir desarrollando el proyecto.

Ejecuta el siguiente comando para generar un nuevo proyecto:

```shell
sam init
```

En el asistente, selecciona Plantillas de inicio rápido de AWS y Hello World Example. No utilices el atajo para usar 
la última versión de Python.

```
Choose an AWS Quick Start application template
        1 - Hello World Example
        2 - Data processing
        3 - Hello World Example with Powertools for AWS Lambda
        4 - Multi-step workflow
        5 - Scheduled task
        6 - Standalone function
        7 - Serverless API
        8 - Infrastructure event management
        9 - Lambda Response Streaming
        10 - Serverless Connector Hello World Example
        11 - Multi-step workflow with Connectors
        12 - GraphQLApi Hello World Example
        13 - Full Stack
        14 - Lambda EFS example
        15 - DynamoDB Example
        16 - Machine Learning
Template: 1

Use the most popular runtime and package type? (Python and zip) [y/N]: n
```

A continuación, selecciona tu runtime preferido y versión. Asegúrate de seleccionar la versión correcta 
como se muestra a continuación.

```
Which runtime would you like to use?
        1 - aot.dotnet7 (provided.al2)
        2 - dotnet8
        3 - dotnet6
        4 - go (provided.al2)
        5 - go (provided.al2023)
        6 - graalvm.java11 (provided.al2)
        7 - graalvm.java17 (provided.al2)
        8 - java21
        9 - java17
        10 - java11
        11 - java8.al2
        12 - nodejs20.x
        13 - nodejs18.x
        14 - nodejs16.x
        15 - python3.9
        16 - python3.8
        17 - python3.12
        18 - python3.11
        19 - python3.10
        20 - ruby3.3
        21 - ruby3.2
        22 - rust (provided.al2)
        23 - rust (provided.al2023)
```

```
nodejs20.x
```

Selecciona Zip como tipo de paquete y pon el nombre para el proyecto.

```
What package type would you like to use?
        1 - Zip
        2 - Image
Package type: 1

Based on your selections, the only dependency manager available is npm.
We will proceed copying the template using npm.

Select your starter template
        1 - Hello World Example
        2 - Hello World Example TypeScript
Template: 2

Would you like to enable X-Ray tracing on the function(s) in your application?  [y/N]: n

Would you like to enable monitoring using CloudWatch Application Insights?
For more info, please view https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-application-insights.html [y/N]: n

Project name [sam-app]:
```
El proyecto debe estar inicializado ahora

Deberías ver una nueva carpeta con el nombre del proyecto creada con un andamiaje básico de Hello World Example TypeScript.

![image](exmpl-prj.png)

> **Nota**
> Si estás interesado en aprender más sobre la inicialización de proyectos SAM, 
> puedes encontrar la referencia completa para el comando `sam init` por el enlace [SAM CLI reference](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-init.html).