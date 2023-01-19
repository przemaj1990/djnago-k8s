1. Test Django
```
source /home/cloud_user/Projects/Djnago-on-k8s/djnago-k8s/dev/django-k8s/venv/bin/activate`
python manage.py test`
```
2. Build container
```
docker build -f Dockerfile \
    -t docker.io/przemaj1990/django-k8s:latest \
    -t docker.io/przemaj1990/django-k8s:v1 \
    .
```
3. Push Container to container registry:
```
docker push docker.io/przemaj1990/django-k8s --all-tags
```
4. Update secrets

```
kubectl delete secret django-k8s-web-prod-env
kubectl create secret generic django-k8s-web-prod-env --from-env-file=.env
```
5. Update Deployment
```
kubectl apply -f /k8s/apps/django-k8s-web.yaml
```
6. Wait for Rollout to finish
```
kubectl rollout status deployment/django-k8s-web-deployment
```
7. Migrate databse
```
export SINGLE_POD_NAME=$(kubectl get pod -l app=django-k8s-web-deployment -o jsonpath="{.items[0].metadata.name}")
or export SINGLE_POD_NAME=$(kubectl get pod -l=app=django-k8s-web-deployment -o NAME | tail -n 1)

kubectl exec -it $SINGLE_POD_NAME -- bash /app/migrate.sh
```
8. 