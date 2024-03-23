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
