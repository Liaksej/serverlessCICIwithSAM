# Características de configuración al usar TypeScript en proyectos con SAM

Para construir y empaquetar **AWS Lambda Functions** escritas en TypeScript, puedes utilizar la CLI de AWS SAM con 
el **empaquetador** de JavaScript `esbuild`. El empaquetador esbuild es compatible con las funciones Lambda 
que escribas en TypeScript.

Para construir una función Lambda de TypeScript con esbuild, agregue un objeto `Metadata` a su recurso 
`AWS:Serverless::Function` y especifique `esbuild` para el `BuildMethod`. Cuando ejecute el comando `sam build`, 
AWS SAM utiliza esbuild para empaquetar el código de su función Lambda.

## Propiedades de Metadata

El objeto `Metadata` admite las siguientes propiedades para esbuild.

### BuildMethod

Especifica el empaquetador para tu aplicación. El único valor admitido es `esbuild`.

### BuildProperties

Especifica las propiedades de compilación para el código de tu Lambda Function.

El objeto `BuildProperties` soporta las siguientes propiedades para esbuild. Todas las propiedades son opcionales. 
Por defecto, AWS SAM utiliza el handler de tu Lambda function como punto de entrada.

#### EntryPoints
Especifica los puntos de entrada para tu aplicación.

#### External
Especifica la lista de paquetes que deben omitirse en la compilación. Para obtener más información, consulte [External](https://esbuild.github.io/api/#external) en el sitio web de esbuild.

#### Format
Especifica el formato de salida de los archivos JavaScript generados en su aplicación. Para obtener más información, consulte [Format](https://esbuild.github.io/api/#format) en el sitio web de esbuild.

#### Loader
Especifica la lista de configuraciones para cargar datos para un tipo de archivo dado.

#### MainFields
Especifica qué campos de `package.json` intentar importar al resolver un paquete. El valor predeterminado es `main,module`.

#### Minify
Especifica si se debe minificar el código de salida empaquetado. El valor predeterminado es `true`.

#### OutExtension
Personalice la extensión de los archivos que genera esbuild. Para obtener más información, consulte [Out extension](https://esbuild.github.io/api/#out-extension) en el sitio web de esbuild.

#### Sourcemap
Especifica si el empaquetador produce un archivo de mapa de origen. El valor predeterminado es `false`.

Cuando se establece en `true`, `NODE_OPTIONS: --enable-source-maps` se añade a las variables de entorno de la Lambda Function, 
y se genera un mapa fuente que se incluye en la función.

Como alternativa, cuando se incluye `NODE_OPTIONS: --enable-source-maps` en las variables de entorno de la función, 
`Sourcemap` se establece automáticamente en `true`.

Cuando haya conflictos, `Sourcemap: false` tiene prioridad sobre `NODE_OPTIONS: --enable-source-maps`.

> **Nota**
> Por defecto, Lambda cifra todas las variables de entorno en reposo con AWS Key Management Service (AWS KMS). 
> Cuando se utilizan mapas de origen, para que el despliegue tenga éxito, el rol de ejecución de tu función debe tener permiso para 
> realizar la acción `kms:Encrypt`.

### SourcesContent

Especifica si incluir tu código fuente en tu archivo de mapa fuente. Configura esta propiedad cuando `Sourcemap` se establece en `true`.

* Especificar SourcesContent: `true` para incluir todo el código fuente.
* Especifique `SourcesContent: 'false'` para excluir todo el código fuente. Esto da como resultado tamaños de archivos 
de mapas de origen más pequeños, lo cual es útil en producción al reducir los tiempos de inicio. Sin embargo, el código 
fuente no estará disponible en el depurador.

El valor por defecto es `SourcesContent: true`.

#### Target

Especifica la versión objetivo de ECMAScript. El valor predeterminado es `es2020`.

### Un ejemplo de TypeScript Lambda function

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
      Environment:
        Variables:
          NODE_OPTIONS: --enable-source-maps
    Metadata:
      BuildMethod: esbuild
      BuildProperties:
        Format: esm
        Minify: false
        OutExtension:
          - .js=.mjs
        Target: "es2020"
        Sourcemap: true
        EntryPoints: 
          - app.ts
        External:
          - "<package-to-exclude>"
```

