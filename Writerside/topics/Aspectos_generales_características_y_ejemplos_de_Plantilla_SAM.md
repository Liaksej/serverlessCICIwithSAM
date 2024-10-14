# Plantilla SAM

El Hello World Example TypeScript que acabas de inicializar crea una función Lambda y una API Gateway que expone 
un recurso `/hello`. Cuando se llama con una solicitud GET de HTTP, la API Gateway invoca la función que asume un rol
de ejecución IAM con permisos para interactuar con otros recursos de AWS, como por ejemplo una base de datos.

![image](image_3.2.1.png)

### Explora la plantilla SAM

Tomémonos un momento para entender la estructura de una aplicación SAM explorando la plantilla SAM que representa 
la arquitectura de la aplicación _serverless_. Abre el archivo `sam-app/template.yaml`.

Debe tener una estructura como la siguiente. Esta es una aplicación Node y su template.yaml se verá ligeramente 
diferente si se usa un runtime distinto.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Description: >
  sam-app

  Sample SAM Template for sam-app

Globals:
  Function:
    Timeout: 3
    MemorySize: 128

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: hello-world/app.lambdaHandler
      Runtime: nodejs20.x
      Architectures:
        - x86_64
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
    Metadata: # Manage esbuild properties
      BuildMethod: esbuild
      BuildProperties:
        Minify: true
        Target: ES2022
        Sourcemap: true
        KeepNames: true
        Format: esm
        SourcesContent: true
        MainFields: module,main
        TreeShaking: true
        EntryPoints:
          - hello-world/app.ts
        External:
          - '@aws-lambda-powertools/*'
          - '@aws-sdk/*'
            
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
Puedes notar que la sintaxis se parece exactamente a AWS CloudFormation, esto se debe a que las plantillas SAM son 
una extensión de las plantillas de CloudFormation. Es decir, **cualquier recurso que puedas declarar en CloudFormation, 
también puedes declararlo en una plantilla SAM**. Vamos a analizar más de cerca los componentes de la plantilla.

### Transformación

Observe la línea de transformación de la plantilla resaltada a continuación. Esta línea indica a CloudFormation que 
la plantilla se adhiere al AWS SAM [specification](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md):

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
```

### Globals

Esta sección define propiedades comunes a todas sus **funciones serverless** y **APIs**. En este caso, especifica 
que todas las funciones en este proyecto tendrán un tiempo de espera predeterminado de 3 segundos y una memoria 
predeterminada de 128 MB.

```yaml
Globals:
  Function:
    Timeout: 3
    Memory : 128
```

### Función Hello World

La siguiente sección crea una función Lambda con un rol de ejecución IAM. También especifica que el código de 
esta función Lambda se encuentra en una carpeta especificada en la clave CodeUri. La clave Handler define el archivo 
y nombre de la función del punto de entrada.

```yaml
HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: hello-world/app.lambdaHandler
      Runtime: nodejs20.x
      Architectures:
        - x86_64
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
    Metadata: # Manage esbuild properties
      BuildMethod: esbuild
      BuildProperties:
        Minify: true
        Target: ES2022
        Sourcemap: true
        KeepNames: true
        Format: esm
        SourcesContent: true
        MainFields: module,main
        TreeShaking: true
        EntryPoints:
          - hello-world/app.ts
        External:
          "*"
```
Observa que el **IAM role no está especificado explícitamente**, esto se debe a que SAM creará uno nuevo por defecto.
Puedes anular este comportamiento y pasar tu propio role especificando el parámetro `Role`. 
Para obtener una lista completa de los parámetros que puede especificar para una función Lambda, consulte la página SAM 
[referencia](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction).

### Desencadenadores de Eventos

La sección de **Eventos** es parte de la definición de la función. Esta sección especifica los diferentes eventos que 
desencadenarán la función Lambda. En este caso, estamos especificando una solicitud GET HTTP a API Gateway
con un extremo de `/hello`.

```yaml
HelloWorldFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: hello-world/
    Handler: app.lambdaHandler
    Runtime: nodejs20.x
    Events:
      HelloWorld:
        Type: Api
        Properties:
          Path: /hello
          Method: get
```

### Outputs

La sección Outputs es opcional y declara los valores de salida que puedes importar a otras pilas de CloudFormation 
(para crear referencias entre pilas), o simplemente para verlos en la consola de CloudFormation. En este caso, 
estamos haciendo disponibles como Outputs la URL del punto final de la API Gateway, el ARN de 
la función Lambda, y el ARN del rol de IAM para facilitar su localización.

```yaml
Outputs:
  # ServerlessRestApi is an implicit API created out of Events key under Serverless::Function
  # Find out more about other implicit resources you can reference within SAM
  # https://github.com/awslabs/serverless-application-model/blob/master/docs/internals/generated_resources.rst#api
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

## Recursos adicionales

* [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
* [AWS SAM resources and properties](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-resources-and-properties.html)
