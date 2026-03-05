# Pedido-App – Parcial I Kubernetes

Aplicación **Full Stack desplegada en Kubernetes** que permite ingresar valores desde una interfaz web, almacenarlos en una base de datos y visualizarlos en una lista dinámica.

El proyecto implementa buenas prácticas de:

- Contenedorización con Docker
- Persistencia de datos
- Definición de recursos (`requests` / `limits`)
- Escalamiento automático con **Horizontal Pod Autoscaler (HPA)**
- Despliegue mediante **Helm**
- **GitOps con ArgoCD**

## Prerrequisitos

Antes de ejecutar el proyecto es necesario contar con:

- Kubernetes funcionando (Minikube recomendado)
- kubectl configurado
- Helm 3 instalado
- ArgoCD instalado en el cluster
- Acceso al repositorio Git

---

# Arquitectura

La aplicación está compuesta por tres servicios principales.


### Frontend
Interfaz web que permite:

- Ingresar valores
- Visualizar los datos almacenados
- Consumir la API del backend

### Backend API
API REST que:

- Recibe solicitudes del frontend
- Guarda información en la base de datos
- Expone endpoints para insertar y consultar datos

Endpoints principales:

- `POST /api`
- `GET /api`

### Base de Datos

Base de datos **PostgreSQL** encargada de almacenar los valores ingresados.

Se implementa persistencia mediante:

- **PersistentVolume (PV)**
- **PersistentVolumeClaim (PVC)**

Esto asegura que los datos **no se pierdan si el pod se reinicia**.

---

# Flujo de funcionamiento

1. El usuario ingresa un valor en el **frontend**
2. El frontend envía una solicitud **POST al backend**
3. El backend guarda el valor en **PostgreSQL**
4. El frontend consulta los valores mediante **GET**
5. Se actualiza la lista en pantalla

---

# Stack Tecnológico

| Componente | Tecnología |
|------------|------------|
| Frontend | HTML / JavaScript |
| Backend | Node.js |
| Base de datos | PostgreSQL |
| Orquestación | Kubernetes |
| Gestión de paquetes | Helm |
| GitOps | ArgoCD |
| Cluster local | Minikube |

---

# Estructura del repositorio

```text
charts/
└── pedido-app/
    ├── Chart.yaml
    ├── values.yaml
    ├── values-dev.yaml
    ├── values-prod.yaml
    └── templates/
        ├── backend-deployment.yaml
        ├── backend-service.yaml
        ├── frontend-deployment.yaml
        ├── frontend-service.yaml
        ├── db-deployment.yaml
        ├── db-service.yaml
        ├── db-pvc.yaml
        ├── ingress.yaml
        ├── secret.yaml
        └── hpa.yaml

environments/
├── dev/
│   └── application.yaml
└── prod/
    └── application.yaml
```


---

# Instalación manual con Helm

## Prerequisitos

Antes de iniciar se debe contar con:

- Kubernetes funcionando
- Minikube instalado
- kubectl configurado
- Helm 3 instalado

---

## Agregar repositorio Bitnami

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## Instalar en entorno de desarrollo

```bash
helm install pedido-dev ./charts/pedido-app \
-n pedido-dev \
--create-namespace \
-f values-dev.yaml
```

## Actualizar la aplicación

```bash
helm upgrade pedido-dev ./charts/pedido-app \
-n pedido-dev \
-f values-dev.yaml
```

## Desinstalar

```bash
helm uninstall pedido-dev -n pedido-dev
```

---

## Endpoints

| Servicio | Endpoint |
|---------|---------|
| Frontend | `minikube service pedido-dev-pedido-app-frontend` |
| Backend API | `http://pedido.local/api` |
| Base de datos | `postgresql://pedidos_user:userpass123@pedido-dev-postgresql` |

---

## Escalamiento automático (HPA)

El **backend** utiliza **Horizontal Pod Autoscaler (HPA)** para escalar automáticamente según el uso de CPU.

### Parámetros principales

| Parámetro | Descripción |
|-----------|-------------|
| `minReplicas` | Número mínimo de pods que siempre estarán en ejecución |
| `maxReplicas` | Número máximo de pods que Kubernetes puede crear |
| `targetCPUUtilization` | Porcentaje de uso de CPU que activa el escalamiento |

