# Estructura del proyecto

La estructura general de los archivos y carpetas puede variar de un proyecto a otro. 
A lo largo de este documento, consideraremos varios escenarios. En este ejemplo, analizaremos **la estructura 
más básica** sin archivos adicionales relacionados con CI/CD, sobre los cuales hablaremos más tarde en la sección de CI/CD.

Consideremos la estructura general, avanzando de arriba hacia abajo a través de las carpetas.

![image](image_general_folder_review.png)

### Carpeta `.aws-sam` 

Esta carpeta contiene archivos de configuración y plantillas de implementación SAM. 
Se genera y actualiza automáticamente bajo comando: 

```shell
sam build
```

Consideremos detenidamente su estructura:

#### Carpeta build
Contiene los archivos compilados del proyecto, en particular el archivo `.js` de la función Hola Mundo.

Además, en la carpeta se encuentra `template.yaml`, que es una plantilla de AWS CloudFormation convertida 
de la plantilla de SAM, ubicada en la raíz del proyecto. Es esta plantilla la que despliega los recursos 
en AWS CloudFormation al ejecutar el comando:

```shell
sam deploy
```

### Carpetas .github и events

Estas carpetas no contienen información importante para configurar nuestro proyecto actual SAM en esta etapa.

### Carpeta hello-world

Esta carpeta contiene el código fuente del proyecto: la función de Lambda del controlador y las pruebas unitarias para ella.

Es importante tener en cuenta cuál es la ruta relativa exacta que tiene el handler con respecto a 
la carpeta principal del proyecto. Esta ubicación afecta significativamente la compilación del proyecto. 
Si al compilar localmente manualmente con `sam build` la ubicación no importa en general, teniendo en cuenta que el proyecto 
tiene **esbuil** instalado en la carpeta `node_modules` a nivel de todo el proyecto. 
Sin embargo, al compilar a través de un **CI/CD pipeline**, la misma compilación generará errores, para superarlos, 
es necesario recurrir a instrucciones adicionales en forma de comandos de shell a nivel del contenedor de compilación. 
Un ejemplo de esta configuración se encuentra en el archivo `buildspec_build_package.yml`, que contiene instrucciones 
para compilar el proyecto con AWS CodeBuild:

```shell
version: 0.2
phases:
  build:
    commands:
#     The for loop is necessary for transpiling TypeScript in JavaScript,
#     therefore, Lambdas should only be located in the functions folder
      - for dir in src/functions/*; do
          if [ -d "$dir" ]; then
            cd "$dir";
            if [ ! -f package.json ]; then
              npm init -y;
            fi;
            npm install esbuild;
            cd -;
          fi;
        done
#     Build start here
      - sam build
```

En este ejemplo, las **funciones** se encuentran en el directorio `src/functions/`

El script realiza las siguientes operaciones:

* entra en la carpeta de cada una de las funciones;
* instala `package.json` en esta carpeta, si no está allí.
* instala `esbuild` como dependencia.

Este enfoque permite no copiar el archivo `package.json` durante el desarrollo por todo el proyecto, mantener el código fuente 
limpio y compilar TypeScript a JavaScript de manera correcta.

### Carpeta `node_modules`  

Contiene los módulos instalados de Node.js

### `.eslintignore` y `.eslintrc.js`

Contienen ajustes para [ESLint](https://eslint.org).

### `.gitignore` y `.npmignore`

Estos archivos contienen configuraciones de ignorar para Git y NPM.

### `.prettierrc.js`

Contiene ajustes para [Prettier](https://prettier.io) 

### `.jest.config.ts`

Contiene ajustes para [Jest](https://jestjs.io) - un framework para pruebas unitarias.

### `Makefile`

Contiene configuraciones de compilación para funciones Lambda o Lambda Layers. 

#### Uso de `Makefile` para la función Lambda

El siguiente plantilla de AWS SAM declara una función que utiliza un runtime personalizado para 
una función Lambda escrita en TypeScript, e instruye a `sam build` a ejecutar los comandos para el objetivo de 
compilación `Mylambda`.

```yaml
  Resources:
    MyLambda:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .
      Handler: src/functions/fetch-s3buckets/app.handler
    Metadata: # Manage esbuild properties
      BuildMethod: makefile
```

El siguiente archivo de configuración contiene el objetivo de construcción y los comandos que se ejecutarán. 
Hay que tener en cuenta que la propiedad CodeUri está configurada en `.`, por lo que el archivo de configuración debe estar 
ubicado en el directorio raíz del proyecto (es decir, el mismo directorio que el archivo de plantilla AWS SAM de la aplicación). 
El nombre de archivo debe ser `Makefile`.

```makefile
.PHONY: build-RuntimeDependenciesLayer build-lambda-common

build-lambda-common:
	npm install
	rm -rf dist
	echo "{\"extends\": \"./tsconfig.json\", \"include\": [\"${HANDLER}\"] }" > tsconfig-only-handler.json
	npm run build -- --build tsconfig-only-handler.json
	cp -r dist "$(ARTIFACTS_DIR)/"
```

#### Uso de `Makefile` para Lambda Layer

El siguiente plantilla de AWS SAM declara un Lambda Layer que utiliza una compilación personalizada e instruye 
a `sam build` a ejecutar los comandos para el objetivo de compilación.

```yaml
Resources:
  MyLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      LayerName: !Sub "MyLayer"
      ContentUri: .
      CompatibleRuntimes:
        - nodejs20.x
      RetentionPolicy: Retain
    Metadata:
      BuildMethod: makefile
```

El siguiente `makefile` contiene el objetivo de construcción y los comandos que se ejecutarán. 
Nota que la propiedad `CodeUri` se establece en `.`, por lo que el makefile debe estar ubicado en el directorio raíz del proyecto 
(es decir, el mismo directorio que el archivo de plantilla de AWS SAM de la aplicación). 
El nombre de archivo debe ser `Makefile`.

```makefile
.PHONY: build-S3OrphansLayer

build-S3OrphansLayer:
	mkdir -p "$(ARTIFACTS_DIR)/nodejs"
	cp package.json package-lock.json "$(ARTIFACTS_DIR)/nodejs/"
	npm install --production --prefix "$(ARTIFACTS_DIR)/nodejs/"
	rm "$(ARTIFACTS_DIR)/nodejs/package.json" "$(ARTIFACTS_DIR)/nodejs/package-lock.json" # to avoid rebuilding when changes aren't related to dependencies
```

Así, todas las dependencias comunes del proyecto serán reunidas en una única capa, que será desplegada automáticamente 
por SAM, y las Lambda Functions del proyecto tendrán acceso a ella. Hablaremos en las siguientes secciones sobre cómo 
asignar una capa a una función Lambda.

### `package.json` и `package-lock.json `

Estos archivos contienen dependencias comunes de las funciones del proyecto.

### `README.md`

El archivo contiene una descripción del proyecto.

### `samconfig.toml`

AWS SAM CLI soporta un archivo de configuración a nivel de proyecto que puedes usar para configurar los valores de los 
parámetros de comando de AWS SAM CLI.

Encontrará información detallada sobre la configuración almacenada en [aquí](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-config.html).

### `template.yaml`

Es la **Plantilla SAM**, que contiene configuraciones para todos los recursos del proyecto. Es de vital importancia para el proyecto.
La consideramos en [el capítulo anterior](Aspectos_generales_características_y_ejemplos_de_Plantilla_SAM.md).