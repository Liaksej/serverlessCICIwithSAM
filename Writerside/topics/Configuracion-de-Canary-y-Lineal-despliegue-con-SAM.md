# Configuración de despliegue Canary y Lineal

El cambio de tráfico con aliases de Lambda Function está directamente integrado en AWS SAM. Si deseas utilizar despliegues 
**all-at-once**, **canary** o **lineales** con tus Lambda Functions, puedes incorporarlo directamente en tus plantillas de AWS SAM. 
Puede hacerlo en la sección de preferencias de despliegue de la plantilla. AWS CodeDeploy utiliza la sección de 
preferencias de despliegue para gestionar la implementación de funciones como parte de la actualización del stack de 
AWS CloudFormation. SAM tiene varias preferencias de despliegue preconfiguradas que se puede utilizar para implementar el código. 
Consulta la siguiente tabla para ver ejemplos. 

| Tipo de deployment            | Descripción                                                                                                     |
|-------------------------------|-----------------------------------------------------------------------------------------------------------------|
| Canary10Percent30Minutes      | Desplaza el 10% del tráfico en el primer incremento. El 90 por ciento restante se despliega 30 minutos después. |
| Canary10Percent5Minutes       | Desplaza el 10% del tráfico en el primer incremento. El 90 por ciento restante se despliega 5 minutos después.  |
| Canary10Percent10Minutes      | Desplaza el 10% del tráfico en el primer incremento. El 90 por ciento restante se despliega 10 minutos después. |
| Canary10Percent15Minutes      | Desplaza el 10% del tráfico en el primer incremento. El 90 por ciento restante se despliega 15 minutos después. |
| Linear10PercentEvery10Minutes | Desplaza el 10% del tráfico cada 10 minutos hasta que todo el tráfico esté desplazado.                          |
| Linear10PercentEvery1Minute   | Desplaza el 10% del tráfico cada minuto hasta que todo el tráfico esté desplazado.                              |
| Linear10PercentEvery2Minutes  | Desplaza el 10% del tráfico cada 2 minutos hasta que todo el tráfico esté desplazado.                           |
| Linear10PercentEvery3Minutes  | Desplaza el 10% del tráfico cada 3 minutos hasta que todo el tráfico esté desplazado.                           |
| AllAtOnce                     | Desplaza todo el tráfico a las funciones Lambda actualizadas de una sola vez.                                   |


Un despliegue `Canary` es una técnica que reduce el riesgo de desplegar una nueva versión de una aplicación desplegando lentamente 
los cambios a un pequeño subgrupo de usuarios antes de desplegarla a toda la base de clientes.

La estrategia `Linear` significa que el tráfico se desplaza en lotes cada X minutos. Por ejemplo, 
`Linear10PercentEvery10Minutes` desplazará un 10% adicional de tráfico cada 10 minutos. El despliegue completo 
tomará aproximadamente 100 minutos (más probablemente cerca de 90, ya que el primer lote se desplaza cuando comienza 
el despliegue).

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/canary-deployments.png)

## Como funciona Canary

> **Nota**  
[Despliegue gradual con Canary](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/automating-updates-to-serverless-apps.html) 
> es una característica nativa de SAM y no requiere un proceso CI/CD. Este módulo muestra capturas de pantalla de la canalización CI/CD 
> creado en el módulo CodePipeline. Si no ha creado un CodePipeline todavía puede trabajar a través de este módulo. 
> Sepa que aún puede ver el estado del despliegue en AWS CodeDeploy, pero no tendrá una canalización que inspeccionar.

Antes de sumergirnos en la implementación, primero entendamos cómo funcionan los despliegues Canary:

Los conceptos de despliegues _blue/green_ y _canary_ llevan un tiempo entre nosotros y se han establecido como buenas 
prácticas para reducir el riesgo de despliegues de software. En las aplicaciones tradicionales, actualizas lentamente 
y de forma incremental los servidores en tu flota mientras verificas simultáneamente la salud de la aplicación. 
Sin embargo, hay cierta falta de coincidencia al mapear estos conceptos a un mundo _serverless_. 
No puedes desplegar incrementalmente tu software a través de una flota de servidores cuando no hay servidores.