Cuando el uso de CPU supera el límite configurado, **Kubernetes crea nuevos pods automáticamente** para manejar la carga adicional.

---

## Persistencia de datos

La base de datos utiliza almacenamiento persistente mediante:

- **PersistentVolume (PV)**
- **PersistentVolumeClaim (PVC)**

Esto permite que los datos permanezcan almacenados incluso si el **pod se reinicia o se recrea**, evitando la pérdida de información en la base de datos.

---

## GitOps con ArgoCD

**ArgoCD** permite desplegar la aplicación automáticamente utilizando el **repositorio Git como fuente de verdad**.

Cada vez que se realiza un cambio en el repositorio, ArgoCD puede sincronizar el estado del clúster con la configuración definida en Git.

### Entornos configurados

| Entorno | Namespace | Archivo |
|--------|-----------|--------|
| Dev | `pedido-dev` | `application-dev.yaml` |
| Prod | `pedido-prod` | `application-prod.yaml` |

---

## Aplicar aplicaciones en ArgoCD

```bash
kubectl apply -f application-dev.yaml
kubectl apply -f application-prod.yaml
```

También pueden aplicarse desde la interfaz web de ArgoCD.

---

## Sincronización automática

Las aplicaciones tienen habilitada la siguiente configuración en **ArgoCD**:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```
Esto permite que ArgoCD:

- Detecte automáticamente cambios en el repositorio Git.
- Actualice el cluster de Kubernetes para mantenerlo sincronizado.
- Elimine recursos que ya no existen en el repositorio.
- De esta forma se mantiene un modelo GitOps, donde Git actúa como la fuente de verdad del estado deseado de la infraestructura.

---

## Demostración GitOps

Para demostrar la **sincronización automática con ArgoCD** se puede realizar el siguiente procedimiento:

### 1️ Cambiar el número de réplicas

Modificar el archivo `values-dev.yaml`:

```yaml
backend:
  replicas: 3
```

### 2 Hacer commit y push

```bash
git add .
git commit -m "Cambio de replicas para demo GitOps"
git push
```

### 3 ArgoCD detecta el cambio y sincroniza el cluster automáticamente.

---

## Diagrama del sistema

### Diagrama de Arquitectura

<img width="454" height="357" alt="image" src="https://github.com/user-attachments/assets/a90817b7-9922-44f5-9a78-e51e79eee304" />

### Diagrama del despliegue en Kubernetes

<img width="275" height="456" alt="image" src="https://github.com/user-attachments/assets/80ba8b73-44d4-4089-838a-f58597c259a3" />

---

# Video de Demostración

Se presenta un video mostrando el funcionamiento del sistema:

- Pods ejecutándose correctamente en Kubernetes
- Acceso al frontend desde el navegador
- Funcionamiento de la API backend
- Persistencia de datos en PostgreSQL
- Escalamiento automático mediante HPA
- Sincronización automática de ArgoCD

**Video:**

[Ver video](https://www.youtube.com/watch?v=iiLEIzqqyWk)

---

## Guía para la demostración del parcial

### PARTE 1 - Mostrar el cluster funcionando

```powershell
# Ver todos los pods corriendo
kubectl get pods -n demo

# Ver todos los servicios
kubectl get svc -n demo

# Ver ingress
kubectl get ingress -n demo

# Ver PVC (persistencia)
kubectl get pvc -n demo

# Ver HPA
kubectl get hpa -n demo

# Ver configmaps y secrets
kubectl get configmap -n demo
kubectl get secret -n demo
```

---

### PARTE 2 - Abrir ArgoCD

```powershell
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Abrir: https://localhost:8081
- Mostrar pedido-app-dev: Healthy + auto-sync activado
- Mostrar pedido-app-prod: Healthy + auto-sync activado

---

### PARTE 3 - Mostrar la app funcionando

```powershell
# Port-forward al ingress
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
```

Abrir: http://pedido-dev.local:8080
1. Presionar boton Recargar -> mostrar datos cargando del backend
2. Presionar + Agregar Dueno -> llenar el formulario -> Guardar
3. Recargar y verificar que el nuevo dueno aparece

---

### PARTE 4 - Demostrar persistencia de datos

