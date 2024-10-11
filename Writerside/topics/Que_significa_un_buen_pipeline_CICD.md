# Qué significa un buen pipeline CI/CD?

Con el desarrollo serverless, el término `deployment` puede adquirir un nuevo significado. Al desarrollar aplicaciones 
serverless, ya no despliegas nuevo código de aplicación en servidores, porque no hay servidores.

Utilizando servicios de infraestructura como código, como AWS CloudFormation, AWS Cloud Development Kit (AWS CDK), 
Terraform y el Framework Serverless, los desarrolladores son capaces de crear recursos de AWS de manera ordenada y predecible.

Con AWS Lambda, una implementación puede ser tan sencilla como una llamada a la API para crear una función 
o actualizar el código de la función.

## Automatización del Pipeline de Implementación

Las aplicaciones modernas despliegan actualizaciones con frecuencia para implementar nuevas funcionalidades, 
pero actualizar o cambiar una aplicación en producción a menudo conlleva riesgos y podría introducir errores. 
Es importante considerar tu estrategia de implementación y determinar cómo puede ayudarte a desplegar 
de forma segura y gradual.

### Implementaciones seguras y graduales

Al cargar un fragmento de código en **un repositorio Git**, no quieres esperar a que un humano lo apruebe manualmente 
o hacer que cada fragmento de código pase por diferentes controles de calidad. Tampoco quieres liberar un fragmento 
de código a producción tan pronto lo cargas. Esta documentación muestra cómo automatizar un proceso de despliegue 
de CI/CD seguro y gradual.

### Versiones y aliases de Lambda
Para entender las estrategias de implementación, primero necesitas comprender el concepto de versiones de Lambda y alias de Lambda.

#### Versiones de Lambda

Al crear una función Lambda, solo existe una versión llamada `$LATEST`. Cada vez que publiques una versión, 
Lambda toma una copia instantánea de `$LATEST` para crear la nueva versión. Esta copia no se puede modificar.

![4C4dFG6_qEg9cO2I_NrnKtOK71kkkLSY7.gif](4C4dFG6_qEg9cO2I_NrnKtOK71kkkLSY7.gif)

#### Lambda aliases

Un alias de Lambda es un puntero a una versión específica de una función. Por defecto, un alias apunta a una 
sola versión de Lambda. Cuando se actualiza el alias para que apunte a una versión diferente de la función, 
todo el tráfico de solicitudes entrantes se redirigirá hacia la versión actualizada de la función Lambda.

![2U1qqxVxhgyt0nD0_KXE0-taAyoDoOTa1.gif](2U1qqxVxhgyt0nD0_KXE0-taAyoDoOTa1.gif)

## Estrategias de despliegue

Con Lambda traffic shifting, puedes enviar un pequeño subconjunto de tráfico a tu nueva versión de función mientras 
mantienes la mayoría del tráfico de producción entrante en tu versión antigua y estable. Algunas de las siguientes 
strategias de implementación utilizan el tráfico shifting. El tráfico shifting te ayuda a validar que tu nueva 
versión de Lambda funciona como se espera antes de enviar todo el tráfico de producción a ella.

#### All-at-once

Las implementaciones de una sola vez cambian instantáneamente el tráfico de la función de Lambda original (antigua) 
a la función de Lambda actualizada (nueva), todo al mismo tiempo. Las implementaciones de una sola vez pueden 
ser beneficiosas cuando la velocidad de tus implementaciones es importante. En esta estrategia, la nueva versión 
de tu código se lanza rápidamente y todos tus usuarios pueden acceder a ella de inmediato.

![XoHW_kQiATFSkaTP_Qq2k-YqtjYkHdDAI.gif](XoHW_kQiATFSkaTP_Qq2k-YqtjYkHdDAI.gif)

#### Canary

En una implementación de canario, despliegas la nueva versión de tu código de aplicación y rediriges un pequeño 
porcentaje del tráfico de producción para que apunte a esa nueva versión. Después de validar que esta versión 
es segura y no está causando errores, diriges todo el tráfico a la nueva versión de tu código.

