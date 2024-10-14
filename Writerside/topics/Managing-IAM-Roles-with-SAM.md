# Administrar roles de IAM con SAM

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/what-is-an-iam-role.png)

Hasta ahora no hemos hablado qué se permite hacer a la función Lambda o cómo controlar el acceso a otros servicios
o recursos de AWS. Debido a que la aplicación de muestra y la función Lambda simplemente devuelven un mensaje
de 'Hello World, es posible asumir que no depende de otros servicios de AWS. Esta es una suposición natural,
pero la función sí depende de al menos otro servicio de AWS. ¿Puedes adivinar cuál es?

En este capítulo, aprenderás diferentes formas de otorgar acceso de privilegio mínimo a los recursos de AWS desde
tus funciones Lambda. Al final, tendrás un entendimiento de los tres mecanismos diferentes para gestionar
el acceso de Lambda:

* Plantillas de directivas de AWS SAM
* Conectores de AWS SAM
* Políticas de IAM escritas a mano

## IAM Execution Role

Antes de empezar, veamos cómo Lambda es capaz, o no es capaz, de acceder a diferentes servicios y recursos de AWS.

A continuación se muestra el archivo `template.yaml` del ejemplo anterior. Observa que no se ve ningún
permiso explícito para la función Lambda:

```yaml
Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello-world/
      Handler: app.handler
      Runtime: nodejs20.x
      Architectures:
        - x86_64
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
      ...
    Metadata:
      BuildMethod: esbuild
      BuildProperties:
        Format: esm
        ...
```

Por defecto, las funciones de Lambda escriben registros en AWS CloudWatch Logs. Si Lambda escribe en CloudWatch
sin permiso explícito de parte tuya, ¿cómo está funcionando esto?

Cada función Lambda tiene un rol de ejecución de AWS Identity and Access Management (IAM) llamado un rol de ejecución.
Un rol de ejecución otorga a la función permiso para acceder a servicios y recursos de AWS. En el ejemplo "Hello World"
que has estado utilizando, SAM ha creado un rol de ejecución para ti con los permisos mínimos para acceder
a CloudWatch Logs.

Vamos a echar un vistazo al rol de ejecución de esa función en la consola de AWS.

<procedure title="Procedimiento">
    <step>Abra la página <a href="https://console.aws.amazon.com/lambda/home#/functions">Functions page of the Lambda console</a></step>
    <step>Localiza la función en la lista (utiliza la barra de búsqueda si es necesario) y haz clic sobre ella.</step>
    <img src="https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/select-lambda-function.png"/>
    <step>Haz clic en la pestaña "Configuración", y luego selecciona "Permisos" en el menú de la izquierda.</step>
    <img src="https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/select-configuration-tab.png"/>
    <step>En la sección de resumen de Recursos, elige un servicio de la lista desplegable para ver los permisos relacionados
   con dicho servicio. En este caso, solo debería haber un servicio enlistado, Amazon CloudWatch Logs.</step>
    <img src="https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/resource-summary-1.png"/>
    <step>Bajo el resumen de recursos, revisa los servicios y recursos a los que la <b>función</b> puede acceder.
   En el cuadro de alerta de información, vemos que estos permisos provienen de una Política Administrada de AWS,
   <code>AWSLambdaBasicExecutionRole</code>.</step>
    <img src="https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/resource-summary-2.png"/>
    <step>Por último, haga clic en el enlace del rol de ejecución para abrir la consola IAM.</step>
    <img src="https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/open-iam-console.png"/>
    <img src="https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/lambda-execution-role.png"/>
</procedure>

Este Rol de IAM define a qué puedes acceder tu función Lambda. IAM trabaja en base al principio de privilegio mínimo:
sin permisos explícitos, tu función Lambda no podrá acceder a otros recursos de AWS. El rol de ejecución puede tener
cero o más políticas adjuntas a él. En las secciones siguientes aprenderás cómo gestionar este rol de ejecución
y añadir tus propias políticas.

