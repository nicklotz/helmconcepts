# Lab 07: Work with Helm Plugins

## A. Install and use third party plugins

1. List existing Helm plugins.

```
helm plugins list
```

2. Download a new, third party plugin, [helm diff](https://github.com/databus23/helm-diff).

```
helm plugin install https://github.com/databus23/helm-diff
```

3. To test the plugin, create a new chart.

```
helm create pluginschart
```
```
cd pluginschart
```

4. Deploy the chart as-is.

```
helm install pluginschart .
```

5. Now modify one of the defaults in **values.yaml** and increment the chart version.

```
sed -i 's/replicaCount: 1/replicaCount: 2/g' values.yaml
```
```
sed -i 's/version: 0.1.0/version: 0.2.0/g' Chart.yaml
```

6. Install the updated chart as a separate release.

```
helm install pluginschartv2 .
```

7. Generate a diff of the manifests between the two releases.

```
helm diff release pluginschartv2 pluginschart
```

## B. Write a simple plugin

1. Create a directory to store a plugin called **helm-user**.

```
mkdir helm-user/
```
```
cd helm-user/
```

2. Paste and run the following to create a simple plugin/file script.

```
cat << EOF > helm-user
#!/bin/bash
finger $USER
EOF
```
```
chmod +x helm-user
```

3. Paste and run the following to create a **plugin.yaml** containing plugin metadata.

```
cat << EOF > plugin.yaml
name: "user"
version: "0.1.0"
usage: "helm user"
description: "This plugin shows details about the current user using Helm"
command: "$HELM_PLUGIN_DIR/helm-user"
```

4. Install the plugin locally.

```
helm plugin install $(pwd)
```

5. Test the plugin.

```
helm user
```



