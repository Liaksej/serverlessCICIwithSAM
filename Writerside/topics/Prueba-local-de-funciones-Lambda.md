# Prueba local de funciones Lambda

Ahora que tienes una aplicación SAM. Aprenderás cómo ejecutarla y probarla localmente utilizando la CLI de AWS SAM. Esto es importante porque es parte del flujo de trabajo diario de desarrollo. Te ayuda a verificar si la aplicación se está comportando como se espera, depurar lo que está mal y solucionar cualquier problema antes de enviar tus cambios a un repositorio central.

![image_3.4.1.png](image_3.4.1.png)

### Install dependencies

Before we run the application locally, it's a common practice to install third-party libraries or dependencies that 
your application might be using. These dependencies are defined in a file that varies depending on the runtime, for 
example package.json for Node.js projects, requirements.txt for Python, and pom.xml for Java.

```shell
cd ~/environment/sam-app/hello-world
npm install
```

Resultado de ejemplo:

```
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@~2.3.1 (node_modules/chokidar/node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.3.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})

added 100 packages from 72 contributors and audited 101 packages in 3.879s

18 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

### Ejecutar usando SAM CLI

Hay dos formas de ejecutar una aplicación sin servidor de forma local:

Al invocar una función Lambda individual ([SAM Local Invoke reference](https://docs.aws.amazon.com/en_pv/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html))

```shell
cd ~/environment/sam-app
sam local invoke --event events/event.json
```

Al ejecutar un servidor HTTP local que simule API Gateway

```shell
cd ~/environment/sam-app
sam local start-api --port 8080
```

En este módulo, ejecutaremos un servidor HTTP local que simula API Gateway.

### Probar tu punto de acceso

Una vez que tu servidor local esté en ejecución, podemos enviar solicitudes HTTP para probarlo. Elije una de las siguientes opciones:

> **Nota**
> La primera solicitud llevará varios segundos ya que SAM descarga una imagen Docker. Sigue leyendo para aprender por qué SAM hace esto.

Sin detener el proceso en ejecución, abre un nuevo terminal.

![image3.4.2.png](image3.4.2.png)

Prueba tu endpoint ejecutando un comando CURL que provoque una solicitud GET HTTP.

```shell
curl http://localhost:8080/hello
```

> **Nota**
> Observa cómo SAM está descargando una imagen de contenedor Docker. De esta forma, SAM puede simular el runtime de Lambda de forma local 
> y ejecutar tu función. La primera invocación puede tardar unos segundos debido al comando `docker pull`, pero las invocaciones posteriores 
> serán más rápidas. SAM descargará el contenedor apropiado basado en el tiempo de ejecución que hayas configurado en `template.yaml`.