En este ejemplo, hay una única política llamada `AWSLambdaBasicExecutionRole`. `AWSLambdaBasicExecutionRole` es
una [AWS managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies)
creada y administrada por AWS. Las políticas gestionadas de AWS están diseñadas para proporcionar
permisos para muchos casos de uso comunes. Cada función de Lambda necesita un conjunto mínimo de permisos para utilizar
logs de CloudWatch. El `AWSLambdaBasicExecutionRole` define estos permisos que, por defecto, se aplican a
todas las funciones Lambda.

Pero, ¿qué sucede cuando tu función Lambda necesita acceso a otros recursos de AWS como una tabla DynamoDB?
La respuesta es que la función requerirá permisos adicionales, como `dynamodb:Query` o `dynamodb:Scan`.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/lambda-dynamodb-permissions.png)

Para permitir que la función Lambda acceda a DynamoDB, debes ampliar la definición de la función en `template.yaml`
añadiendo los permisos necesarios. Una vez que se añadan los permisos correctos de `dynamoddb:` al rol de ejecución de
la función, esta podrá realizar acciones en tu tabla de DynamoDB.

Para demostrar las diferentes opciones para especificar permisos IAM en SAM, actualizarás la muestra de la aplicación
para que lea y escriba desde y hacia una tabla de DynamoDB.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/sam-workshop-connectors-architecture.png)

## Inicializa el proyecto

> **Asegúrate de seleccionar la opción correcta**  
> A continuación mostramos la opción API _serverless_ como opción 3. A medida que se publiquen nuevas plantillas
> de inicio rápido, la ubicación exacta de esta opción puede cambiar. Si estás interesado en explorar las plantillas
> de inicio rápido, puedes encontrarlas
> en [AWS SAM CLI Application Templates](https://github.com/aws/aws-sam-cli-app-templates)

Ejecuta el siguiente comando para generar la estructura básica de un nuevo proyecto:

```shell
sam init
```

En el asistente, selecciona

* `AWS QuickSart Templates` y luego
* `Serverless API`

No utilices el atajo para usar la última versión de Python. _Ten en cuenta que las respuestas numeradas pueden ser
distintas en tu terminal._

```
Which template source would you like to use?
        1 - AWS Quick Start Templates <---- SELECT THIS
        2 - Custom Template Location
Choice: 1

Choose an AWS Quick Start application template
        1 - Hello World Example
        2 - Data processing
        3 - Hello World Example with Powertools for AWS Lambda
        4 - Multi-step workflow
        5 - Scheduled task
        6 - Standalone function
        7 - Serverless API <---- SELECT THIS TEMPLATE
        ...
Template: 7

Use the most popular runtime and package type? (Python and zip) [y/N]: n
```

A continuación, selecciona tu runtime y versión preferidos. Para Node.js, elige `nodejs20.x`.

```
Which runtime would you like to use?
        1 - dotnet6
        2 - nodejs20.x
        3 - nodejs18.x
        4 - nodejs16.x
Runtime: 2
```

Seleccione 'no' para trazado de X-ray y deje `sam-app` como el `nombre del proyecto`.

```
Based on your selections, the only Package type available is Zip.
We will proceed to selecting the Package type as Zip.

Based on your selections, the only dependency manager available is npm.
We will proceed copying the template using npm.

Would you like to enable X-Ray tracing on the function(s) in your application?  [y/N]: n

Project name [sam-app]:
```

Deberías ver ahora una nueva carpeta, `sam-app`, que contiene el andamiaje generado para el proyecto.
Observa que hay tres archivos de JavaScript, cada uno de los cuales representa una función Lambda y
una responsabilidad específica.

* `get-all-items.js` -> obtiene todos los elementos almacenados en la tabla de DynamoDB
* `get-by-id.js` -> recupera un elemento por su ID único
* `put-item.js` -> escribir un elemento en la tabla de DynamoDB

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/project-structure.png)

Siéntate libre de abrir cualquiera de estos archivos para ver cómo se utiliza el SDK de AWS para acceder a DynamoDB.

## Despliegue de la aplicación

Dado que es la primera vez que estás implementando esta aplicación, utilizarás la bandera `--guided` con el comando
`sam deploy`.

