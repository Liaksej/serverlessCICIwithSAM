# Despliegue manual con SAM

Antes de comenzar a construir un flujo de trabajo de integración y entrega continua (CI/CD) completamente automatizado, 
vamos a aprender cómo construir, empaquetar y implementar una aplicación sin servidor utilizando AWS SAM CLI.

![image_3.5.1.png](image_3.5.1.png)

Es importante aprender los fundamentos de cómo empaquetar e implementar una aplicación serverless aunque este taller 
enseñe cómo automatizar implementaciones con un pipeline CI/CD. Aprender cómo realizar implementaciones manuales 
también es útil para los desarrolladores que desean crear un stack personal no productivo.

## Artefactos

Los **artefactos** se refieren a la salida de tu proceso de compilación en el contexto de CI/CD. Los artefactos suelen 
ser en forma de un archivo zip/tar, imagen de contenedor o un binario, por ejemplo. Puedes tomar estos artefactos y 
desplegarlos en tus diferentes entornos (es decir, Dev, Test, Prod). Para proyectos _serverless_, los artefactos `.zip` 
deben ser subidos a un bucket S3 para que el servicio Lambda los recoja. El SAM CLI se encarga de gestionar este 
proceso de subida de artefactos a S3 y de hacer referencia a ellos en el momento del despliegue.

### El archivo `.zip`

El primer artefacto que se genera en un proyecto *serverless* es tu código de aplicación y las bibliotecas de soporte. 
Por defecto, estos se comprimen en un archivo `.zip` y se cargan en un bucket S3 por el SAM CLI durante la fase de 
empaquetado (más información sobre esto más adelante).

### Template empaquetado

El segundo artefacto que SAM CLI genera durante la fase de empaquetado es la plantilla empaquetada. 
Que es una copia de la `template.yaml` de tu proyecto, excepto que hace referencia a la ubicación del archivo `.zip` 
(primer artefacto) en el bucket de S3. La siguiente imagen muestra un ejemplo de una plantilla empaquetada.

![image_3.5.2.png](image_3.5.2.png)

Fíjate en cómo el CodeUri hace referencia al archivo `.zip` en un bucket de S3, en lugar de en un directorio local. 
De esta forma, AWS Lambda puede extraer tu código en el momento de la implementación.

## Build de app

Para construir un proyecto SAM, vamos a utilizar el comando `sam build`. Este comando itera a través de las funciones 
en tu aplicación, buscando el archivo de manifiesto (`package.json`) que contenga dependencias, y automáticamente crea 
los artefactos de despliegue.

Desde la raíz de la carpeta `sam-app`, ejecuta el siguiente comando en la terminal:

```shell
cd ~/sam-app
sam build
```

```
Building codeuri: /home/ec2-user/sam-app/hello-world runtime: nodejs20.x metadata: {} architecture: x86_64 functions: ['HelloWorldFunction']
Running NodejsNpmBuilder:NpmPack
Running NodejsNpmBuilder:CopyNpmrc
Running NodejsNpmBuilder:CopySource
Running NodejsNpmBuilder:NpmInstall
Running NodejsNpmBuilder:CleanUpNpmrc
Running NodejsNpmBuilder:LockfileCleanUp

Build Succeeded

Built Artifacts  : .aws-sam/build
Built Template   : .aws-sam/build/template.yaml

Commands you can use next
=========================
[*] Invoke Function: sam local invoke
[*] Test Function in the Cloud: sam sync --stack-name {stack-name} --watch
[*] Deploy: sam deploy --guided
```

### Build completada

Cuando la compilación finaliza con éxito, verás un nuevo directorio creado en la raíz del proyecto llamado `.aws-sam`. 
Es una carpeta oculta, así que si deseas verla en el IDE, **asegúrate de habilitar** `Mostrar archivos ocultos`.

![image_3.5.3.png](image_3.5.3.png)

### Explora la carpeta de construcción

Tómete un momento para explorar el contenido de la carpeta de construcción. Nota que las pruebas unitarias están 
automáticamente excluidas y las dependencias de terceros están incluidas. SAM se encarga de esto por nosotros.

![image_3.5.4.png](image_3.5.4.png)

La carpeta de construcción incluye el archivo app.js como el punto de entrada para la aplicación Lambda, 
el directorio `node_modules` con las dependencias, y el archivo `package.json` que declara las dependencias de la aplicación.

```
admin:~/node-sam-app $ ls -l .aws-sam/build/HelloWorldFunction/
total 8
-rw-r--r-- 1 ec2-user ec2-user 331 Oct 26  1985 app.js
drwxrwxr-x 5 ec2-user ec2-user  57 Mar  9 22:35 node_modules
-rw-r--r-- 1 ec2-user ec2-user 468 Oct 26  1985 package.json
```

## Deploy de aplicación

El comando `sam deploy` despliega tu aplicación lanzando una pila de CloudFormation. Este comando tiene un modo 
interactivo guiado, que puedes habilitar especificando el parámetro `--guided`. Se recomienda desplegar en modo 
guiado la primera vez, ya que capturará la configuración para futuros despliegues en un nuevo archivo `samconfig.toml` 
descrito al final de esta sección.

Ejecuta el siguiente comando en el mismo nivel de directorio donde se encuentra el archivo `template.yaml`:

```shell
sam deploy --guided
```


Este proceso te guiará a través de una serie de preguntas. Tus respuestas se guardarán en el archivo de configuración 
al final, lo que acelerará despliegues futuros. Presionar la tecla `Enter` aceptará el valor predeterminado mostrado 
entre corchetes para cada pregunta, por ejemplo `[sam-app]`. Las letras en MAYÚSCULAS son los valores predeterminados, 
por ejemplo, presionar enter para `[y/N]` resultará en `No` por defecto.

