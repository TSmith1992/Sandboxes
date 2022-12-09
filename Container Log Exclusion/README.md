# Container Log Exclusion  With Datadog

Objective
-----------

To understand and replicate the process of excluding logs from certain containers in a Kubernetes environment, as described in the Container Discovery Management documentation below:

```
https://docs.datadoghq.com/agent/guide/autodiscovery-management/?tab=containerizedagent#pagetitle
```

![Container Discovery Management 2022-10-28 at 2 31 18 PM](https://user-images.githubusercontent.com/60328238/198711855-f43d7ab1-cc91-40c2-a59d-d09b5e0c0bc1.jpg)


This is distinct from the practice of excluding/including certain logs from certain containers, discussed in our Advanced Logging documentation:

```
https://docs.datadoghq.com/agent/logs/advanced_log_collection/?tab=configurationfile#pagetitle
```

![Advanced Log Collection 2022-10-28 at 2 34 59 PM](https://user-images.githubusercontent.com/60328238/198711744-ad91a93b-d2fe-4c20-bf1e-1b859ccc9ad9.jpg)


Prerequisites
-------------

As a pre-requisite you should install Docker Desktop for Mac, as well as install Minikube and Kubernetes tools needed.

[Install Docker Desktop](https://www.docker.com/products/docker-desktop/)
[Install command line tools with brew
](https://brew.sh/)

```
brew install minikube
brew install kubernetes-cli
brew install helm
```

Add the Datadog Helm Repository

```
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

Once this is installed run minikube start to initialize your Minikube Kubernetes Cluster. The default parameters of this should be sufficient. Once this is ready you can run the following to validate the setup is ready.

```
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```


Setting up API Key
---------

The following sessions provide a Datadog Agent configuration via Helm. These samples reference a pre-existing Secret storing your Datadog API Key. To create this you can run the following with respect to one of your API Keys:

```
kubectl create secret generic datadog-agent --from-literal='api-key=<API_KEY>'
```

Example:

```
kubectl create secret generic datadog-agent --from-literal='api-key=abcdabcdabcdabcdabcdabcdabcdabcd'
```

When running this please make sure you swap in your api key correctly. You can validate that your API Key exists correctly by running:

```
$ kubectl describe secret datadog-agent
Name:         datadog-agent
Namespace:    default
Labels:       <none>
Annotations:  <none>
```

Type:  Opaque

Data
====
api-key:  32 bytes

This resulting Data should be an api-key containing 32 bytes representing the 32 characters of your API Key. If this does not line up, run `kubectl delete secret datadog-agent` and double check the command you ran.



# <h1> *** Reproduction Steps ***
</h1>

1 - Configuring Your Helm Chart Deployment
---

For this exercise, we will be using the Helm chart deplomyent method.  

Use the Datadog `values.yaml` file (found in this repo) and be sure to add `datadog-agent` as the value for `apiKeyExistingSecret`:

![image](https://user-images.githubusercontent.com/60328238/206782118-1046016c-76df-4245-957b-ff9d57432025.png)

After, make sure that log collection is enabled and that the `containerCollectAll:` value is set to `true`: 

![image](https://user-images.githubusercontent.com/60328238/206782320-12f12b44-1d01-4c72-b070-357b69feef75.png)

Save this file.

2 - Start Minikube
---
Spin up your cluster environment. In this exercise, we are using minikube. As such, run the command `minikube start` in your terminal. After a few moments, you should receive a message outlining that the minikube cluster is now ready for use:

`Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default`

3 - Add/Update Helm Chart
---

In your terminal, ensure that the you have the datadog helm repository added and up to date: 

```
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

4 - Install the Datadog Agent
---
***In the same path where you saved your `values.yaml`***, run the following command:


```
helm install <RELEASE_NAME> -f dd-agent_default.yaml datadog/datadog --set targetSystem=<TARGET_SYSTEM>
```

*Note:* Replace `<RELEASE_NAME>` and `<TARGET_SYSTEM>` with the release name of the deployment (ex: `datadog-agent`) and the name of your OS (ex: `linux` or `windows`)m respectively. 

You will know the helm chart has been successfully installed and the agent deployed by waiting a moment and running the command `kubectl get pods`, the command to see the number of pods running in your environment and their statuses, similar to below: 

![image](https://user-images.githubusercontent.com/60328238/198738442-276687d7-560c-4e7d-8c83-b59b4fdaa2a1.png)

*Note:* Two pods beginning with "datadog" should be deployed, with one being the Cluster Agent. BOTH should have a `Running` status and should have matching paired numbers under the "Ready" column. 

***GREAT! You have deployed the Node and Cluster Agents to your environment!***

5 - Deploy the Log Containers
---

Download the `loggers.yaml` file, ideally placing it in the same directory as your `dd-agent_default.yaml` file. Run the following command to deploy the applications to K8s: 

```
kubectl apply -f loggers.yaml
```
Next, run the `kubectl get pods` command and you should see a new pod appear:

![image](https://user-images.githubusercontent.com/60328238/198740903-87fcd34e-9572-4d18-8edf-6372225029d6.png)

*Note:* Do not proceed until the new pod has a `Running` status and has matching paired numbers under the "Ready" column similar to the Agent pods. 

6 - Confirm Container Logs are Appearing 
---
Check your logs page to confirm that the containers emitting logs from the `busybox.yaml` deployment. A great "tell" is looking for the logs containing the word "cookies," as one of the containers has logs that will be emitting logs with this word present: 

![image](https://user-images.githubusercontent.com/60328238/206784546-f5220e91-ac92-4e00-8070-bfd61846efff.png)

***From the yaml file:***
```
    spec:
      containers:
      # Container named "example-logger" outputting date-related logs every 5 seconds
      - name: example-logger
        image: busybox
        imagePullPolicy: Always
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 5; 
          echo `date '+%FT%T'` example stdout log; 
          echo `date '+%FT%T'` example stderr log 1>&2;
        done;"
        ]
      # Container named "yummy-logger" outputting hunger-inducing logs every 5 seconds
      - name: yummy-logger
        image: busybox
        imagePullPolicy: Always
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 5; 
          echo `date '+%FT%T'` I like cookies and milk on Thursdays; 
          echo `date '+%FT%T'` But only if I have chocolate and bonbons on Fridays;
        done;"
        ]
      # Container named "random-logger" outputting logs every 5 seconds
      - name: random-logger
        image: chentex/random-logger
        imagePullPolicy: Always
```

Note: You are also free to change the log phrases, but ensure that those logs appear on your Logs page. 

7 - Implement Container Exclusion 
---

Reviewing our documentation, you may exclude containers from the Agent Autodiscovery perimeter with an exclude rule based on their `name`, `image`, or `kube_namespace` to collect ***NO DATA*** from these containers. If a container matches an exclude rule, it is not included unless it first matches an include rule. To continue, we can also exclude certain *behaviors* from a container, and not the entire container itself. We will illustrate this now by excluding **logs from certain containers**

![image](https://user-images.githubusercontent.com/60328238/201407644-103284be-7508-4ac5-8a70-7f2ec2544c10.png)

Following the image above, we will update our values.yaml file to exclude one (1) of the logs containers from having their logs appear on the platform

```
  env:
    - name: DD_CONTAINER_EXCLUDE
      value: "image:chentex/random-logger"
```

The full values.yaml file...

```
targetSystem: "linux"
datadog:
  apiKeyExistingSecret: datadog-agent
  logLevel: DEBUG
  processAgent:
    enabled: false
  kubelet:
    tlsVerify: false
  kubeStateMetricsEnabled: false
  logs:
    enabled: true
    containerCollectAll: true
  env:
  - name: DD_CONTAINER_EXCLUDE
    value: "image:chentex/random-logger"
```

After adding the above in your helm chart, save your file and apply the changes by upgrading your helm chart:

`helm upgrade <RELEASE-NAME> -f values.yaml datadog/datadog --set targetSystem=<TARGET-SYSTEM>`

8 - Confirmation of Exclusion
---

Run `kubectl get pods` to ensure that your busybox deployment, Node, and Cluster Agents are running in your environment:

![image](https://user-images.githubusercontent.com/60328238/206787591-3f07b177-6acf-431e-b9bb-a1608d84bcaa.png)

Now, notice that in our example we are trying to exclude logs from the `chentex/random-logger` container. Let's go to our log page and filter by facet to only see `image_name:"chentex/random-logger"` logs: 

![image](https://user-images.githubusercontent.com/60328238/206786356-03ee33c2-4b0f-40ce-b5b2-7af20425ec7f.png)

We see that these particular logs stopped after a certain point. Good sign! Let's filter by another container in the `loggers` deployment file (like `service:busybox`):

![image](https://user-images.githubusercontent.com/60328238/206786793-fa4620a9-0037-4f02-abe1-c967d575e5a3.png)

We see that *these* logs are still coming, meaning we've sucessfully excluded logs from a certain container :)

To revert to receiving **all** logs again, simply comment out the exclusion configuration in your values.yaml file and redeploy the Agent. Similar steps can be taken if one would like to exclude other things from containers, such as their metrics. 

9 - Removing / Uninstalling the Agent
---

To remove the Datadog Cluster and Node Agents, you can run the following command: 

```
helm uninstall datadog-agent
```

To remove the logger container deployment:

```
kubectl delete deployments loggers 
```

Important Links
====

- K8s Installation Steps (Helm) ***Ensure Logs are enabled*** => https://docs.datadoghq.com/containers/kubernetes/installation/?tab=helm

- Container Discovery Setup => https://docs.datadoghq.com/agent/guide/autodiscovery-management/?tab=containerizedagent#inclusion-and-exclusion-behavior

- Environment Variable list (Where/How to input container images to exclude logs from) => https://docs.datadoghq.com/containers/docker/?tab=standard#environment-variables



