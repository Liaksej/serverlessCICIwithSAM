# Creación del flujo de trabajo CI/CD con SAM

En este capítulo, vamos a utilizar una característica de SAM llamada [SAM Pipelines](https://aws.amazon.com/blogs/compute/introducing-aws-sam-pipelines-automatically-generate-deployment-pipelines-for-serverless-applications/). 
Cuando estés listo para desplegar tu aplicación serverless de forma automatizada, puedes generar un pipeline de despliegue 
para el sistema CI/CD de su elección. AWS SAM proporciona un conjunto de plantillas de canalizaciones de inicio con las que puede generar 
canalizaciones en cuestión de minutos mediante el comando sam pipeline init.

Actualmente, AWS SAM CLI admite generar configuraciones iniciales de canalización CI/CD para los siguientes proveedores:

* [AWS CodePipeline](https://aws.amazon.com/codepipeline/) 
* [Jenkins](https://www.jenkins.io/) 
* [GitLab CI/CD](https://docs.gitlab.com/ee/ci/) 
* [GitHub Actions](https://github.com/features/actions) 
* [Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/) 

![image_4.1.png](image_4.1.png)

### 4.1. Selección de herramientas para CI/CD (AWS CodePipeline, Github Actions y otros) 

Prerrequisitos

* [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html), last version
* [Git client](https://git-scm.com/downloads)
* IAM Role like describen in [Reqisitos previos](Requisitos-previos.md).
* [Bitbucket](https://bitbucket.org) account

### Create a Git Repository

Any CI/CD pipeline starts with a code repository. In this module, we use Bitbucket.

Por favor, inicia sesión con tu cuenta existente de Bitbucket y crea un repositorio llamado `sam-app`.

La visibilidad del **repositorio** (público o privado) no importa para este taller.

### Subir el código

Agrega la URL de tu repositorio de Bitbucket como un remoto en tu proyecto git local.

## Cómo construir un pipeline

La mejor manera de automatizar la creación de tuberías de CI/CD es aprovisionándolas programáticamente utilizando 
Infraestructura como Código (IaC). Esto es útil en un entorno de microservicios donde puedes tener una tubería por 
servicio. En tales entornos, podría haber docenas o incluso cientos de tuberías de CI/CD. Contar con una forma 
automatizada de crear esas tuberías de CI/CD permite a los desarrolladores avanzar rápidamente sin la carga 
de construirlas manualmente. SAM Pipelines, que estarás utilizando, es una herramienta para aliviar esa carga.

### Diferentes formas de crear infraestructura en la nube

Los equipos técnicos utilizan diferentes herramientas y marcos de trabajo de IaC para crear recursos en 
la nube de forma programática. A continuación se enumeran algunas opciones.

* [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
* [AWS CloudFormation](https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials.html) 
* [AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/codepipeline_example.html) 
* [Terraform](https://www.terraform.io/docs/providers/aws/r/codepipeline.html) 

En este taller, estamos utilizando AWS SAM, exclusivamente. Vale la pena aclarar las diferencias entre AWS SAM y 
CloudFormation, si no estás familiarizado. Puede ser útil pensar en SAM como un lenguaje de programación de un 
nivel más alto como C o C++, y CloudFormation como lenguaje ensamblador. Realizamos nuestro trabajo en AWS SAM, 
que genera e implementa plantillas de CloudFormation en nuestro nombre. En secciones anteriores, usamos comandos 
locales de sam para probar nuestra aplicación sin servidor de forma local. AWS SAM es un conjunto de herramientas 
destinado a aumentar la productividad al desarrollar aplicaciones sin servidor y ofrece funciones como sam local que 
no están presentes en otras herramientas de IaC.

## Presentación de AWS SAM Pipelines

[SAM Pipelines](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-pipeline-bootstrap.html) funcionan creando un conjunto de archivos de configuración e infraestructura que usted utiliza 
para crear y gestionar su canalización CI/CD.

A partir de esta redacción, las SAM Pipelines pueden iniciar las pipelines CI/CD para los siguientes proveedores:

* [AWS CodePipeline](https://aws.amazon.com/codepipeline/) 
* [Jenkins](https://www.jenkins.io/) 
* [GitLab CI/CD](https://docs.gitlab.com/ee/ci/) 
* [GitHub Actions](https://github.com/features/actions) 
* [Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)

> Las SAM Pipelines es una feature que inicia las pipelines CI/CD para los proveedores listados.
> Esto se ahorra el trabajo de configurarlas desde cero. Sin embargo, se puede usar SAM como una herramienta de implementación con cualquier
> proveedor de CI/CD. Utilizas varios comandos de sam para construir e implementar aplicaciones SAM, independientemente del conjunto de herramientas de CI/CD.
> Además, las configuraciones que crea SAM Pipelines son una conveniencia para empezar. Eres libre de editar estos
> archivos de configuración CI/CD después de que SAM los haya creado.

SAM Pipelines crea archivos de configuración apropiados para su proveedor de CI/CD elegido. Por ejemplo, al usar 
AWS CodePipeline, SAM sintetizará un archivo `codepipeline.yaml` y la carpeta `pipeline` con archivos 
`buildspec*.yml`. Esos archivos definen su canalización de CI/CD utilizando AWS CodePipeline, AWS CodeBuild, 
AWS CodeDeploy.

### Arquitectura de AWS CodePipeline

Al final de esta sección, tendremos un pipeline de CI/CD que se actualizará automáticamente utilizando 
AWS CodePipeline y que llevará a cabo los siguientes pasos.

1. Desencadenar después de un envío a la rama principal (en: `push` en la captura de pantalla de abajo)
2. Ejecutar pruebas unitarias a través de AWS CodeBuild
3. Construir y empaquetar el código de la aplicación a través de AWS CodeBuild
4. Despliegue a un entorno de desarrollo/pruebas a través de AWS CodeDeploy.
5. Prueba de integración
6. Desplegar a un entorno de producción a través de AWS CodeDeploy.

![s3orphans-Pipeline.jpg](s3orphans-Pipeline.jpg)

## Generar un pipeline inicial para AWS CodePipeline en AWS SAM

Para generar una configuración inicial de tuberías para AWS CodePipeline, realiza las siguientes tareas en este orden:

1. Crear recursos de infraestructura.
2. Crear la configuración del flujo de trabajo.
3. Guardar la configuración de pipeline a Git.
4. Conectar el repositorio Git con el sistema CI/CD.

> **Nota**
> El siguiente procedimiento utiliza dos comandos de AWS SAM CLI, `sam pipeline bootstrap` y `sam pipeline init`. 
> La razón por la que hay dos comandos es para manejar el caso de uso en el que los administradores (es decir, usuarios 
> que necesitan permisos para configurar recursos de infraestructura AWS como usuarios IAM y roles) tienen más permisos 
> que los desarrolladores (es decir, usuarios que solo necesitan permisos para configurar pipelines individuales, pero 
> no los recursos de infraestructura AWS requeridos).

### Paso 1: Crear recursos de infraestructura

Las **tuberías** que utilizan AWS SAM requieren ciertos **recursos de AWS**, como un usuario de IAM y roles con 
los permisos necesarios, un depósito de Amazon S3 y opcionalmente un repositorio de Amazon ECR. Debes tener 
un conjunto de **recursos de infraestructura** para cada etapa de implementación de la **tubería**.

Puedes ejecutar el siguiente comando para ayudar con esta configuración:

#### Crear la etapa dev del pipeline

```shell
cd ~/environment/sam-app
sam pipeline bootstrap --stage dev
```

A continuación se enumeran una lista de las preguntas y respuestas requeridas para este taller. Presta especial atención al seleccionar 
OpenID Connect (OIDC) para el proveedor de permisos de usuario y GitHub Actions cuando se te pida seleccionar un proveedor OIDC. 
**Ten en cuenta que los números pueden ser diferentes al elegir de una lista enumerada**. La salida completa y las respuestas se proporcionan a continuación 
como referencia adicional.

1. Select a pipeline template to get started: `AWS Quick Start Pipeline Templates` (1)
2. Select CI/CD system: `AWS CodePipeline` (3)
3. Do you want to go through stage setup process now? [Y/n]:
4. [1] Stage definition. Stage name: `dev`
5. [2] Account details. Select a credential source to associate with this stage: `default (named profile)` (2)
6. Enter the region in which you want these resources to be created: `Your region of choice`
7. **Select a user permissions provider**: OpenID Connect (OIDC) (2)
8. **Select an OIDC provider**: Bitbucket (1)
9. Enter the URL of the OIDC provider: `return/enter`
10. Enter the OIDC client ID (sometimes called audience) [sts.amazonaws.com]: `return/enter`
11. Enter the Bitbucket organization that the code repository belongs to. If there is no organization enter your username instead: `Your Bitbucket username`
12. Enter Bitbucket repository name: flumotion/sam-app
13. Enter the name of the branch that deployments will occur from [main]: `return/enter`
14. Enter the pipeline execution role ARN if you have previously ... []: `return/enter`
15. Enter the CloudFormation execution role ARN if you have previously ... []: `return/enter`
16. Please enter the artifact bucket ARN for your Lambda function. If you do not ... []: `return/enter`
17. Does your application contain any IMAGE type Lambda functions? [y/N]: `N`
18. Press enter to confirm the values above ... : `return/enter`
19. Should we proceed with the creation? [y/N]: `y`

Una vez que esto se complete, verás un resultado que se parecerá a lo siguiente:

```
Successfully updated!
The following resources were created in your account:
	- Pipeline execution role
	- CloudFormation execution role
	- Artifact bucket
	- IAM OIDC Identity Provider
...
```

Estos recursos fueron creados con una pila de CloudFormation que Pipelines SAM sintetizó y lanzó.
Opcionalmente, puedes navegar hasta la consola de CloudFormation e inspeccionar esta pila para ver todo lo que se creó.

En este paso, los flujos de trabajo de SAM crearon un proveedor de identidad IAM OIDC resaltado a continuación. 
Los proveedores de identidad IAM OIDC son entidades en IAM que describen un servicio de proveedor de identidad 
externo (IdP) que admite el estándar OpenID Connect (OIDC), como Google o Salesforce. Para nuestros propósitos, 
el proveedor de identidad externo es GitHub. El proveedor de identidad IAM OIDC se utiliza para establecer confianza 
entre su cuenta de AWS y GitHub. Esto permitirá que las acciones de GitHub asuman un rol IAM de AWS para la 
actividad de implementación.

![image_4.2.3.png](image_4.2.3.png)

#### Crear la etapa prod del pipeline

Ahora que has creado los recursos necesarios para la etapa de construcción de desarrollo, necesitas seguir los 
mismos pasos para la etapa de producción.

Ejecuta los siguientes comandos:

```shell
cd ~/environment/sam-app
sam pipeline bootstrap --stage prod
```

A continuación se enumeran la lista de **preguntas y respuestas** requeridas para este taller. 
**Tenga en cuenta que los números pueden ser diferentes al elegir de una lista enumerada**. 
La salida completa y las respuestas se proporcionan a continuación como referencia adicional.

1. [2] Account details. Select a credential source to associate with this stage: `default (named profile)` (2)
2. Enter the region in which you want these resources to be created: `Your region of choice`
3. Enter the pipeline execution role ARN if you have previously ... []: `return/enter`
4. Enter the CloudFormation execution role ARN if you have previously ... []: `return/enter`
5. Please enter the artifact bucket ARN for your Lambda function. If you do not ... []: `return/enter`
6. Does your application contain any IMAGE type Lambda functions? [y/N]: `N`
7. Press enter to confirm the values above ... : `return/enter`
8. Should we proceed with the creation? [y/N]: `y`

![image_4.2.4.png](image_4.2.4.png)

### Paso 2: Generar la configuración del pipeline

Para generar la configuración del pipeline, ejecuta el siguiente comando:

```shell
sam pipeline init
```

A continuación se enumera una lista de las preguntas y respuestas requeridas para este taller. Tenga en cuenta que los números pueden ser diferentes al elegir de una lista enumerada. La salida completa y las respuestas se proporcionan a continuación como una referencia adicional.

1. Select a pipeline template to get started: AWS Quick Start Pipeline Templates (1)
2. Select CI/CD system: AWS CodePipeline (3)
3. What is the git branch used for production deployments? [main]: `return/enter`
4. What is the template file path? [template.yaml]: `return/enter`
5. Select an index or enter the stage 1's configuration name (as provided during the bootstrapping): `1`
6. What is the sam application stack name for stage 1? [sam-app]: `sam-app-dev`
7. Select an index or enter the stage 2's configuration name (as provided during the bootstrapping): `2`
8. What is the sam application stack name for stage 2? [sam-app]: `sam-app-prod`

```
sam pipeline init generates a pipeline configuration file that your CI/CD system
can use to deploy serverless applications using AWS SAM.
We will guide you through the process to bootstrap resources for each stage,
then walk through the details necessary for creating the pipeline config file.

Please ensure you are in the root folder of your SAM application before you begin.

Select a pipeline template to get started:
	1 - AWS Quick Start Pipeline Templates
	2 - Custom Pipeline Template Location
Choice: 1

Cloning from https://github.com/aws/aws-sam-cli-pipeline-init-templates.git (process may take a moment)
Select CI/CD system
	1 - Jenkins
	2 - GitLab CI/CD
	3 - GitHub Actions
	4 - Bitbucket Pipelines
	5 - AWS CodePipeline
Choice: 3
You are using the 2-stage pipeline template.
 _________    _________
|         |  |         |
| Stage 1 |->| Stage 2 |
|_________|  |_________|

Checking for existing stages...

2 stage(s) were detected, matching the template requirements. If these are incorrect, delete .aws-sam/pipeline/pipelineconfig.toml and rerun

This template configures a pipeline that deploys a serverless application to a testing and a production stage.

What is the git branch used for production deployments? [main]:
What is the template file path? [template.yaml]:
We use the stage configuration name to automatically retrieve the bootstrapped resources created when you ran `sam pipeline bootstrap`.

Here are the stage configuration names detected in .aws-sam/pipeline/pipelineconfig.toml:
	1 - dev
	2 - prod
Select an index or enter the stage 1's configuration name (as provided during the bootstrapping): 1
What is the sam application stack name for stage 1? [sam-app]: sam-dev
Stage 1 configured successfully, configuring stage 2.

Here are the stage configuration names detected in .aws-sam/pipeline/pipelineconfig.toml:
	1 - dev
	2 - prod
Select an index or enter the stage 2's configuration name (as provided during the bootstrapping): 2
What is the sam application stack name for stage 2? [sam-app]: sam-prod
Stage 2 configured successfully.

SUMMARY
We will generate a pipeline config file based on the following information:
	Select a user permissions provider.: OpenID Connect (OIDC)
	What is the git branch used for production deployments?: main
	What is the template file path?: template.yaml
	Select an index or enter the stage 1's configuration name (as provided during the bootstrapping): 1
	What is the sam application stack name for stage 1?: sam-dev
	What is the pipeline execution role ARN for stage 1?: arn:aws:iam::750353892954:role/aws-sam-cli-managed-dev-pipe-PipelineExecutionRole-1MY2CND2E58CK
	What is the CloudFormation execution role ARN for stage 1?: arn:aws:iam::750353892954:role/aws-sam-cli-managed-dev-p-CloudFormationExecutionR-11CGGAM8W2IXZ
	What is the S3 bucket name for artifacts for stage 1?: aws-sam-cli-managed-dev-pipeline-artifactsbucket-1ppvh61mp2vj1
	What is the ECR repository URI for stage 1?:
	What is the AWS region for stage 1?: us-west-2
	Select an index or enter the stage 2's configuration name (as provided during the bootstrapping): 2
	What is the sam application stack name for stage 2?: sam-prod
	What is the pipeline execution role ARN for stage 2?: arn:aws:iam::750353892954:role/aws-sam-cli-managed-prod-pip-PipelineExecutionRole-152A2QJI3ORPD
	What is the CloudFormation execution role ARN for stage 2?: arn:aws:iam::750353892954:role/aws-sam-cli-managed-prod-CloudFormationExecutionR-O4DKAM9I6JS0
	What is the S3 bucket name for artifacts for stage 2?: aws-sam-cli-managed-prod-pipeline-artifactsbucket-nrg2vlxhrjg2
	What is the ECR repository URI for stage 2?:
	What is the AWS region for stage 2?: us-west-2
Successfully created the pipeline configuration file(s):
	- codepipeline.yaml
What we've just done is create the AWS CodePipeline workflow to create a full CI/CD Pipeline using AWS CodePipeline and other AWS services.

Your project should have the structure below (only the most relevant files and folders are shown).

└── pipeline
    ├── buildspec_build_package.yml
    ├── buildspec_deploy.yml
    ├── buildspec_feature.yml
    ├── buildspec_integration_test.yml
    ├── buildspec_unit_test.yml
└──── codepipeline.yaml # (new) AWS Pipeline Workflow

```

Puedes abrir opcionalmente `codepipeline.yaml` y otros archivos para ver lo que las Canalizaciones SAM crearon 
para nosotros. Al mirar `codepipeline.yaml`, puedes ver que hay casi 1000 líneas de flujo de trabajo de 
Clowdformation que SAM creó. ¡Piensa en cuánto tiempo acabas de ahorrar usando las Canalizaciones 
SAM en lugar de hacer esto a mano!

### Paso 3: Conecta el repositorio Bitbucket con tu sistema de CI/CD

Para **AWS CodePipeline** ahora puedes crear la conexión ejecutando el siguiente comando:

```shell
sam deploy -t codepipeline.yaml --stack-name <pipeline-stack-name> --capabilities=CAPABILITY_IAM --region <region-X>
```

> **Nota**
> `<pipeline-stack-name>` - es el nombre de tu canalización, no el nombre de tu etapa de implementación. Por lo tanto, 
> como de costumbre necesitas > para crear **solo un pipeline**. Este pipeline contendrá dos etapas de 
> despliegue: prod y dev.

Después de ejecutar el comando `sam deploy` previamente, completa la conexión siguiendo los pasos indicados en Para 
completar una conexión en el tema [Actualizar una conexión pendiente](https://docs.aws.amazon.com/dtconsole/latest/userguide/connections-update.html) 
de la Guía del usuario de la consola de herramientas de desarrollo. Además, guarde una copia del `CodeStarConnectionArn` 
de la salida del comando porque la necesitarás si quieres usar AWS CodePipeline con otra rama que no sea `main`.

### Paso 4: Haz commit y push de la configuración del Pipeline en el repositorio de Git

Este paso es necesario para asegurar que su sistema CI/CD esté al tanto de la configuración de su canalización, y se 
ejecutará cuando se realicen cambios.

### Configura otras ramas

Por defecto, AWS CodePipeline utiliza la rama principal con AWS SAM. Si deseas utilizar una rama distinta a principal, 
debes ejecutar nuevamente el comando `sam deploy`. Ten en cuenta que, dependiendo del repositorio Git que estés 
utilizando, es posible que también necesites proporcionar el `CodeStarConnectionArn`:

```shell
# For GitHub and Bitbucket
sam deploy -t codepipeline.yaml --stack-name <feature-pipeline-stack-name> --capabilities=CAPABILITY_IAM --parameter-overrides="FeatureGitBranch=<branch-name> CodeStarConnectionArn=<codestar-connection-arn>"
```

Ahora que la *configuración* ha sido subida a tu repositorio de GitHub, **GitHub Actions** se hará cargo, creará 
y ejecutará tu primer flujo de trabajo basado en el pipeline.yaml que has **confirmado**.

## Automatice la implementación de su aplicación AWS SAM.

Para configurar su canalización [AWS CodePipeline](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html) 
para automatizar la creación e implementación de su aplicación AWS SAM, la plantilla de AWS CloudFormation y el archivo 
`buildspec*.yml` deben contener líneas que hagan lo siguiente:

1. Hacer referencia a una imagen de contenedor de construcción con el tiempo de ejecución necesario de entre las 
imágenes disponibles. 
El siguiente ejemplo utiliza la imagen de contenedor de construcción `public.ecr.aws/sam/build-nodejs20.x`.
2. Configura las etapas del *pipeline* para ejecutar los comandos necesarios de la interfaz de línea de comandos 
(CLI) de AWS SAM.

En el siguiente ejemplo se ejecutan dos comandos de AWS SAM CLI: `sam build` y `sam deploy` (**con las opciones necesarias**).

Este ejemplo supone que has declarado todas las funciones y capas en tu archivo de plantilla de AWS SAM con `runtime: nodejs12.x`.

**AWS CloudFormation template snippet de codepipeline.yaml:**

```yaml
  CodeBuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      Environment:
        ComputeType: BUILD_GENERAL1_SMALL
        Image: public.ecr.aws/sam/build-nodejs20.x
        Type: LINUX_CONTAINER
      ...
```

**buildspec*.yml snippet:**

```shell
version: 0.2
phases:
  build:
    commands:
      - sam build
      - sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
```

Para obtener una lista de imágenes de contenedores de compilación de Amazon Elastic Container Registry (Amazon ECR) disponibles para diferentes tiempos de ejecución, 
[consulte Repositorios de imágenes para AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-image-repositories.html).


### Hacer las pruebas de la aplicación en entornos de desarrollo y producción.

Dirígete a la consola de CloudFormation. Después de que haya finalizado la ejecución de tu primer flujo de trabajo, 
notarás dos nuevas pilas sam-dev y sam-prod. Estos son los nombres que proporcionaste durante el asistente de 
SAM Pipelines en el paso anterior.

AWS Codepipeline creó el stack sam-dev durante el paso de despliegue de prueba del Pipeline. De manera similar, 
AWS Codepipeline creó sam-prod durante el paso de despliegue de producción.

Mira la pestaña Outputs de cada una de estas pilas de CloudFormation para ver los puntos finales de la API. 
Puedes usar curl u otros métodos para verificar la funcionalidad de tus dos nuevas APIs. Puedes exportar los puntos 
finales de URL para ambas etapas en una terminal.

```shell
export DEV_ENDPOINT=$(aws cloudformation describe-stacks --stack-name sam-app-dev | jq -r '.Stacks[].Outputs[].OutputValue | select(startswith("https://"))')
export PROD_ENDPOINT=$(aws cloudformation describe-stacks --stack-name sam-app-prod | jq -r '.Stacks[].Outputs[].OutputValue | select(startswith("https://"))')

echo "Dev endpoint: $DEV_ENDPOINT"
echo "Prod endpoint: $PROD_ENDPOINT"

curl -s $DEV_ENDPOINT
curl -s $PROD_ENDPOINT
```

![image_4.2.7.png](image_4.2.7.png)

Puede que hayas notado que las pruebas unitarias no se están ejecutando en tu canalización. ¡Arreglemos eso en la próxima sección!

## Habilitar pruebas unitarias

Para habilitar pruebas unitarias en tu canalización hay dos pasos.

1. Editar `codepipeline.yaml` y agregar comandos a la etapa de construcción de pruebas.
2. Realiza commit y push a estos cambios.

Abra el archivo `sam-app/codepipeline.yaml` en su editor. Busque la cadena `test`. Encontrará un comentario que indica 
dónde puede agregar comandos para ejecutar sus pruebas unitarias. Por favor, agregue los comandos que sean apropiados 
para el tiempo de ejecución que ha elegido.

```yaml
...
  CodeBuildProjectUnitTest:
    Type: AWS::CodeBuild::Project
    Properties:
      Artifacts:
        Type: CODEPIPELINE
      Environment:
        Type: LINUX_CONTAINER
        ComputeType: BUILD_GENERAL1_SMALL
        Image: public.ecr.aws/sam/build-nodejs20.x
      ServiceRole: !GetAtt CodeBuildServiceRole.Arn
      Source:
        Type: CODEPIPELINE
        BuildSpec: pipeline/buildspec_unit_test.yml

...
        - Name: UnitTest
          Actions:
            - Name: UnitTest
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: "1"
              Configuration:
                ProjectName: !Ref CodeBuildProjectUnitTest
              InputArtifacts:
                - Name: SourceCodeAsZip
...
```

### Haz commit y push de los cambios

```shell
cd ./sam-app-path
git commit -am 'Enable unit tests in pipeline'
git push
```

### Observa el Pipeline auto-actualizándose y ejecutando pruebas.

Abre tu Flujo de trabajo desde el panel de control de AWS CodePipeline. Si expandes el trabajo de prueba, entonces verás 
la salida de tus nuevas pruebas unitarias. Recuerda que cualquier cambio que hagas en el `codepipeline.yaml` se aplicará 
automáticamente una vez que lo hayas confirmado.

![image_unit-test.png](image_unit-test.png)

¡Felicidades! ¡Has creado un pipeline de Integración Continua/Despliegue Continuo para una aplicación Serverless!