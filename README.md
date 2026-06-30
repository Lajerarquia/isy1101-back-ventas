# Backend Ventas — Innovatech Chile

API REST construida con Java 21 y Spring Boot que expone los endpoints CRUD para el módulo de ventas. Persiste los datos en una base de datos MySQL y se despliega como contenedor en AWS ECS Fargate.

## Arquitectura

```
Internet → ALB (alb-innovatech) → /ventas/* → tg-ventas (puerto 8081) → ECS svc-ventas
```

El servicio corre dentro de la VPC de Innovatech y solo recibe tráfico desde el Security Group del ALB, no está expuesto directamente a internet.

## Requisitos previos

- Java 21
- Maven 3.9+
- Docker

## Correr localmente con Docker

```bash
docker build -t ventas-backend .
docker run -p 8081:8081 \
  -e DB_ENDPOINT=localhost \
  -e DB_PORT=3306 \
  -e DB_NAME=nombre_bd \
  -e DB_USERNAME=admin \
  -e DB_PASSWORD=tu_password \
  ventas-backend
```

La API queda disponible en `http://localhost:8081/ventas`.

## Variables de entorno

| Nombre | Descripción | Obligatoria |
|--------|-------------|-------------|
| DB_ENDPOINT | Host del servidor MySQL (endpoint RDS) | Sí |
| DB_PORT | Puerto MySQL | Sí |
| DB_NAME | Nombre de la base de datos | Sí |
| DB_USERNAME | Usuario de la base de datos | Sí |
| DB_PASSWORD | Contraseña de la base de datos | Sí |

En ECS estas variables se inyectan desde AWS Secrets Manager (`innovatech/db-credentials`) vía `valueFrom` en la Task Definition.

## Despliegue en ECS

El pipeline se dispara automáticamente en cada push a `main`. Los pasos son:

1. Autenticación en AWS con las credenciales del Learner Lab (secrets del repo)
2. Login a Amazon ECR
3. Build de la imagen Docker desde `Springboot-API-REST/`, tag con el SHA del commit, push a ECR (`innovatech-ventas-backend`)
4. Descarga de la Task Definition actual (`td-ventas-backend`) desde ECS
5. Actualización del campo de imagen con la nueva versión
6. Deploy al servicio `svc-ventas` en el cluster `innovatech-cluster`, esperando estabilización

> **Nota:** las credenciales de AWS del Learner Lab vencen con cada sesión. Actualizar los secrets `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` y `AWS_SESSION_TOKEN` en GitHub antes de cada clase.

## URL de acceso

`http://<ALB_URL_AQUI>/ventas`

<!-- TODO: reemplazar <ALB_URL_AQUI> con la URL real del ALB una vez creado en AWS (paso AWS-9) -->