```shell
cd ~/environment/sam-app
sam build
sam deploy --guided
```

> **Autorización faltante**  
> Asegúrate de responder y a la pregunta sobre si las funciones Lambda no tienen definida la autorización:
> `The function getAllItemsFunction may not have defined autorization, Is it ok? [y/N]: y`

### Despliegue completado

Este comando tardará unos minutos en finalizar porque está creando los recursos (funciones Lambda, Puerta de enlace API
y roles IAM, etc.). Después de que la implementación se complete con éxito, verás una salida similar a la siguiente:

```
CloudFormation outputs from deployed stack
---------------------------------------------------------------------------------------------------------------------------
Outputs
---------------------------------------------------------------------------------------------------------------------------
Key                 WebEndpoint
Description         API Gateway endpoint URL for Prod stage
Value               https://q9kbh1buk3.execute-api.us-west-2.amazonaws.com/Prod/
---------------------------------------------------------------------------------------------------------------------------
```

Exporta el nuevo endpoint HTTPS:

```shell
export ENDPOINT=$(aws cloudformation describe-stacks --stack-name sam-app | \
    jq -r '.Stacks[].Outputs[].OutputValue | select(startswith("https://"))')
```

### Probar el endpoint

Como comprobación de funcionamiento, realiza una solicitud a tu nueva aplicación sin servidor para asegurarte de que
esté funcionando. Dado que tu nueva API está leyendo de una tabla DynamoDB, puedes esperar una respuesta vacía.

```shell
curl -s $ENDPOINT
```

La respuesta será un array vacío.

```shell
[]
```

### Añadimos datos

Agrega algunos elementos a la tabla DynamoDB para que tengas datos para consultar. Esto ejecuta la API putItem con tres
elementos únicos.

```shell
curl -d "{\"id\": \"id1\",\"name\": \"name1\"}" -X POST $ENDPOINT
curl -d "{\"id\": \"id2\",\"name\": \"name2\"}" -X POST $ENDPOINT
curl -d "{\"id\": \"id3\",\"name\": \"name3\"}" -X POST $ENDPOINT
```

Utiliza la API `getAllItems` para ver los elementos que acabas de agregar:

```shell
curl -s $ENDPOINT | jq
```

Deberías ver una lista de elementos.

```json
[
  {
    "id": "id3",
    "name": "name3"
  },
  {
    "id": "id1",
    "name": "name1"
  },
  {
    "id": "id2",
    "name": "name2"
  }
]
```

Finalmente, obtén elementos individuales con la API `getItemById`:

```shell
curl -s $ENDPOINT/id1 | jq
curl -s $ENDPOINT/id2 | jq
curl -s $ENDPOINT/id3 | jq
```

## SAM Policy Templates

[SAM Policy Templates](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html)
son accesos directos que facilitan la adición de permisos de ámbito a funciones Lambda y flujos de trabajo de
flujos de trabajo de Step Functions. Son específicas de SAM y pueden facilitar el trabajo de gestión de permisos de IAM.
El equipo de AWS SAM mantiene una serie de plantillas de políticas para integraciones comunes como DynamoDB y S3.
En este módulo aprenderá cómo funcionan las plantillas de políticas de SAM y cómo utilizarlas en sus propias
aplicaciones.

### Tu primera plantilla de política

En la última sección se utilizó el API para leer y escribir datos desde y hacia una tabla de DynamoDB. SAM crea
automáticamente un rol de ejecución básico para cada función Lambda en su aplicación SAM que permite a las
funciones acceder a CloudWatch Logs. Pero en esta nueva aplicación, ¿cómo acceden las funciones Lambda a DynamoDB?

Abre el archivo `template.yaml` y mira la sección de 'Policies' en la definición de la función `getAllItemsFunction`.