Por fortuna, existen varios servicios y funcionalidades involucrados para hacer posible esto al operar cargas 
de trabajo _serverless_.

### Versiones y aliases de Lambda

AWS Lambda te permite publicar varias versiones de la misma función. Cada versión tiene su propio código y dependencias 
asociadas, así como su propia configuración de función (como asignación de memoria, tiempo de espera y variables de 
entorno). Puedes hacer referencia a una versión específica utilizando un alias de Lambda. Un alias no es más que 
un nombre que puede apuntar a una versión dada de una función de Lambda.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/lambda-versions-aliases.png)

### Redirección de tráfico con alias de Lambda

Con la introducción de tráfico de alias, ahora es posible implementar fácilmente despliegues _Canary_ de funciones 
Lambda. Al actualizar pesos adicionales de versión en un alias, el tráfico de invocación se dirige a las nuevas versiones 
de las funciones basado en el peso especificado. Se pueden analizar métricas detalladas de CloudWatch para el alias 
y la versión durante el despliegue, o realizar otros controles de salud, para asegurarse de que la nueva versión 
es saludable antes de proceder.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/traffic-shifting.png)

### Reenvío de tráfico con SAM y AWS CodeDeploy

AWS CodeDeploy proporciona una implementación llave en mano intuitiva de esta funcionalidad integrada directamente 
en AWS SAM. Los despliegues con cambio de tráfico se pueden declarar en una plantilla SAM y CodeDeploy gestiona 
la implementación de la función como parte de la actualización del stack de CloudFormation. Si algo sale mal, 
las alarmas de CloudWatch se pueden configurar para desencadenar un rollback del stack.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/traffic-shifting-codedeploy.png)

## Actualizar plantilla SAM

Abre la plantilla SAM (`sam-app/template.yaml`) en tu proyecto y añade los bloques `AutoPublishAlias` y 
`DeploymentPreference` en la sección de propiedades de la función `HelloWorldFunction`.

`~/sam-app/template.yaml`

```yaml
Runtime: nodejs20.x
AutoPublishAlias: live
DeploymentPreference:
  Type: Canary10Percent5Minutes
Architectures:
  - x86_64
```

### Validar la plantilla SAM

Ejecuta el siguiente comando en tu terminal:

```shell
cd ~/sam-app
sam validate --lint
```

Si la plantilla es correcta, verás `template.yaml es una plantilla SAM válida`. Si ves un error, entonces es probable 
que tengas un problema de indentación en el archivo YAML. Verifica nuevamente y asegúrate de que coincida con 
la captura de pantalla mostrada arriba.

La herramienta de código abierto AWS CloudFormation Linter (cfn-lint) valida plantillas AWS SAM con la CLI AWS SAM 
ejecutando sam `validate` con la opción `--lint`. cfn-lint realiza una validación detallada en las plantillas de 
AWS CloudFormation utilizando reglas guiadas por la especificación de recursos de AWS CloudFormation y reglas 
personalizadas definidas en un archivo de configuración para validar en contra de ellas.

### Aplicar los cambios

En la terminal, ejecuta los siguientes comandos desde el directorio raíz de tu proyecto `sam-app`.

```shell
git add .
git commit -m "Add Canary deployment configuration to SAM"
git push
```

## Despliegue gradualmente la actualización

Nuestro pipeline está configurado para implementar código nuevo utilizando la estrategia de _Canary_ al 10 por ciento 
durante 5 minutos. Para ver esto en acción, necesitamos actualizar nuestra aplicación.

### Realizamos un cambio de código

AWS SAM no desplegará nada cuando su código no haya cambiado. Dado que nuestro canal está utilizando AWS SAM como 
la herramienta de despliegue, necesitamos realizar algunos cambios en la aplicación.

