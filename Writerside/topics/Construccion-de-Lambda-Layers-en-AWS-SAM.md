# Construcción de Lambda Layers en AWS SAM

Para crear un Layer personalizado, declárela en su archivo de plantilla AWS SAM e incluya 
una sección de atributo de recurso `Metadata` con una entrada `BuildMethod`. Los valores válidos para `BuildMethod` son 
identificadores para un [tiempo de ejecución de AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html), 
o `makefile`. Incluir una entrada `BuildArchitecture` para especificar las arquitecturas de conjuntos de instrucciones 
que admite su Layer. Los valores válidos para `BuildArchitecture` son [Lambda instruction set architectures](https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html).

Si especificas `makefile`, proporciona el makefile personalizado, donde declares un objetivo de construcción 
de la forma `build-layer-logical-id` que contenga los comandos de construcción para tu Layer. Tu `makefile` es responsable 
de compilar el Layer si es necesario, y de copiar los artefactos de construcción en la ubicación adecuada requerida para 
los pasos posteriores en tu flujo de trabajo. La ubicación del makefile se especifica mediante la propiedad `ContentUri` 
del recurso de la capa, y debe tener el nombre de `Makefile`.

> **Nota**
> Cuando creas una capa personalizada, AWS Lambda depende de variables de entorno para encontrar el código de tu Layer. 
> Los tiempos de ejecución de Lambda incluyen rutas en el directorio `/opt donde se copia el código del Layer. 
> La estructura de carpetas de artefactos de compilación de su proyecto debe coincidir con la estructura de carpetas 
> esperada del tiempo de ejecución para que se pueda encontrar su código de capa personalizado.
>
> Para NodeJS, puedes colocar tu código en el subdirectorio `nodejs/node_modules/`.
>
> Para obtener más información, consulte [Including library dependencies in a layer](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html#configuration-layers-path) en la Guía del desarrollador de AWS Lambda.

El siguiente es un ejemplo sección de atributos de recursos de metadatos.

```yaml
      MyLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      Description: Layer for S3 Orphans
      ContentUri: .
      CompatibleRuntimes:
        - nodejs20.x
      RetentionPolicy: Retain
    Metadata:
      BuildMethod: makefile
```

> **Nota**
> Si no incluye la sección de atributo de recurso `Metadata`, AWS SAM no construye el Layer. En su lugar, copia 
> los artefactos de compilación desde la ubicación especificada en la propiedad `CodeUri` del recurso de capa. 
> 
> Para obtener más información, consulte la propiedad [ContentUri](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-layerversion.html#sam-layerversion-contenturi) 
> propiedad del tipo de recurso `AWS::Serverless::LayerVersion`.

Cuando incluyes la sección de atributos del recurso `Metadata`, puedes utilizar el comando `sam build` para construir 
la capa, ya sea como un objeto independiente o como una dependencia de una función AWS Lambda.

* **Como un objeto independiente.** Es posible que desee construir solo el objeto de Layer, por ejemplo, cuando esté 
probando localmente un cambio de código en el Layer y no necesite construir toda su aplicación. Para construir el Layer
de forma independiente, especifique el recurso de Layer con el comando `sam build layer-logical-id`.
* Como una dependencia de una función Lambda. Cuando incluyes el ID lógico de un Layer en la propiedad `Layers` de una 
función Lambda en el mismo archivo de plantilla AWS SAM, el Layer es una dependencia de esa función Lambda. 
Cuando esa capa también incluye una sección de atributos de recurso `Metadata` con una entrada `BuildMethod`, 
construyes el Layer ya sea construyendo toda la aplicación con el comando `sam build` o especificando el recurso 
de función con el comando `sam build function-logical-id`.

### Ejemplo

```yaml
Resources:
  MyLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      ContentUri: my_layer
      CompatibleRuntimes:
        - nodejs20.x
    Metadata:
      BuildMethod: makefile
```

El siguiente archivo `makefile` contiene el objetivo de construcción y los comandos que se ejecutarán. 
Tenga en cuenta que la propiedad `ContentUri` está configurada en `my_layer`, por lo que el makefile debe ubicarse en 
la raíz del subdirectorio `my_layer`, y el nombre de archivo debe ser `Makefile`. También tenga en cuenta que 
los artefactos de construcción se copian en el subdirectorio `nodejs/` para que AWS Lambda pueda encontrar el código 
de la capa.

```makefile
build-S3OrphansLayer:
	mkdir -p "$(ARTIFACTS_DIR)/nodejs"
	cp package.json package-lock.json "$(ARTIFACTS_DIR)/nodejs/"
	npm install --production --prefix "$(ARTIFACTS_DIR)/nodejs/"
	rm "$(ARTIFACTS_DIR)/nodejs/package.json" "$(ARTIFACTS_DIR)/nodejs/package-lock.json" # to avoid rebuilding when changes aren't related to dependencies
```

### Ejemplo sam build commands

Los siguientes comandos `sam build` construyen capas que incluyen las secciones de atributos de recursos `Metadata`.

```shell
# Build the 'layer-logical-id' resource independently
sam build layer-logical-id
            
# Build the 'function-logical-id' resource and layers that this function depends on
sam build function-logical-id

# Build the entire application, including the layers that any function depends on
sam build
```