```yaml
 # This is a Lambda function config associated with the source code: get-all-items.js
 getAllItemsFunction:
   Type: AWS::Serverless::Function
   Properties:
     Handler: src/handlers/get-all-items.getAllItemsHandler
     Runtime: nodejs18.x
     Architectures:
       - x86_64
     MemorySize: 128
     Timeout: 100
     Description: A simple example includes a HTTP get method to get all items from a DynamoDB table.
     Policies:
       # Give Create/Read/Update/Delete Permissions to the SampleTable
       - DynamoDBCrudPolicy:
           TableName: !Ref SampleTable
     Environment:
       Variables:
         # Make table name accessible as environment variable from function code during execution
         SAMPLE_TABLE: !Ref SampleTable
     Events:
       Api:
         Type: Api
         Properties:
           Path: /
           Method: GET
```

Hay qye tener en cuenta que `DynamoDBCrudPolicy` aparece en la propiedad `Policies`. El nombre clave, `DynamoDBCrudPolicy`,
hace referencia a
una [plantilla de política específica](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-template-list.html#dynamo-db-crud-policy).
Esta plantilla de política espera un valor para la variable `TableName`
que SAM utiliza para añadir una serie de permisos IAM al rol de ejecución de su Lambda. Como puedes suponer
SAM añade acceso de creación, lectura, actualización y eliminación (CRUD) para que la función pueda realizar
las siguientes acciones en la tabla `SampleTable` de DynamoDB:

* put items
* get items
* scan items

### Eliminar permisos

A continuación, eliminarás estos permisos y verás los efectos.

Quita la propiedad `DynamoDBCrudPolicy` comentándola en el archivo `template.yaml`.

```yaml
# This is a Lambda function config associated with the source code: get-all-items.js
getAllItemsFunction:
  Type: AWS::Serverless::Function
  Properties:
    Handler: src/handlers/get-all-items.getAllItemsHandler
    # ...
    #      Policies:
    #        - DynamoDBCrudPolicy:
    #            TableName: !Ref SampleTable
    # ...
```

A continuación, realiza los siguientes cambios en el código de la función Lambda get-all-items.js. En el bloque `catch`,
agrega la excepción a la respuesta para que tengamos mejor información diagnóstica si hay un error. Las líneas
resaltadas muestran los cambios a realizar. También está bien copiar y pegar el bloque de código completo
a continuación y reemplazar la función existente.

```javascript
// Create clients and set shared const values outside of the handler.

// Create a DocumentClient that represents the query to add an item
import {DynamoDBClient, ResourceNotFoundException} from '@aws-sdk/client-dynamodb';
import {DynamoDBDocumentClient, ScanCommand} from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddbDocClient = DynamoDBDocumentClient.from(client);

// Get the DynamoDB table name from environment variables
const tableName = process.env.SAMPLE_TABLE;

// A simple example includes a HTTP get method to get all items from a DynamoDB table.
export const getAllItemsHandler = async (event) => {
  if (event.httpMethod !== 'GET') {
    throw new Error(`getAllItems only accept GET method, you tried: ${event.httpMethod}`);
  }
  // All log statements are written to CloudWatch
  console.info('received:', event);

  // get all items from the table (only first 1MB data, you can use `LastEvaluatedKey` to get the rest of data)
  // https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#scan-property
  // https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Scan.html
  var params = {
    TableName: tableName
  };

  let response = {}

  try {
    const data = await ddbDocClient.send(new ScanCommand(params));
    response = {
      statusCode: 200,
      body: JSON.stringify(data.Items)
    };
  } catch (ResourceNotFoundException) {
    response = {
      statusCode: 404,
      body: "Unable to call DynamoDB. Table resource not found. " + ResourceNotFoundException
    };
  }
  ;

  // All log statements are written to CloudWatch
  console.info(`response from: ${event.path} statusCode: ${response.statusCode} body: ${response.body}`);
  return response;
}
```

A continuación, construye e implementa la aplicación.

```shell
cd ~/environment/sam-app
sam build && sam deploy
```

Después de que se complete el despliegue, haz una solicitud a la API `getAllItems`. Verás un error que se ve similar
al siguiente, con detalles para tu función Lambda y tabla DynamoDB:

```shell
curl -s $ENDPOINT
```

```
Unable to call DynamoDB. Table resource not found. AccessDeniedException: ...
is not authorized to perform: dynamodb:Scan on resource: ... because no identity-based policy allows the dynamodb:Scan action
```

Como era de esperar, la función Lambda `getAllItemsFunction` lanza un error porque ya no tiene permiso para escanear la
tabla DynamoDB.

Corregiremos la función descomentando el `DynamoDBCrudPolicy` para que nuevamente tenga los permisos requeridos.

> **Nota**  
> Para entender las SAM Policy Templates, se necesita cierta familiaridad con
> la [sintaxis de políticas de IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html).
> Aunque las plantillas de políticas de SAM son prácticas, no sustituyen la lectura y comprensión de las políticas de
> IAM.

### Principio de privilegios mínimos

Abre los otros dos archivos JavaScript y observa las declaraciones de las políticas. Observa que también hacen
referencia a la política `DynamoDBCrudPolicy`. Cada función necesita permisos específicos (lectura, escritura o escaneo)
para realizar sus tareas, pero `DynamoDBCrudPolicy` otorga todos estos permisos. Podemos restringir el acceso a
la tabla de DynamoDB aún más con diferentes plantillas de políticas SAM, al mismo tiempo que permitimos que cada
función realice su tarea. Si observas la lista de plantillas de políticas SAM, encontrarás algunas plantillas más
adecuadas para delimitar los permisos de nuestras tres funciones.

| Function       | SAM Policy Template |
|----------------|---------------------|
| put-item       | DynamoDBWritePolicy |
| get-item-by-id | DynamoDBReadPolicy  |
| get-all-items  | DynamoDBReadPolicy  |

En el archivo `template.yaml`, cambie la política de cada función a la política más específica enumerada arriba.
Ten en cuenta que solo se muestran los cambios en el `.yaml` a continuación.

```yaml
 getAllItemsFunction:
   Type: AWS::Serverless::Function
   Properties:
     Handler: src/handlers/get-all-items.getAllItemsHandler
     ...
     Policies:
       - DynamoDBReadPolicy:
           TableName: !Ref SampleTable
     ...

   getByIdFunction:
     Type: AWS::Serverless::Function
     Properties:
       Handler: src/handlers/get-by-id.getByIdHandler
       ...
       Policies:
         - DynamoDBReadPolicy:
             TableName: !Ref SampleTable
       ...

   putItemFunction:
     Type: AWS::Serverless::Function
     Properties:
       Handler: src/handlers/put-item.putItemHandler
       ...
       Policies:
         - DynamoDBWritePolicy:
             TableName: !Ref SampleTable
       ...
```

Después de haber realizado estos cambios, vuelve a implementar la aplicación y verifica que nuestras APIs sigan
funcionando.

```shell
cd ~/sam-app
sam build && sam deploy
```

Prueba primero el API de colocar-artículos:

```shell
curl -d "{\"id\": \"id4\",\"name\": \"name4\"}" -X POST $ENDPOINT
```

A continuación, prueba la API `get-all-items`:

```shell
curl -s $ENDPOINT | jq 'sort_by(.id)'
```

El ítem 4 ahora está en la lista.

```json
[
  {
    "id": "id1",
    "name": "name1"
  },
  {
    "id": "id2",
    "name": "name2"
  },
  {
    "id": "id3",
    "name": "name3"
  },
  {
    "id": "id4",
    "name": "name4"
  }
]
```

Y finalmente, prueba la API 'get-item-by-id':

```shell
curl -s $ENDPOINT/id1 | jq
```

```json
{
  "id": "id1",
  "name": "name1"
}
```

Acabas de aprender cómo las Plantillas de Políticas de SAM facilitan la concesión de acceso a funciones Lambda
a recursos específicos de AWS sin necesidad de escribir manualmente políticas de IAM.

## Políticas en línea y administradas

En [el apartado anterior](Aspectos_generales_características_y_ejemplos_de_Plantilla_SAM.md) aprendiste que las plantillas de políticas SAM son una forma sencilla de conceder acceso
a diferentes recursos de AWS desde tus funciones Lambda. Por muy útiles y fáciles que sean, a veces puede que necesites
ser más específico en la forma de gestionar el acceso IAM. En otras circunstancias puede que no haya una plantilla
de política que se ajuste a lo que necesitas. También podría haber
una [política administrada por AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies)
que podrías utilizar para simplificar tu aplicación.

SAM te permite elaborar tus propias Políticas de IAM tal como lo harías en CloudFormation sin procesar.
También te permite hacer referencia a las políticas administradas por AWS por nombre o ARN. En este módulo aprenderás
cómo agregar tus propias políticas de bajo nivel y políticas administradas por AWS en tus aplicaciones SAM.

### Crear una Política en línea

En [el apartado anterior](Despliegue_manual_con_SAM.md), usaste la plantilla de política SAM `DynamoDBCrudPolicy` para otorgar acceso CRUD a una tabla
de DynamoDB.

```yaml
  # This is a Lambda function config associated with the source code: get-all-items.js
  getAllItemsFunction:
    ...
    Policies:
      # Give Create/Read/Update/Delete Permissions to the SampleTable
        - DynamoDBCrudPolicy:
        TableName: !Ref SampleTable
    ...
```

En ese párrafo, la propiedad `Policies` hace referencia a una plantilla de directiva SAM única. Sin embargo,
`Policies` también puede incluir una directiva IAM en línea.

Realice el siguiente cambio en su archivo `template.yaml`, reemplazando la declaración `DynamoDBCrudPolicy` por
la política IAM en línea. Las líneas resaltadas a continuación muestran el cambio a realizar.

```yaml
# This is a Lambda function config associated with the source code: get-all-items.js
getAllItemsFunction:
  Type: AWS::Serverless::Function
  Properties:
    Handler: src/handlers/get-all-items.getAllItemsHandler
    Runtime: nodejs16.x
    Architectures:
      - x86_64
    MemorySize: 128
    Timeout: 100
    Description: A simple example includes a HTTP get method to get all items from a DynamoDB table.
    Policies:
      - Statement:
          - Sid: DynamoDBReadPolicy
            Effect: Allow
            Action:
              - dynamodb:Scan
            Resource:
              - Fn::Sub:
                  - "arn:${AWS::Partition}:dynamodb:${AWS::Region}:${AWS::AccountId}:table/${tableName}"
                  - tableName: !Ref SampleTable
    Environment:
      Variables:
        # Make table name accessible as environment variable from function code during execution
        SAMPLE_TABLE: !Ref SampleTable
    Events:
      Api:
        Type: Api
        Properties:
          Path: /
          Method: GET
```

Ve adelante y realiza este cambio en el `template.yaml` y vuelve a implementar la aplicación.

```shell
cd ~/sam-app
sam build && sam deploy
```

Prueba el endpoint después de que se complete el despliegue para ver cómo la función Lambda todavía tiene
acceso de `dynamodb:Scan` a la tabla de DynamoDB.

```shell
curl -s $ENDPOINT | jq 'sort_by(.id)'
```

```json
[
  {
    "id": "id1",
    "name": "name1"
  },
  {
    "id": "id2",
    "name": "name2"
  },
  {
    "id": "id3",
    "name": "name3"
  },
  {
    "id": "id4",
    "name": "name4"
  }
]
```

### Expandir una política en línea

Puedes ver cómo este patrón es fácil de extender cuando tus funciones necesitan políticas IAM específicas.
El ejemplo a continuación muestra cómo podrías otorgar permisos de `DescribeLogGroups` y `DescribeLogStreams` a
tu función Lambda. Ten en cuenta que el campo de `Resource` hace referencia a los grupos de registros de CloudWatch
que comienzan con `/aws/lambda/lambd-`.

```yaml
getAllItemsFunction:
  Type: AWS::Serverless::Function
  Properties:
    Handler: src/handlers/get-all-items.getAllItemsHandler
    ...
    Policies:
      - Statement:
          - Sid: DynamoDBReadPolicy
            Effect: Allow
            Action:
              - dynamodb:Scan
            Resource:
              - Fn::Sub:
                  - "arn:${AWS::Partition}:dynamodb:${AWS::Region}:${AWS::AccountId}:table/${tableName}"
                  - tableName: !Ref SampleTable
          - Sid: ViewLogs
            Effect: Allow
            Action:
              - logs:DescribeLogGroups
              - logs:DescribeLogStreams
            Resource: !Sub "arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/lambda-*"
```

### Utiliza políticas administradas de AWS

Las políticas IAM gestionadas son administradas por AWS. Estas políticas son genéricas y cómodas,
simplificando la concesión de acceso a un amplio conjunto de recursos para casos de uso comunes. Puede consultar
la [lista completa de políticas administradas en la Guía de Referencia](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/CloudWatchEventsFullAccess.html).
Dos ejemplos concretos son `AmazonEC2ReadOnlyAccess` y `CloudWatchEventsFullAccess`.

Imagina una función Lambda que necesita leer detalles de todas tus instancias de EC2. Agregar una política en línea
con privilegios mínimos sería tedioso si hubiera docenas (¡o cientos!) de instancias de EC2. En estos casos, agregar
una política administrada es fácil y eficiente. ¡Siempre ten en cuenta tu postura de seguridad al añadir políticas
administradas!
Probablemente no querrías otorgar `ec2:TerminateInstances` a una función Lambda para todas tus instancias.

El ejemplo a continuación muestra cómo puedes agregar la política administrada `AmazonEC2ReadOnlyAccess` junto con
tus declaraciones de política en línea para otorgar acceso de lectura de EC2 a esta Lambda Function.

```yaml
  getAllItemsFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-all-items.getAllItemsHandler
      ...
      Policies:
        - arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
        - Statement:
            - Sid: DynamoDBReadPolicy
              Effect: Allow
              Action:
                - dynamodb:Scan
              Resource:
                - Fn::Sub:
                    - "arn:${AWS::Partition}:dynamodb:${AWS::Region}:${AWS::AccountId}:table/${tableName}"
                    - tableName: !Ref SampleTable
            - Sid: ViewLogs
              Effect: Allow
              Action:
                - logs:DescribeLogGroups
                - logs:DescribeLogStreams
              Resource: !Sub "arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/lambda-*"

```

Si agregas la política administrada `AmazonEC2ReadOnlyAccess`, navega hasta el rol de ejecución IAM de la función para
verla agregada.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/connectors/add-aws-managed-policy.png)

