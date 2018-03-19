# Update the Web Application using Kubernetes CLI


### Introduction
In this section you will learn how to update the Web App your just deployed in step 5 and scale this component in your Kubernetes cluster. The web application is managed as a Kubernetes Deployment component. You will get to see the deployment's history, and also how to rollback the deployment to a previous version.

### Pre-Requisites
* Complete up to [Step 5](https://cloudcontent.mybluemix.net/devops/method/tutorials/kubernetes?task=5) of the Kubernetes Tutorial.

### Step 1: Checkout tutorial code
1. Checkout `stsa` branch:

```
$ cd ../refarch-cloudnative-bluecompute-web
$ git checkout -b stsa
$ git pull origin stsa
```

2. Open the `StoreWebApp/public/resources/components/views/home.html` in your code editor to see the new changes.
    * You should see a header tag that says `Welcome to STSA!` instead of `BlueCompute Store!`

### Step 2: Update the Web App Deployment with a new Docker Image

On a typical Kubernetes deployment scenario, a developer will perform the following steps:

a). Make changes to application code.  
b). Build a new Docker image with new code changes.  
c). Push the new Docker image to a Docker registry.  
d). Deploy the new Docker image by updating the Kubernetes Deployment by doing the following:  
  * `kubectl set image deployment/<deployment-name> <container-name>=<image-name>:<tag>`  
     * `<deployment-name>` is the deployment name in the `web.yaml` file.  
     * `<container-name>` is the container name in the `web.yaml` file.  
     * `<image-name>` is the name of the new Docker image that was pushed to the Docker registry.  
     * `<tag>` is the tag of the new Docker image that was pushed to the Docker registry.  

To avoid spending too much time building docker images in this tutorial, we created and pushed a pre-built Docker image to the `ibmcase` registry in Docker Hub. You can check out the `stsa` image in Docker Hub [here](https://hub.docker.com/r/ibmcase/bluecompute-web/tags/).

Kubernetes defines its components and deployment through sets of yaml files. In this case, the web application deployment and services are defined in the `web.yaml` file. Take a minute to quickly review this file if you can.

#### 1. Confirm KUBECONFIG is set
Ensure your terminal/command window has kubectl setup to configure *cloudnativedev* cluster.

```
$ kubectl config current-context
cloudnativedev
```

If above command did not return your cluster name, then set the kube configuration by running `export KUBECONFIG=...` command.

Get the correct export command from the output of `bx cs cluster-config cloudnative` and run it in your terminal/command window. A sample export command will look like `export KUBECONFIG=/Users/ibm/.bluemix/plugins/container-service/clusters/cloudnativedev/kube-config-prod-mel01-cloudnativedev.yml`.

Run `kubectl config current-context` again to confirm kubectl is now set to manage your cloudnativedev cluster.

For detailed instructions to configure kubectl, refer to [Task 7 Steps 1 through 4 from the tutorial](https://www.ibm.com/devops/method/tutorials/kubernetes?task=7/).

#### 2. Update Docker image
Now let's update the Web app docker image as follows:

```
$ kubectl set image deployment/bluecompute-web-deployment web-ce=ibmcase/bluecompute-web:stsa
```

The *web-ce* parameter is the Container name defined in the web.yaml file.

#### 3. Validate Deployment History
Let's validate that the new deployment completed successfully.

Run the following command, and from the output verify the **Image** name and tag of the *Running* pods match to what was set (**ibmcase/bluecompute-web:stsa**) in previous step.

```
$ kubectl describe rs bluecompute-web-deployment

Name:           bluecompute-web-deployment-3031554331
Namespace:      default
Image(s):       ibmcase/bluecompute-web:stsa
Selector:       app=bluecompute,chart=web-0.1.0,micro=web-bff,pod-template-hash=3031554331,release=web,tier=frontend
Labels:   app=bluecompute
          chart=web-0.1.0
          micro=web-bff
          pod-template-hash=3031554331
          release=web
          tier=frontend
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
No volumes.
No events.
```
#### 4. Validate the Web App
Now that we updated the deployment's image and replicas, let's open the web application in a web browser and validate that the new code is running. You can use the web app url from the tutorial Task 5 (step 4), or execute the following 2 commands to get the web application IP address and port:

```
# Get node IP
$ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# Get Web Port
$ kubectl get service bluecompute-web -o jsonpath='{.spec.ports[0].nodePort}'
```

Now, open a new browser window and paste the URL (http://ip:port). If successful, you should see web page that looks like the following:

![BlueCompute List](static/bluecompute_stsa.png?raw=true)

You have successfully updated the Bluecompute Web App Deployment!

### Step 3: Scale Up Deployment Replicas

#### 1. Check existing deployment replicas
Now that we updated the deployment to a new Docker image, let's try to scale up the number of pod replicas (Pods are the runtime unit in Kubernetes, it contains one or more Docker containers) in the deployment. Before doing so, let's check how many replicas the deployment currently has with the following command:

```
$ kubectl get deployment bluecompute-web-deployment
NAME                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
bluecompute-web-deployment   1         1         1            1           50m
```

Notice under the `CURRENT` column that the deployment currently has `1` replica.

#### 2. Update the number of replicas in deployment
Now, let's scale it up to `2` replicas with the following command:

```
$ kubectl scale deployment bluecompute-web-deployment --replicas=2
deployment "bluecompute-web-deployment" scaled
```

If successful, you should see an output similar to the above.

#### 3. Validate the new number of replicas in deployment
Now let's validate that the deployment has `2` replicas with the following command:

```
$ kubectl get deployment bluecompute-web-deployment
NAME                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
bluecompute-web-deployment   2         2         2            2           54m
```

Notice under the `CURRENT` column that the deployment now has `2` replicas, which means that the new replica was created successfully!

#### 4. Validate the Web App
From the web browser you have the web app running, refresh the page, you should see your application still running.
You have successfully scaled up the Bluecompute Web App Deployment!


### Step 4: Rollback changes
Now let's rollback the web application deployment to it's original conditions.

#### 1. Scale down number of replicas in deployment
Now, let's scale it down to `1` replica with the following command:

```
$ kubectl scale deployment bluecompute-web-deployment --replicas=1
deployment "bluecompute-web-deployment" scaled
```

If successful, you should see an output similar to the above.

#### 2. Validate the number of replicas in deployment
Now let's validate that the deployment has `1` replica with the following command:

```
$ kubectl get deployment bluecompute-web-deployment
NAME                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
bluecompute-web-deployment   1         1         1            1           1h
```

Notice under the `CURRENT` column that the deployment now has `1` replicas, which means that we scalled down the replica number successfully!

#### 3. Rollback deployment
Now let's rollback the Web App deployment to the original image with the following command:
```
$ kubectl rollout undo deployment/bluecompute-web-deployment
deployment "bluecompute-web-deployment" rolled back
```

If successful, you should see an output similar to above.

#### 4. Validate the Web App Rollback Changes
From the web browser you have the web app running, refresh the page, you should see your application still running. you should see web page that looks like the following:

![BlueCompute List](static/bluecompute_stsa_original.png?raw=true)

You have successfully rolled back the Bluecompute Web App Deployment!