Cambia el mensaje en el código de respuesta de tu función Lambda `I'm using canary deployments`. Recuerda actualizar 
las pruebas unitarias.

`~/sam-app/hello-world/app.js`

```shell
response = {
  statusCode: 200,
  body: JSON.stringify({
    message: "I'm using canary deployments",
  }),
}
```

`~/sam-app/hello-world/tests/unit/test-handler.js`

```shell
describe("Tests index", function () {
  it("verifies successful response", async () => {
    const result = await app.lambdaHandler(event, context)

    expect(result).to.be.an("object")
    expect(result.statusCode).to.equal(200)
    expect(result.body).to.be.an("string")

    let response = JSON.parse(result.body)

    expect(response).to.be.an("object")
    expect(response.message).to.be.equal("I'm using canary deployments")
  })
})
```

### Haz _push_ del código

```shell
git add .
git commit -m "Changed return message"
git push
```

### Observa el despiegue _Canary_

Tomará unos minutos para que tu pipeline llegue a la etapa `DeployTest` que iniciará la implementación _Canary_ 
de la etapa que tu llamó `sam-app-dev`.

Comencemos un observador que imprime el mensaje de su API cada segundo y te ayuda a notar cuándo comienzan y terminan 
los cambios de tráfico. Consulta el punto final de la API de desarrollo cada segundo con `curl` e imprima el valor de 
retorno en la pantalla. Este comando también añade la salida al archivo `outputs.txt` que puedes revisar más tarde. 
Puede ejecutar este comando desde cualquier directorio.

```shell
watch -n 1 "curl -s $DEV_ENDPOINT | jq '.message' 2>&1 | tee -a outputs.txt"
```

Deberías ver `Hello my friend` en la terminal.

Si ha completado el [módulo CI/CD con CodePipeline](), dirige tu atención a la consola de CodePipeline. Espera a que tu 
pipeline llegue a la etapa DeployTest. Una vez que vea que la etapa se vuelve azul con el estado En progreso, navega a la consola CodeDeploy. 
Si no desplegó un pipeline CI/CD, mira la consola CodeDeploy.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-canary-codedeploy-00.png)

En la consola de CodeDeploy, haz clic en Deployments. Deberías ver tu despliegue _En progreso_. 
Si no ves un despliegue, haz clic en el ícono de actualización. ¡Esto puede tardar unos minutos en aparecer! 
Haz clic en el Id. de Despliegue para ver los detalles.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-canary-codedeploy-0.png)

El estado de despliegue muestra que el 10% del tráfico se ha desplazado a la nueva versión (el _Canary_). 
CodeDeploy retendrá el porcentaje restante hasta que haya transcurrido el intervalo de tiempo especificado. 
En este caso, especificamos que el intervalo sea de 5 minutos.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-canary-codedeploy-1.png)

Cuando te encuentras en esta etapa, echa un vistazo a tu terminal donde iniciaste el comando `watch`. 
Verás el mensaje aparecer ocasionalmente como `I'm using canary deployments`.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/code-pipeline-canary.gif)

Después de cinco minutos, CodeDeploy transferirá el tráfico restante a la nueva versión y la implementación estará completada.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-canary-codedeploy-2.png)

En la terminal donde ejecutaste el comando watch, presiona `Ctrl-C` para detenerlo. Cuenta el número de mensajes para 
cada cadena de mensajes para validar que el tráfico se haya desplazado gradualmente. Por supuesto, la proporción de 
mensajes de retorno nuevos con respecto a los antiguos depende de cuándo comenzaste y detuviste el comando watch.

```shell
sort outputs.txt  | uniq -c
```

```
228 "hello my friend"
84 "I'm using canary deployments"
```

Puedes ver también la secuencia de valores de retorno al abrir `outputs.txt`.

```
cat outputs.txt
```

## Monitorear la salud del despliegue _Canary_