Acabas de aprender cómo agregar tus propias Declaraciones de Política IAM para dar acceso a tus funciones Lambda
a los recursos de AWS. Aprendiste cómo estas son diferentes de las Plantillas de Política SAM, y cómo usar la política
gestionada de AWS junto con otras declaraciones de política.

## SAM Connectors

Además de facilitar la creación, construcción, prueba e implementación de aplicaciones _serverless_, AWS SAM ahora
simplifica aún más la gestión de permisos entre los componentes _serverless_ con los Conectores de AWS SAM.
Los Conectores SAM fueron desarrollados para abordar los siguientes desafíos de los clientes:

* No todos son expertos en IAM
    * Actualmente, para poder crear políticas de IAM con éxito, los clientes deben comprender las acciones y alcances de
      IAM de todos los servicios de AWS que utilizan.
* Generar políticas bien definidas es tedioso.
    * Crear políticas bien delimitadas requiere múltiples iteraciones de prueba y error, lo que lleva a los
      desarrolladores a crear unas demasiado permisivas.
* Mantenimiento de la aplicación requiere un constante cuidado de las políticas de IAM.
    * Mantenimiento continuo y desarrollo común a aplicaciones sin servidor requiere actualizaciones continuas a las
      políticas IAM, lo que ralentiza el desarrollo.

