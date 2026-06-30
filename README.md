# tienda-perritos

Aplicación web CRUD para gestión de productos de una tienda de mascotas, desplegada en AWS EKS con pipeline CI/CD automatizado mediante GitHub Actions.

## Arquitectura

```
Usuario → Load Balancer → Nginx (Frontend) → Node.js API (Backend) → MySQL (DB)
```

La aplicación está compuesta por tres servicios containerizados que corren en Kubernetes:

- **Frontend**: HTML + JavaScript servido por Nginx, con proxy reverso hacia el backend
- **Backend**: API REST en Node.js/Express con 5 endpoints CRUD
- **Base de datos**: MySQL 8 con datos iniciales cargados automáticamente

---

## Requisitos previos

- Cuenta en AWS con un clúster EKS activo
- AWS CLI instalado y configurado
- kubectl instalado y conectado al clúster
- Docker instalado (solo para builds locales)
- Repositorio en GitHub con acceso a GitHub Actions

---

## Estructura del proyecto

```
tienda-perritos/
├── frontend/
│   ├── index.html          # Interfaz de usuario
│   ├── app.js              # Lógica del frontend (fetch a la API)
│   ├── default.conf        # Configuración de Nginx con proxy reverso
│   └── Dockerfile
├── backend/
│   ├── server.js           # API REST en Express
│   ├── package.json
│   └── Dockerfile
├── db/
│   ├── init.sql            # Script SQL con tablas y datos iniciales
│   └── Dockerfile
├── k8s/
│   ├── namespace.yaml
│   ├── mysql-secret.yaml
│   ├── mysql-deployment.yaml
│   ├── mysql-service.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── backend-hpa.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   └── frontend-hpa.yaml
└── .github/
    └── workflows/
        └── deploy-eks.yml  # Pipeline CI/CD
```

---

## Despliegue automático (recomendado)

El pipeline se activa automáticamente al hacer push a la rama `deploy`.

### 1. Configurar secrets en GitHub

Ve a **Settings → Secrets and variables → Actions** y agrega:

| Secret | Descripción |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | Access key de AWS |
| `AWS_SECRET_ACCESS_KEY` | Secret key de AWS |
| `AWS_SESSION_TOKEN` | Session token (requerido en AWS Academy) |
| `AWS_REGION` | Región, por ejemplo `us-east-1` |
| `EKS_CLUSTER_NAME` | Nombre del clúster EKS |
| `EKS_NAMESPACE` | Namespace de Kubernetes, por ejemplo `tienda` |

> las credenciales expiran cada ~4 horas. Debes actualizarlas en los secrets antes de cada deploy.

### 2. Crear repositorios en ECR

En AWS → ECR, crea estos tres repositorios privados:

- `tienda-perrito-frontend`
- `tienda-perrito-backend`
- `tienda-perrito-bd`

### 3. Hacer el deploy

```bash
git add .
git commit -m "deploy: descripción del cambio"
git push origin deploy
```

El pipeline realizará automáticamente:
1. Build de las tres imágenes Docker
2. Push a Amazon ECR con tag del commit
3. Aplicación de todos los manifests en Kubernetes
4. Rollout de los deployments con las nuevas imágenes
5. Verificación del estado final de pods y servicios

---

## Acceso a la aplicación

###Port-forward (recomendado para laboratorio)

```bash
# Conectar kubectl al clúster
aws eks update-kubeconfig --region us-east-1 --name Cluster-tienda-perritos

# Exponer el frontend localmente
kubectl port-forward svc/tienda-frontend 8080:80 -n tienda

# Amazon Configure
aws configure
```

Abre el navegador en: **http://localhost:8080**

---

## API REST

El backend expone los siguientes endpoints en el puerto `3001`:

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/productos` | Listar todos los productos |
| GET | `/api/productos/:id` | Obtener un producto por ID |
| POST | `/api/productos` | Crear un nuevo producto |
| PUT | `/api/productos/:id` | Actualizar un producto |
| DELETE | `/api/productos/:id` | Eliminar un producto |
| GET | `/api/health` | Health check del servicio |

Ejemplo de cuerpo para crear un producto:

```json
{
  "nombre": "Collar antipulgas",
  "descripcion": "Collar para perros medianos",
  "precio": 4990,
  "stock": 20
}
```

---

## Verificar el estado del clúster

```bash
# Ver todos los pods
kubectl get pods -n tienda

# Ver servicios y puertos
kubectl get svc -n tienda

# Ver autoscaling
kubectl get hpa -n tienda

# Ver logs del backend
kubectl logs -l app=tienda-backend -n tienda
```

---

## Autoscaling

Se configuraron dos Horizontal Pod Autoscalers:

| Componente | Réplicas mín. | Réplicas máx. | Umbral CPU |
|------------|--------------|--------------|-----------|
| Backend | 2 | 10 | 70% |
| Frontend | 2 | 6 | 60% |

El escalado es automático — Kubernetes agrega o elimina réplicas según la carga.

---

## Actualizar credenciales AWS (AWS Academy)

Cada vez que abres una nueva sesión de laboratorio:

```bash
aws configure set aws_access_key_id TU_ACCESS_KEY
aws configure set aws_secret_access_key TU_SECRET_KEY
aws configure set aws_session_token TU_SESSION_TOKEN
aws configure set default.region us-east-1
```

Y actualiza los mismos valores en los secrets de GitHub antes de hacer el próximo deploy.