El despliegue _Canary_ es considerablemente más exitosa si el código está siendo monitoreado durante la implementación. 
Puedes configurar CodeDeploy para retroceder automáticamente la implementación si una métrica especificada de CloudWatch 
ha superado un umbral. Métricas comunes para monitorear son los errores de Invocación de Lambda o la Duración de 
la Invocación (latencia), por ejemplo.

### Definimos una Alarma de CloudWatch

Agrega la siguiente definición de **alarma** al archivo `template.yaml` en la sección de Recursos después de 
la definición de `HelloWorldFunction`.

Este bloque define una _Alarma de CloudWatch_. La alarma se activa cuando la función Lambda arroja errores. 
El umbral de la alarma se alcanza cuando hay uno o más errores en un minuto dado, durante dos minutos consecutivos.

```yaml
CanaryErrorsAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmDescription: Lambda function canary errors
    ComparisonOperator: GreaterThanThreshold
    EvaluationPeriods: 2
    MetricName: Errors
    Namespace: AWS/Lambda
    Period: 60
    Statistic: Sum
    Threshold: 0
    Dimensions:
      - Name: Resource
        Value: !Sub "${HelloWorldFunction}:live"
      - Name: FunctionName
        Value: !Ref HelloWorldFunction
      - Name: ExecutedVersion
        Value: !GetAtt HelloWorldFunction.Version.Version
```

### Habilitamos _Canary_ y alarma para producción

Las alarmas y el _Canary_ son excelentes para nuestra implementación en producción. Puede que no quieras o necesitas 
usar despliegues _Canary_ para entornos no productivos. Utilizar una estrategia `AllAtOnce` para nuestra etapa de 
desarrollo hará que las implementaciones sean más rápidas. Configuremos nuestra aplicación _serverless_ para utilizar 
un despliegue _Canary_ y la nueva alarma de CloudWatch solo para la etapa `sam-app-prod` utilizando una `Condición` 
de CloudFormation.

Primero, creamos una declaración de Condición de `IsProduction` después de la sección de `Globals` cerca 
del inicio de `template.yaml`.

A continuación, cambie la preferencia de implementación para usar esta nueva condición de `IsProduction`.

```yaml
DeploymentPreference:
  Type: !If [IsProduction, "Canary10Percent5Minutes", "AllAtOnce"]
  Alarms: !If [IsProduction, [!Ref CanaryErrorsAlarm], []]
```

Tu archivo `template.yaml` debería estar así:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
Description: Sample SAM Template for sam-app

Globals:
  Function:
    Timeout: 3

Conditions:
  IsProduction: !Equals [!Ref "AWS::StackName", "sam-app-prod"]

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello-world/
      Handler: app.lambdaHandler
      Runtime: nodejs16.x
      AutoPublishAlias: live
      DeploymentPreference:
        Type: !If [IsProduction, "Canary10Percent5Minutes", "AllAtOnce"]
        Alarms: !If [IsProduction, [!Ref CanaryErrorsAlarm], []]
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get

  CanaryErrorsAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: Lambda function canary errors
      ComparisonOperator: GreaterThanThreshold
      EvaluationPeriods: 2
      MetricName: Errors
      Namespace: AWS/Lambda
      Period: 60
      Statistic: Sum
      Threshold: 0
      Dimensions:
        - Name: Resource
          Value: !Sub "${HelloWorldFunction}:live"
        - Name: FunctionName
          Value: !Ref HelloWorldFunction
        - Name: ExecutedVersion
          Value: !GetAtt HelloWorldFunction.Version.Version

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

## Introducimos un error

El monitoreo de la salud de tu _Canary_ permite a CodeDeploy tomar la decisión de si se necesita hacer un rollback. 
Si nuestra Alarma CloudWatch llega al estado de _ALARM_, CodeDeploy realizará automáticamente el rollback del despliegue.

### Introducimos un error a propósito