Los objetivos de los SAM Connectors son:

* Potenciar al desarrollador para que se enfoque en modelar las relaciones de recursos, no en los permisos de acceso.
* Reducir la experiencia de AWS necesaria para construir con éxito aplicaciones serverless seguras y listas para
  producción.
* Aumenta la velocidad de iteración al desarrollar aplicaciones sin servidor
* Trabajar con los controles de gobierno operativos existentes para reducir la fricción de llevar aplicaciones sin
  servidor a producción.

Los SAM Conectores son compatibles con AWS Step Functions, Amazon DynamoDB, AWS Lambda, Amazon SQS, Amazon SNS,
Amazon API Gateway, Amazon EventBridge y Amazon S3, con más recursos previstos en el futuro.
Si está interesado en la lista completa de servicios compatibles con SAM Connectors,
visite [AWS SAM Connector Reference](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/reference-sam-connector.html).

Las plantillas de políticas de AWS SAM son una característica existente que ayuda a los constructores a implementar
aplicaciones serverless con políticas IAM mínimamente detalladas. Dado que hay un número finito de plantillas, son
ideales cuando existe una plantilla para los servicios que estás utilizando. Los conectores son los mejores para
aquellos que están comenzando y quieren centrarse en modelar el flujo de datos y eventos dentro de sus aplicaciones.
Los conectores tomarán el modelo de relación deseado y crearán los permisos para que la relación exista y funcione
según lo previsto.