> **Autorización faltante**  
> Asegúrate de responder y a la pregunta sobre la autorización faltante: `HelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y`

```
Configuring SAM deploy
======================

   Looking for config file [samconfig.toml] :  Not found

   Setting default arguments for 'sam deploy'
   =========================================
   Stack Name [sam-app]:
   AWS Region [us-west-2]:
   #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
   Confirm changes before deploy [y/N]:
   #SAM needs permission to be able to create roles to connect to the resources in your template
   Allow SAM CLI IAM role creation [Y/n]:
   #Preserves the state of previously provisioned resources when an operation fails
   Disable rollback [y/N]:
   HelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
   Save arguments to configuration file [Y/n]:
   SAM configuration file [samconfig.toml]:
   SAM configuration environment [default]:
```

### Despliegue completado

Este comando puede tardar unos minutos en completarse porque está creando los recursos (función Lambda, API Gateway 
y roles de IAM) en la cuenta de AWS. Cuando se complete con éxito, deberías ver una salida similar a la siguiente:

```
CloudFormation outputs from deployed stack
---------------------------------------------------------------------------------------------------------------------------
Outputs
---------------------------------------------------------------------------------------------------------------------------
Key                 HelloWorldFunctionIamRole
Description         Implicit IAM Role created for Hello World function
Value               arn:aws:iam::123456789123:role/sam-app-node-HelloWorldFunctionRole-1QWE9DKNCP6V3

Key                 HelloWorldApi
Description         API Gateway endpoint URL for Prod stage for Hello World function
Value               https://01111gpgpg.execute-api.us-west-2.amazonaws.com/Prod/hello/

Key                 HelloWorldFunction
Description         Hello World Lambda Function ARN
Value               arn:aws:lambda:us-west-2:123456789123:function:sam-app-node-HelloWorldFunction-5RxoDCiBWUPV
---------------------------------------------------------------------------------------------------------------------------
```

Nota el valor de salida para `HelloWorldApi`. El Valor es el punto de conexión HTTPS para tu nueva Puerta de Enlace 
de API. Puedes cargar esta URL en tu navegador o hacer curl en una terminal.

```shell
curl -s (insert HelloWorldApi Value)
```

```
curl -s https://01111gpgpg.execute-api.us-west-2.amazonaws.com/Prod/hello/

{"message":"hello my friend"}
```

### ¿Qué acaba de suceder?

El despliegue guiado hace pocas cosas por ti. Echemos un vistazo rápido a lo que sucedió bajo el capó durante 
el despliegue guiado para entender mejor este proceso.

1. Tu codebase se empaqueta como un archivo `.zip`.
2. SAM crea un bucket de S3 en tu cuenta, si este aún no existe.
3. El archivo `.zip` se ha subido al depósito S3.
4. SAM ha creado el [packaged template](https://catalog.workshops.aws/complete-aws-sam/en-US/module-3-manual-deploy/10-bucket.md#the-packaged-template) que se refiere a una ubicación del archivo `.zip` en S3.
5. El template empaquetado también se ha subido al bucket S3.
6. SAM inicia el despegue a través de los conjuntos de cambios de CloudFormation.

La primera vez que realices un despegue guiado, se creará un nuevo archivo `samconfig.toml` en la raíz de 
tu proyecto con tus parámetros de implementación especificados. Este archivo acelera los futuros comandos 
de implementación de sam al utilizar los mismos parámetros, sin necesidad de que los ingreses nuevamente.

```
version = 0.1
[default]
[default.deploy]
[default.deploy.parameters]
stack_name = "sam-app"
s3_bucket = "aws-sam-cli-managed-default-samclisourcebucket-3ma345ba4af33"
s3_prefix = "sam-app"
region = "us-west-2"
capabilities = "CAPABILITY_IAM"
image_repositories = []
```

> Puedes obtener más información sobre las implantaciones guiadas y el archivo `samconfig.toml` con [esta entrada de blog](https://aws.amazon.com/blogs/compute/a-simpler-deployment-experience-with-aws-sam-cli).

## Inspeccionando el despliegue

En la sección anterior, desplegamos y probamos nuestra aplicación sin servidor. Si eres nuevo en AWS SAM, 
es bueno entender cómo crea recursos con CloudFormation.

### Abra la consola de CloudFormation

Navega a la [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home), asegúrate de que estás en 
la misma región en la que has estado trabajando hasta ahora. Deberías ver el nuevo stack sam-app en el estado `CREATE_COMPLETE`.

![image_3.5.5.png](image_3.5.5.png)

### Salidas de CloudFormation

Recuerda que nuestro plantilla de AWS SAM define tres resultados al final del archivo `template.yaml`.

```yaml
Outputs:
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
  HelloWorldFunction:
    Description: "Hello World Lambda Function ARN"
    Value: !GetAtt HelloWorldFunction.Arn
  HelloWorldFunctionIamRole:
    Description: "Implicit IAM Role created for Hello World function"
    Value: !GetAtt HelloWorldFunctionRole.Arn
```

Puedes ver en la sección anterior cómo los valores de estas variables se emiten a la consola durante el despliegue. 
Es útil reconocer que se trata de [CloudFormation Outputs estándar](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html).

Puedes ver estos valores de **Outputs** en la consola de CloudFormation. Haz clic en la pila `sam-app` y luego vete 
a la pestaña `Outputs`. En esta pestaña, verás la URL de la API Gateway, el ARN de la función Lambda y el ARN 
del Rol IAM para la función.











