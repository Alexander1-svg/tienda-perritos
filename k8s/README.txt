# Despliegue tienda-mascotas en EKS (namespace 'tienda')

## 1) Configurar kubectl contra el cluster EKS:

```bash
aws eks update-kubeconfig --region us-east-1 --name tienda-perritos-eks

2) Crear namespace:
    kubectl apply -f namespace.yaml

Validar:

  kubectl get namespaces

3) Desplegar base de datos MySQL:

Aplicar secreto de credenciales:

kubectl apply -f mysql-secret.yaml

Crear Deployment de MySQL:

kubectl apply -f mysql-deployment.yaml

Crear Service de base de datos:

kubectl apply -f mysql-service.yaml

4) Desplegar Backend:

Crear Deployment del backend:

kubectl apply -f backend-deployment.yaml

Crear Service del backend:

kubectl apply -f backend-service.yaml

Validar funcionamiento:

kubectl get pods -n tienda

5) Desplegar Frontend:

Crear Deployment del frontend:

kubectl apply -f frontend-deployment.yaml

Crear Service LoadBalancer:

kubectl apply -f frontend-service.yaml

6) Verificar recursos desplegados:

Ver pods:

kubectl get pods -n tienda

Ver servicios:

kubectl get svc -n tienda

7) Acceso a la aplicación:

Copiar el valor de EXTERNAL-IP del servicio:

kubectl get svc tienda-frontend -n tienda

Ejemplo:

k8s-tienda-tiendafr-xxxxxxxx.elb.us-east-1.amazonaws.com

Ingresar la dirección en el navegador:

http://EXTERNAL-IP

Se debe visualizar la página web de Tienda Mascotas 🐾 y permitir la consulta de productos mediante el backend.

8) Verificación del Backend:

Para comprobar que la API funciona correctamente:

kubectl exec -n tienda -it <nombre-pod-backend> -- sh

Ejecutar:

wget -qO- localhost:3001/api/productos

Debe retornar la lista de productos almacenados en la base de datos.

9) Problemas comunes:
Pod en estado Pending

Validar recursos disponibles del cluster:

kubectl describe pod <nombre-pod> -n tienda

Revisar configuración de CPU y memoria en los archivos Deployment.

Frontend en CrashLoopBackOff

Revisar logs:

kubectl logs <nombre-pod-frontend> -n tienda

Validar configuración de Nginx y comunicación con el servicio backend.

Reiniciar despliegue después de cambios:
kubectl rollout restart deployment tienda-frontend -n tienda
kubectl rollout restart deployment tienda-backend -n tienda