![IxwTO6k4B5VpRaf7_J7YZxn9DxihXXYLq.gif](IxwTO6k4B5VpRaf7_J7YZxn9DxihXXYLq.gif)

#### Linear

Un *despliegue linear* es similar a un despliegue *canary*. En esta estrategia, diriges una pequeña cantidad 
de tráfico a tu nueva versión del código al principio. Después de un período de tiempo especificado, 
incrementas automáticamente la cantidad de tráfico que envías a la nueva versión hasta que estés enviando 
el 100 por ciento del tráfico de producción. 

![h3yvwiEveK-7GGQt_XNEUddKdeu5tek0J.gif](h3yvwiEveK-7GGQt_XNEUddKdeu5tek0J.gif)

### Comparando estrategias de implementación

Para ayudarte a decidir qué estrategia de implementación usar para tu aplicación, deberás considerar el impacto en 
el usuario, la posibilidad de deshacer, los factores del modelo de eventos y la velocidad de implementación de cada opción. 
La siguiente tabla comparativa ilustra estos puntos.

| Deployment    | Impacto en el consumidor                                     | Rollback                                                | Factores del modelo de eventos                    | Velocidad de despliegue |
|---------------|--------------------------------------------------------------|---------------------------------------------------------|---------------------------------------------------|-------------------------|
| All-at-once   | Todo a la vez                                                | Reimplantar la versión anterior                         | Cualquier modelo de eventos con baja concurrencia | Inmediato               |
| Canary/Linear | Cambio de tráfico inicial típico del 1-10%, luego escalonado | Revertir el 100% del tráfico a la implantación anterior | Mejor para cargas de trabajo de alta concurrencia | De minutos a horas      |

## Crear un canal de implementación

Cuando se sube un fragmento de código al control de fuente, no se espera a que un humano lo apruebe manualmente ni
a que cada fragmento de código pase por diferentes controles de calidad. Un pipeline de CI/CD puede ayudar 
a automatizar los pasos necesarios para lanzar la implementación de su software y estandarizar en un conjunto 
central de controles de calidad. 

Aquí tienes un ejemplo de un pipeline de despliegue continuo:

![image-cicd-flow.png](image-cicd-flow.png)

Un pipeline de CI/CD está compuesto principalmente por cuatro pasos: 

* Source
* Build
* Test
* Production

CI/CD puede ser visualizado como un pipeline, donde nuevo código es presentado en un extremo, probado 
a lo largo de una serie de etapas (source, build, test), y luego publicado como código listo para producción.

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/DZ1UoaCWMfRMXFVp_ky8CN063ZfgTWXbv.jpg)

AWS ofrece una herramienta para cada fase del pipeline. Estas herramientas incluyen:

* AWS CodeStar para conectar AWS con repositorio Git
* AWS CodeBuild para etapas Build y Test
* AWS CodeDeploy para etapa Production
* AWS CodePipeline para administrar un despliegue continuo

Para obtener más información sobre CI/CD en AWS, consulte la página [CI/CD on AWS whitepaper](https://docs.aws.amazon.com/whitepapers/latest/cicd_for_5g_networks_on_aws/cicd-on-aws.html).


### Automatización de CI/CD pipeline

![image](https://explore.skillbuilder.aws/files/a/w/aws_prod1_docebosaas_com/1728054000/OkLFRSay1U8Sv7H0Idus2Q/tincan/675621_1654804371_p1g5509l8kdo9ri4lrh1vhpr5f4_zip/assets/MGQJYdFXiGyUhga2_1WZyHTw-qYqrfmAF.jpg)

Esta arquitectura muestra cómo puedes usar AWS CodePipeline con AWS SAM. Con CodePipeline, cada vez que haya 
un cambio de código o un nuevo envío de código, puedes iniciar este canal de procesamiento, donde construyes, 
pruebas y despliegas cambios de código.

## Recursos adicionales

* [Generating starter CI/CD pipelines using AWS SAM Pipelines](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-generating-example-ci-cd.html)
* [CI/CD for Serverless and Containerized Applications](https://www.youtube.com/watch?v=01ewawuL-IY&t=419s)
