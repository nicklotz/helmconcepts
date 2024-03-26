# Lab 07: Work with Helm Plugins

## A. Install and use third party plugins

1. List existing Helm plugins.

```
helm plugin list
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

2. Create a simple plugin/file script called **helm-user**.

```
touch helm-user
```
```
chmod +x helm-user
```

3. Populate **helm-user** with the following text.

```
#!/bin/bash
finger $USER
```

3. Create another file called **plugin.yaml** and populate it with the following content.

```yaml
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

## C. Use the Helm S3 plugin

1. Install helm s3 plugin as well as the AWS command line tools if you don't have them, for example. 

```
helm plugin install https://github.com/hypnoglow/helm-s3.git
```
```
sudo apt-get install -y awscli
```

2. Authenticate with the command line client.

```
aws configure
```

**Note:** Refer to the [AWS documentation](https://docs.aws.amazon.com/cli/v1/userguide/cli-authentication-user.html) for what's needed to authenticate.

3. Set a variable representing a unique name for an S3 bucket.

```
BUCKETNAME="helmbucket$(date +%s)"
```
```
echo $BUCKETNAME
```

4. Create a private S3 bucket.

```
aws s3 mb s3://$BUCKETNAME
```
```
aws s3 ls
```

5. Initialize the bucket to be a chart repository.

```
helm s3 init s3://$BUCKETNAME/charts
```

6. Verify the initialization. You should see an **index.yaml** file was created.

```
aws s3 ls s3://$BUCKETNAME --recursive
```

5. Create a dedicated repo for your **pluginscharts**.

```
helm repo add pluginschart s3://$BUCKETNAME/charts
```

6. Verify the repo reference locally.

```
helm repo list
```

7. Package and push the current version of the **pluginschart** to S3.

```
helm package plugins
```
```
helm s3 push pluginschart-0.2.0.tgz pluginschart
```

8. Verify the push succeeded.

```
aws s3 ls s3://$BUCKETNAME --recursive
```

9. Delete the local chart file, then fetch it from S3.

```
rm pluginschart-0.2.0.tgz
```
```
helm fetch pluginschart/pluginschart --version 0.2.0
```

## D. Unit testing

Helm has a **unittest** plugins for unit testing charts (not to be confused with the built-in **helm test** command**.

1. Install the **unittest** plugin.

```
helm plugin install https://github.com/helm-unittest/helm-unittest.git
```

2. Create a new chart with helm defaults.

```
helm create unittestchart
```
```
cd unittestchart
```

3. Paste and run the following to create a new unit test in **unittestchart/tests/**.

```
mkdir tests/
```
```
cat << EOF > tests/deployment_test.yaml
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should be a deployment with a set replica count
    set:
      replicaCount: 3
    asserts:
      - hasDocuments:
          count: 1
      - isKind:
          of: Deployment
      - equal:
          path: spec.replicas
          value: 3
  - it: manifest should match snapshot
    asserts:
      - matchSnapshot: {}
EOF
```
```
cat tests/deployment_test.yaml
```

4. Run unit tests (which is just one in this case)/

```
helm unittest --update-snapshot .
```

5. Make a couple changes to the deployment manifest.

```
sed -i 's/replicas/# replicas/g' templates/deployment.yaml
```
```
sed -i 's/Deployment/NotDeployment/g' templates/deployment.yaml
```

6. Run unit tests again. You should notice a couple errors.

```
helm unittest --update-snapshot .
```

7. Revert the changes and run unit tests again. It should pass.

```
sed -i 's/# replicas/replicas/g' templates/deployment.yaml
```
```
sed -i 's/NotDeployment/Deployment/g' templates/deployment.yaml
```
```
helm unittest --update-snapshot .
```