```powershell
# Ver pod de postgresql antes de eliminar
kubectl get pods -n demo -l app=pedido-app-dev-postgresql

# Eliminar el pod de postgresql
kubectl delete pod -n demo -l app=pedido-app-dev-postgresql

# Ver como se recrea automaticamente
kubectl get pods -n demo -l app=pedido-app-dev-postgresql -w
```

---

### PARTE 5 - Demostrar GitOps (auto-sync ArgoCD)

### Paso 1: Hacer el cambio en Git
En `charts/pedido-app/values-dev.yaml` cambiar replicas de 2 a 3:

```yaml
backend:
  replicas: 3
```

### Paso 2: Push

```powershell
git add charts/pedido-app/values-dev.yaml
git commit -m "demo: cambiar replicas backend de 2 a 3"
git push
```

### Paso 3: Mostrar ArgoCD sincronizando
- Abrir ArgoCD en https://localhost:8081
- Ver como pedido-app-dev pasa a OutOfSync y luego sincroniza solo
- Esperar 1-3 minutos

### Paso 4: Verificar en el cluster

```powershell
# Debe mostrar 3 pods del backend
kubectl get pods -n demo -l app=backend -w
```

### Paso 5: Revertir el cambio

```powershell
# En values-dev.yaml cambiar replicas de 3 a 2
git add charts/pedido-app/values-dev.yaml
git commit -m "demo: revertir replicas a 2"
git push

# Verificar que ArgoCD vuelve a sincronizar a 2 pods
kubectl get pods -n demo -l app=backend -w
```

---

### PARTE 6 - Mostrar HPA

```powershell
# Ver HPA configurado
kubectl get hpa -n demo

# Ver detalle del HPA
kubectl describe hpa pedido-app-dev-backend -n demo
```
El HPA está configurado para escalar el backend entre 2 y 10 réplicas cuando el CPU supere el 70%. Actualmente muestra unknown en las métricas porque Minikube no tiene el metrics-server habilitado por defecto. En un ambiente real de producción esto funcionaría correctamente. Para habilitarlo en Minikube se ejecuta minikube addons enable metrics-server
---

### PARTE 7 - Mostrar el README en GitHub

Abrir: https://github.com/JuanLacouture/pedido-app

---

### COMANDOS RAPIDOS DE EMERGENCIA

```powershell
# Si el frontend no carga
kubectl port-forward -n demo svc/pedido-app-dev-frontend 8888:80

# Si ArgoCD no abre
kubectl port-forward svc/argocd-server -n argocd 8081:443

# Ver logs del backend
kubectl logs -n demo deployment/pedido-app-dev-backend --tail=20

# Ver todos los recursos de un vistazo
kubectl get all -n demo
```

---

## Conclusiones

En este proyecto se implementó una aplicación Full Stack desplegada en Kubernetes utilizando buenas prácticas de despliegue moderno y automatización.

Se cumplieron los entregables solicitados:

- Se creó un **repositorio Git** que contiene el **Helm Chart completo** de la aplicación junto con las configuraciones necesarias para su despliegue.
- Se incluyeron las **definiciones de ArgoCD**, permitiendo gestionar los despliegues mediante un enfoque **GitOps**, donde el repositorio actúa como fuente de verdad del estado del clúster.
- En el **README del proyecto** se documentó:
  - Cómo instalar la aplicación manualmente usando **Helm**.
  - Cómo está configurado **ArgoCD** para sincronizar automáticamente los cambios desde el repositorio.
  - Los **endpoints de acceso** tanto para el frontend como para el backend.

Adicionalmente, el sistema incorpora características importantes del ecosistema cloud-native como:

- **Escalamiento automático con Horizontal Pod Autoscaler (HPA)**.
- **Persistencia de datos mediante Persistent Volumes**.
- **Despliegue reproducible utilizando Helm Charts**.

Durante la demostración se evidenció el funcionamiento del enfoque **GitOps**, realizando un cambio en el archivo `values.yaml` (por ejemplo, modificar el tag de la imagen o el número de réplicas) y mostrando cómo **ArgoCD detecta el cambio y actualiza automáticamente el clúster sin intervención manual**.

Este proyecto demuestra cómo herramientas modernas como **Kubernetes, Helm y ArgoCD** permiten construir sistemas **escalables, reproducibles y automatizados**, alineados con prácticas actuales de **DevOps y despliegue continuo**.

---

# Autores

- Juan José Cárdenas Carrero
- Juan Andrés Lacouture

---



