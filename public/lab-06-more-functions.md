# Lab 06: More Functions Practice

## A. Create a new chart and custom function

1. Spend a couple minutes looking through [this list](https://helm.sh/docs/chart_template_guide/function_list/) of supported Helm template functions.

2. From a clean workspace/directory, create a new Helm chart.

```
helm create functionschart
```

3. Change into the chart directory.

```
cd functionschart/
```

4. Paste and run the following to create a new **ConfigMap** template that stores several pieces of data.

```yaml
cat << EOF > templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  isapproved: "{{ empty .Values.approver | not}}"
  deploytime: "{{ now | unixEpoch }}"
  messagesha: "{{ sha256sum .Values.deploymessage }}"
EOF
```

What are the template functions used in this ConfigMap? Look them up in the Helm documentation to see what they do.

5. Paste and run the following to add new keys to **values.yaml**.

```yaml
cat << EOF >> values.yaml

approver: "nick"

deploymessage: "To infinity and beyond!"
EOF
```

6. Check for any syntax errors.

```
helm lint .
```

7. Deploy the new Helm chart.

```
helm install functionschart .
```

8. Check if the ConfigMap was created. Does it have the number of data objects you expect?

```
kubectl get configmap
```

9. Display the ConfigMap's values. Are they what you'd expect?

```
kubectl get configmap functionschart-configmap -o yaml
```

10. Uninstall the current release.

```
helm uninstall functionschart
```

11. Reinstall the Helm chart with a runtime input.

```
read -p "Approver's name: " APPROVER && helm install functionschart . --set approver="$APPROVER"
```

12. Enter your name (or any name) and press **Return**.

13. Check the ConfigMap. Is the **isapproved** value what you expect? How does the template determine this?

```
kubectl get configmap functionschart-configmap -o yaml
```

## B. Create and use external functions file

1. Create a new file in **functionschart/templates**.

```
touch templates/customfunctions.tpl
```

2. Add a new function to your custom functions file.

```go
cat << EOF >> templates/customfunctions.tpl
{{- define "customfunctions.enforceloweralphas" -}}
{{ . | camelcase | nospace | lower }}
{{- end -}}
EOF
```
```
cat templates/customfunctions.tlp
```

3. Add a new value to your ConfigMap.

```yaml
cat << EOF >> templates/configmap.yaml
  deployregion: "{{ include "customfunctions.enforceloweralphas" .Values.deployregion }}"
EOF
```
```
cat templates/configmap.yaml
```

4. Updates values.yaml.

```yaml
cat << EOF >> values.yaml

deployregion: "Bismarck North_Dakota"
EOF
```

5. Check how the function would be applied to your values (no need to deploy it turns out).

```
helm template .
```

How does the custom function manipulate the value of **deployregion** in the config map?

Next we'll add a loop to apply the custom function iteratively to data in a list.

6. Run the following to replace **deployregion** in **values.yaml** with multiple deployment regions.

```
sed -i '/^deployregion/d' values.yaml
```
```
sed -i '/^deployregion/d' templates/configmap.yaml
```
```yaml
cat << EOF >> values.yaml

deployregions:
  - region: "Bismarck North_Dakota"
  - region: "Washington DC"
  - region: "Seoul South_Korea"
EOF
```
```
cat -n values.yaml
```

7. Modify **configmap.yaml** to store the regions, parsed by your custom function.

```yaml
cat << EOF >> templates/configmap.yaml
  deployregions:
      {{- range .Values.deployregions }}
      - region: "{{ include "customfunctions.enforceloweralphas" .region }}"
      {{- end }}
EOF
```

8. Check to see how the template function is applied to a list.

```
helm template .
```
