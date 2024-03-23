## A. Add required dependencies to your helm chart

1. In your terminal, navigate to the **myapp/helm/myapp** directory.

2. Paste and run the following to update your chart to require **MySQL** and **Redis** as dependencies.

```yaml
cat << EOF >> Chart.yaml 

dependencies:
  - name: mysql
    version: "10.1.0"
    repository: "https://charts.bitnami.com/bitnami"
  - name: redis
    version: "19.0.1"
    repository: "https://charts.bitnami.com/bitnami"
EOF
```

3. Also run the following to incrment your **chart** version to **0.3.0**.

```
sed -i 's/version: 0.2.0/version: 0.3.0/g' Chart.yaml
```

4. Verify the changes to Chart.yaml.

```
cat Chart.yaml
```

5. Update helm with the required dependencies.

```
helm dependency update
```

6. Check the **charts/** directory to see if the dependencies have been added.

```
ls -l charts/
```

7. Also inspect the Chart.lock file that should be newly created in the **helm/myapp/** directory.

```
cat Chart.lock
```

8. Check for any syntax errors.

```
helm lint .
```

## B. Build and deploy the helm chart with added dependencies

1. Navigate back up to the **myapp/helm** directory.

```
cd ../
```
```
pwd
```

2. Package up the updated **myapp** helm chart.

```
helm package myapp
```

3. Deploy **myapp** with **mysql** and **redis**.

```
helm install myapp ./myapp-0.3.0.tgz
```

4. Verify helm performed the deployment.

```
helm list
```

5. Verify connectivity to the **myapp** service.

```
curl http://localhost
```

6. Inspect all the deployed services in the default namespace.

```
kubectl get service
```
```
kubectl get pods
```

7. Inspect the **myapp-mysql** service. What does the **Annotations:** field tell you about how this service is managed?

```
kubectl describe service myapp-mysql
```

## C. Cleanup
1. Delete the currently running **myapp** release.

```
helm uninstall myapp
```
```
helm list
```

2. Delete the persistent volumes used by the database services.

```
kubectl delete pvc --all
```