Vamos a romper la función Lambda a propósito para que se active la alarma de errores de `CanaryErrorsAlarm` durante 
el despliegue. Actualiza el código de la Lambda para lanzar un error en cada invocación. Asegúrate de actualizar 
la prueba unitaria, de lo contrario la compilación fallará.

`~/sam-app/hello-world/app.js`

```javascript
let response

exports.lambdaHandler = async (event, context) => {
  throw new Error("This will cause a deployment rollback")
  // try {
  //     response = {
  //         "statusCode": 200,
  //         "body": JSON.stringify({
  //             message: "I'm using canary deployments",
  //         })
  //     }
  // } catch (err) {
  //     console.log(err);
  //     return err;
  // }

  // return response
}
```

```javascript
// 'use strict';

// const app = require('../../app.js');
// const chai = require('chai');
// const expect = chai.expect;
// var event, context;

// describe('Tests index', function () {
//     it('verifies successful response', async () => {
//         const result = await app.lambdaHandler(event, context)

//         expect(result).to.be.an('object');
//         expect(result.statusCode).to.equal(200);
//         expect(result.body).to.be.an('string');

//         let response = JSON.parse(result.body);

//         expect(response).to.be.an('object');
//         expect(response.message).to.be.equal("hello my friend with canaries");
//     });
// });
```

### Hacemos push de los cambios

En la terminal, ejecuta los siguientes comandos desde el directorio raíz de tu proyecto `sam-app`.

```shell
cd ~/sam-app
git add .
git commit -m "Breaking the lambda function on purpose"
git push
```

### Generamos el tráfico

Una vez que hayas enviado el código, deberás generar tráfico en el punto final de la API Gateway de producción. 
Si no generas tráfico para tu función Lambda, la alarma de CloudWatch no se activará.

##### Si no has exportado el *PROD_ENDPOINT*, ejecuta el siguiente comando. {collapsible="true"}
```shell
export PROD_ENDPOINT=$(aws cloudformation describe-stacks --stack-name sam-app-prod | jq -r '.Stacks[].Outputs[].OutputValue | select(startswith("https://"))')
echo "$PROD_ENDPOINT"
```

Inicia un comando `watch` que consulta este punto final dos veces por segundo.

```shell
watch -n 0.5 "curl -s $PROD_ENDPOINT"
```

## Monitorizando de rollback

_Si has creado un pipeline CI/CD_, navega a tu pipeline en la consola de CodePipeline y vigila tu progresión. Verás que la etapa `DeployTest` se despliega rápidamente ya que utiliza la estrategia `AllAtOnce`. 
A pesar de que nuestro código está roto, gracias a nuestro error codificado, se despliega automáticamente en la etapa `DeployTest`.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/canary-deployment-dev-success.png)

Una vez que tu pipeline avanza a la etapa `DeployProd`, las cosas se vuelven más interesantes. En la ventana 
del terminal donde estás ejecutando el comando `watch`, notarás que el mensaje cambia rápidamente de 
`'I'm using canary deployments'` a `Internal server error`.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/code-pipeline-errors.gif)

Después de unos minutos, verás que CodeDeploy marca este despliegue como fallido y vuelve a la versión anterior. 
Los mensajes de error de servidor interno desaparecerán cuando todo el tráfico vuelva a la versión anterior.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/canary-deployment-prod-rollback.png)

Navegate a la página [AWS CodeDeploy Console](https://console.aws.amazon.com/codedeploy/home). 
Mira la implementación, que puede estar en curso o detenida, dependiendo de si se ha completado la reversión. 
Haz clic en el _Id_ del despliegue para ver sus detalles.

Verás que CodeDeploy detectó que `CanaryErrorsAlarm` se ha activado y detuvo el despliegue.

![image](https://static.us-east-1.prod.workshops.aws/public/1d9be5f3-006d-47cd-bd23-bdc7436c4fb0/static/canaries/screenshot-codedeploy-rollback.png)