### ¿Cómo se definen los AWS SAM Connectors?

Los conectores son un tipo de recurso abstracto del Modelo de Aplicación Serverless de AWS (AWS SAM),
identificado como `AWS::Serverless::Connector`, que proporciona permisos simples y bien delimitados entre los recursos
de tu aplicación serverless. Utiliza el atributo de recurso Connectors incrustándolo dentro de un recurso fuente.
Luego, define tu recurso de destino y describe cómo deben fluir los datos o eventos entre esos recursos.
Luego, AWS SAM compone las políticas de acceso necesarias para facilitar las interacciones requeridas.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
...
Resources:
  <source-resource-logical-id>:
    Type: <resource-type>
    ...
    Connectors:
      <connector-name>:
        Properties:
          Destination:
            <properties-that-identify-destination-resource>
          Permissions:
            <permission-types-to-provision>
  ...
```

Vamos a modificar nuestra aplicación para utilizar AWS SAM Connectors y mantener las mejores prácticas de seguridad.

Para cada una de las tres funciones Lambda, eliminar el atributo `Policies` (bajo el atributo `Properties`) y
agregar un atributo `Connectors` como se muestra en las líneas resaltadas a continuación. En cada uno de los conectores,
señalar que el destino es la tabla DynamoDB `SampleTable`.

```yaml
Resources:
  # Each Lambda function is defined by properties:
  # https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction

  # This is a Lambda function config associated with the source code: get-all-items.js
  getAllItemsFunction:
    Type: AWS::Serverless::Function
    Connectors:
      GetItemsConnector:
        Properties:
          Destination:
            Id: SampleTable
          Permissions:
            - Read
    Properties:
      Handler: src/handlers/get-all-items.getAllItemsHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: A simple example includes a HTTP get method to get all items from a DynamoDB table.
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /
            Method: GET
  # Each Lambda function is defined by properties:
  # https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction

  # This is a Lambda function config associated with the source code: get-by-id.js
  getByIdFunction:
    Type: AWS::Serverless::Function
    Connectors:
      GetItemByIdConnector:
        Properties:
          Destination:
            Id: SampleTable
          Permissions:
            - Read
    Properties:
      Handler: src/handlers/get-by-id.getByIdHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: A simple example includes a HTTP get method to get one item by id from a DynamoDB table.
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /{id}
            Method: GET
  # Each Lambda function is defined by properties:
  # https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction

  # This is a Lambda function config associated with the source code: put-item.js
  putItemFunction:
    Type: AWS::Serverless::Function
    Connectors:
      PutItemConnector:
        Properties:
          Destination:
            Id: SampleTable
          Permissions:
            - Write
    Properties:
      Handler: src/handlers/put-item.putItemHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: A simple example includes a HTTP post method to add one item to a DynamoDB table.
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /
            Method: POST
```

Despleguemos la aplicación con estos cambios.

```shell
cd ~/sam-app
sam build && sam deploy --no-confirm-changeset
```

Prueba el API `getAllItems` y deberías ver la misma salida familiar.

```shell
curl -s $ENDPOINT | jq
```

¡Felicidades! Eso concluye nuestro vistazo a los Conectores de AWS SAM y al módulo de Permisos de AWS SAM.

