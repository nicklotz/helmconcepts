# Lab 08: Subcharts

## A. Create a parent-subchart structure

1. Initialize a new helm chart called **parentchart**.

```
helm create parentchart
```

2. Navigate into **parentchart/charts**.

```
cd parentchart/charts
```

3. Create a subchart within the parent.

```
helm create subchart
```

4. Deploy the parent-subchart to Kubernetes.

```
cd ../
```
```
helm install mycharts .
```

5. View the release. You should only see the parent chart in **helm list**, but you should see the Kubernetes resources of both charts deployed.

```
helm list
```
```
kubectl get deployments
```
```
kubectl get svc
```
```
kubectl get pods
```

6. Test the release with the default **integration test**.

```
helm test
```

7. Uninstall the release.

```
helm uninstall mycharts
```

8. Verify the resources from both charts were removed.

```
kubectl get deployments
```

## B. Manage override values

1. Paste and run the following to create a **ConfigMap** resource in your subchart.

```
cat << EOF >> charts/subchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "subchart.fullname" . }}-configmap
data:
  os: "{{ .Values.os }}"
  arch: "{{ .Values.arch }}"
EOF
```
```
cat charts/subchart/templates/configmap.yaml
```

2. Add values to the *parent* chart's **values.yaml**. 

```
cat << EOF >> values.yaml

os: mac

arch: arm64
```
```
cat -n values.yaml
```

3. Check the values in the subchart's ConfigMap. Is it what you expect? Why or why not?

```
helm template . | grep configmap -A 4
```

4. Add values to the *subchart's* **values.yaml**.

```
cat << EOF >> charts/subchart/values.yaml

os: linux

arch: amd64
EOF
```
```
cat -n charts/subchart/values.yaml
```

5. Check the values in the subchart's ConfigMap. Which values does it use?

```
helm template . | grep configmap -A 4
```

6. Add another set of values to the *parent* chart's **values.yaml**.

```
cat << EOF >> values.yaml

subchart:
  os: windows
  arch: x86_64
EOF
```
```
cat -n values.yaml
```

7. Check the values in the subchart's ConfigMap again. What values does it use?

```
helm template . | grep configmap -A 4
```

8. Create a set of global of values in the parent **values.yaml**.

```
cat << EOF >> values.yaml

global:
  os: solaris
  arch: systemz
EOF
```
```
cat -n values.yaml
```

9. Modify the subchart's configmap to use global values.

```
sed -i 's/Values.os/Values.global.os/g' charts/subchart/templates/configmap.yaml
```
```
sed -i 's/Values.arch/Values.global.arch/g' charts/subchart/templates/configmap.yaml
```
```
cat -n charts/subchart/templates/configmap.yaml
```

10. Check the values in the subchart's ConfigMap again. Does it use the global values?

```
helm template . | grep configmap -A 4
```

**NOTE:** All the steps we performed worked fine given the implicit dependency relationship of **subchart** existing as a directory within **parentchart**.

However it's generally considered best practice to declare dependencies explicitly, like as follows within the *parent* chart's Chart.yaml.

```yaml
dependencies:
  - name: subchart
    version: 0.1.0
    repository: "file://charts/subchart"
```

