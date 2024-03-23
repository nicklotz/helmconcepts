# Lab 04: Templating

## A. Understand template and values.yaml overrides

1. In a new/empty directory create a new helm chart.

```
helm create myapptemplate
```

2. Change into the myapptemplate directory.

```
cd myapptemplate/
```

3. Inspect **values.yaml**. Which version of nginx will be deployed?

```
cat values.yaml
```

4. Package and deploy the helm chart as is.

```
cd ../
```
```
helm package myapptemplate
```
```
helm install myapptemplate myapptemplate --version 0.1.0
```

5. Check what image your deployed application is using. Is it what you expect?

```
kubectl describe pods | grep Image
```

6. Navigate to **myapptemplates/templates/**

```
cd myapptemplates/templates
```

7. Inspect **deployment.yaml**. Which line(s) specify the Docker image used in the deployment?

```
cat -n deployment.yaml
```

8. Make the following modification to **deployment.yaml**. Do you think this is a good change?

```
sed -i 's/{{ .Values.image.tag | default .Chart.AppVersion }}/1.24.0/g' deployment.yaml
```
```
cat deployment.yaml | grep -n image
```

9. Increment the chart version in **Chart.yaml**.

```
cd ../
```
```
sed -i 's/version: 0.1.0/version: 0.2.0/g' Chart.yaml
```
```
cat -n Chart.yaml
```

10. Check for any syntax errors.

```
helm lint .
```

11. Package a new version of the chart.

```
cd ../
```
```
helm package myapptemplate
```

12. Upgrade the existing release with the new chart version.

```
helm upgrade myapptemplate myapptemplate --version 0.2.0
```

13. Check the image version in the new release. What has changed?

```
kubectl describe pods | grep Image
```

14. Change back into the chart directory and change the image tag in **values.yaml**.

```
cd myapptemplate/
```
```
sed -i 's/tag: ""/tag: 1.25.0/g' values.yaml
```
```
cat values.yaml | grep -n tag:
```

15. Increment the chart version in **Chart.yaml**.

```
sed -i 's/version: 0.2.0/version: 0.3.0/g' Chart.yaml
```
```
cat Chart.yaml | grep -n version
```

16. Check for any syntax errors.

```
helm lint .
```

17. Package a new version of the chart.

```
cd ../
```
```
helm package myapptemplate
```

18. Upgrade the existing release with the new chart version.

```
helm upgrade myapptemplate myapptemplate --version 0.3.0
```
19. Check the image version in the new release. 

```
kubectl describe pods | grep Image
```

How has it changed? What does this tell you about the precedence of override values in Helm charts? In which of **Chart.yaml**, **values.yaml**, or **deployment.yaml** should we have set the tag?

20. Navigate into **myapptemplates/templates**.

```
cd myapptemplates/templates
```

21. Restore **deployment.yaml** back to its previous configuration

```
sed -i 's/1.24.0/{{ .Values.image.tag | default .Chart.AppVersion }}/g' deployment.yaml
```
```
cat deployment.yaml | grep -n tag
```

22. Increment chart version and check for errors.
```
cd ../
```
```
sed -i 's/version: 0.3.0/version: 0.4.0/g' Chart.yaml
```
```
cat Chart.yaml | grep -n version
```
```
helm lint .
```

23. Package and deploy the updated chart.
```
cd ../
```
```
helm package myapptemplate
```
```
helm upgrade myapptemplate myapptemplate --version 0.4.0
```

24. Check the image tag in the updated deployment. Where is the value used coming from?

```
kubectl describe pods | grep Image
```

25. Finally, update the chart, this time with a value passed in at install.

```
helm upgrade myapptemplate myapptemplate --version 0.4.0 --set image.repository=nginx,image.tag=1.25.4
```
```
kubectl describe pods | grep Image
```

26. Cleanup the current release.

```
helm uninstall myapptemplate
```

## B. Go logic in templates

1. Navigate into **myapptemplate/templates**.

```
cd myapptemplate/templates
```

2. Paste the following into the **spec.containers** portion of **deployments.yaml**, just after line 53.

```yaml
          env:
           - name: ENVIRONMENT
             value: test
          {{- end }}
```

The indentation relative to the surrounding code should look like this:

```
    50	          volumeMounts:
    51	            {{- toYaml . | nindent 12 }}
    52	          {{- end }}
    53	          {{- if .Values.environment.isTest }}
    54	          env:
    55	           - name: ENVIRONMENT
    56	             value: test
    57	          {{- end }}
    58	      {{- with .Values.volumes }}
    59	      volumes:
    60	        {{- toYaml . | nindent 8 }}
    61        {{- end }}
```

3. Navigate up to **myapptemplates**. Paste and run the follwoing to add a booleon environment variable to **values.yaml**.

```
cd ../
```
```
```yaml
cat << EOF >> values.yaml
environment:
  isTest: false
EOF
```
```
cat -n values.yaml
```

4. Package and update the release.

```
cd ../
```
```
helm package myapptemplate
```
```
helm upgrade myapptemplate myapptemplate --version 0.4.0 --set environment.isTest=true
```

5. Check the deployment's environment variables.

```
kubectl set env deployment/myapptemplate --list
```

We can also use **range**, basically a for loop, to iterate over an arbitrary list of values.

6. Navigate back to **myapptemplates/templates/**.

```
cd myapptemplates/templates/
```

7. Paste the following code below line 57 in **deployment.yaml**, i.e. underneath your previous change.

```yaml
          env:
          {{- range .Values.envVars }}
           - name: {{ .name }}
             value: {{ .value }}
          {{- end }}
```

That section of deployment.yaml should therefore look like:

```yaml
    50	          volumeMounts:
    51	            {{- toYaml . | nindent 12 }}
    52	          {{- end }}
    53	          {{- if .Values.environment.isTest }}
    54	          env:
    55	           - name: ENVIRONMENT
    56	             value: test
    57	          {{- end }}
    58	          env:
    59	          {{- range .Values.envVars }}
    60	           - name: {{ .name }}
    61	             value: {{ .value }}
    62	          {{- end }}
    63	      {{- with .Values.volumes }}
    64	      volumes:
    65	        {{- toYaml . | nindent 8 }}
```

8. Add some environment variables to **values.yaml**.

```
cd ../
```
```yaml
cat << EOF >> values.yaml

envVars:
- name: os
  value: ubuntu
- name: region
  value: emea
- name: tier
  value: premium
EOF
```

9. Package and update the release.

```
cd ../
```
```
helm package myapptemplate
```
```
helm upgrade myapptemplate myapptemplate --version 0.4.0 --set environment.isTest=true
```

10. Check the deployment's environment variables. Do you see the variables you defined?

```
kubectl set env deployment/myapptemplate --list
```
