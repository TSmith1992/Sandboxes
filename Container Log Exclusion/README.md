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

Install Docker Desktop for Mac
Install command line tools with brew

```
brew install minikube
brew install kubernetes-cli
brew install helm
Add the Datadog Helm Repository
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

This resulting Data should be an api-key containing 32 bytes representing the 32 characters of your API Key. If this does not line up `kubectl delete secret datadog-agent` and double check the command you ran.

