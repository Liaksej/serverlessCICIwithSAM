# Eliminar la aplicación desplegada

Para eliminar la aplicación, ejecuta `sam delete`.

```shell
cd ~/sam-app
sam delete
```
Para eliminar un stack específico, por ejempo un stack de pipeline CI/CD, ejecuta:

```shell
cd ~/sam-app
sam delete --stack-name <name-of-the-stack> --region eu-central-1
```

Responde a todas las preguntas:

```
  Are you sure you want to delete the stack sam-app in the region us-west-2 ? [y/N]: y
  Are you sure you want to delete the folder sam-app in S3 which contains the artifacts? [y/N]: y
```