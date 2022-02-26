1. Test Django
```
python manage.py test
```
2. Build container
```
docker build -f Dockerfile \
-t registry.digitalocean.com/django-k8s/djk8app:latest \
-t registry.digitalocean.com/django-k8s/djk8app:v1 \
.
```

3. Push this container to Digital Ocean
```
docker push registry.digitalocean.com/django-k8s/djk8app --all-tags
```

4. Update secretes
```
    kubectl delete secret django-k8s-prod-secret
    kubectl create secret generic django-k8s-prod-secret --from-env-file=web/.env.prod
```
5. Update Deployment
```
kubectl apply -f k8s/apps/django-k8s-web.yaml
```
6. Wait for Rollout to Finish
```
kubectl rollout status deployment/django-k8s-web-deployment
```
7. Migrate database
```
export SINGLE_POD_NAME=$(kubectl get pod -l app=django-k8s-web-deployment -o jsonpath="{.items[0].metadata.name}")

kubectl exec -it $SINGLE_POD_NAME -- bash /app/migrate.sh

      # POSTGRES_DB=${{ secrets.POSTGRES_DB }}
          # POSTGRES_PASSWORD=${{ secrets.POSTGRES_PASSWORD }}
          # POSTGRES_USER=${{ secrets.POSTGRES_USER }}
          # POSTGRES_HOST=${{ secrets.POSTGRES_HOST }}
          # POSTGRES_PORT=${{ secrets.POSTGRES_PORT }}