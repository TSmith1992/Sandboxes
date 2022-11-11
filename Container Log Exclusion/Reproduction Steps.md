**Reproduction Steps**

1
---

For this exercise, we will be using the Helm chart deplomyent method.  

Use the Datadog `values.yaml` file (found in this repo as `dd-agent_default.yaml`) and be sure to add your API key as the value for `apiKey`:

![image](https://user-images.githubusercontent.com/60328238/198735437-29476e19-65b6-43a3-92f3-00095e6b8da6.png)

After, make sure that log collection is enabled and that the `containerCollectAll:` value is set to `true`: 

![image](https://user-images.githubusercontent.com/60328238/198735794-140677a8-072d-4815-a995-e108449c2d35.png)

Save this file.

2
---
Spin up your cluster environment. In this exercise, we are using minikube. As such, run the command `minikube start` in your terminal. After a few moments, you should receive a message outlining that the minikube cluster is now ready for use:

`Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default`

3
---

In your terminal, ensure that the you have the datadog helm repository added and up to date: 

```
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

4
---
***In the same path where you saved your `dd-agent_default.yaml`***, run the following command:

```
helm install <RELEASE_NAME> -f dd-agent_default.yaml datadog/datadog --set targetSystem=<TARGET_SYSTEM>
```
*Note:* Replace `<RELEASE_NAME>` and `<TARGET_SYSTEM>` with the release name of the deployment (ex: `datadog-agent`) and the name of your OS (ex: `linux` or `windows`)m respectively. 

You will know the helm chart has been successfully installed and the agent deployed by waiting a moment and running the command `kubectl get pods`, the command to see the number of pods running in your environment and their statuses, similar to below: 

![image](https://user-images.githubusercontent.com/60328238/198738442-276687d7-560c-4e7d-8c83-b59b4fdaa2a1.png)

*Note:* Two pods beginning with "datadog" should be deployed, with one being the Cluster Agent. BOTH should have a `Running` status and should have matching paired numbers under the "Ready" column. 

***GREAT! You have deployed the Node and Cluster Agents to your environment!***

5
---

Download the `busybox.yaml` file, ideally placing it in the same directory as your `dd-agent_default.yaml` file. Run the following command to deploy the applications to K8s: 

```
kubectl apply -f busybox.yaml
```
Next, run the `kubectl get pods` command and you should see a new pod appear:

![image](https://user-images.githubusercontent.com/60328238/198740903-87fcd34e-9572-4d18-8edf-6372225029d6.png)

*Note:* Do not proceed until the new pod has a `Running` status and has matching paired numbers under the "Ready" column similar to the Agent pods. 

6
---
Check your logs page to confirm that the containers emitting logs from the `busybox.yaml` deployment. A great "tell" is looking for the logs containing the word "cookies," as one of the containers has logs that will be emitting logs with this word present: 

![image](https://user-images.githubusercontent.com/60328238/201404254-857a52ea-025d-478d-93bc-bce51f1ef339.png)


***From the yaml file:***
```
    spec:
      containers:
      # Container named "my-container" outputting logs every 5 seconds
      - name: my-container
        image: busybox
        imagePullPolicy: Always
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 5; 
          echo `date '+%FT%T'` example stdout log; 
          echo `date '+%FT%T'` example stderr log 1>&2;
        done;"
        ]
      # Container named "my-container" outputting logs every 5 seconds
      - name: my-container1
        image: busybox
        imagePullPolicy: Always
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 5; 
          echo `date '+%FT%T'` I like cookies and milk on Thursdays; 
          echo `date '+%FT%T'` But only if I have chocolate and bonbons on Fridays;
        done;"
        ]
      # Container named "my-container" outputting logs every 5 seconds
      - name: random-test-logging
        image: chentex/random-logger
        imagePullPolicy: Always
```
Note: You are also free to change the log phrases, but ensure that those logs appear on your Logs page. 

7
---

Reviewing our documentation, you may exclude containers from the Agent Autodiscovery perimeter with an exclude rule based on their `name`, `image`, or `kube_namespace` to collect ***NO DATA*** from these containers. If a container matches an exclude rule, it is not included unless it first matches an include rule. To continue, we can also exclude certain *behaviors* from a container, and not the entire container itself. We will illustrate this now by excluding **logs from certain containers**

![image](https://user-images.githubusercontent.com/60328238/201407644-103284be-7508-4ac5-8a70-7f2ec2544c10.png)

Following the image above, we will update our values.yaml file to exclude one (1) of the logs containers from having their logs appear on the platform

```
  env:
    - name: DD_CONTAINER_EXCLUDE
      value: "image:chentex/random-logger"
```
After adding the above in your helm chart, save your file and apply the changes via uninstalling and reinstalling your helm chart:

`helm uninstall <RELEASE-NAME>`

`helm install <RELEASE-NAME> -f dd-agent_default.yaml datadog/datadog --set targetSystem=<TARGET-SYSTEM>`

8
---

Run `kubectl get pods` to ensure that your busybox deployment, Node, and Cluster Agents are running in your environment:

![image](https://user-images.githubusercontent.com/60328238/201409584-31bce230-8f3d-4c4a-840d-27b519f28a6b.png)

Now, notice that in our example we are trying to exclude logs from the `chentex/random-logger` container. Let's go to our log page and filter by facet to only see `service:random-logger` logs: 

![image](https://user-images.githubusercontent.com/60328238/201409976-b4dbf92e-9a09-44d8-a952-f48978542cf6.png)

We see that these particular logs stopped after a certain point. Good sign! Let's filter by another container in the `busybox` deployment file (like `service:busybox`):

![image](https://user-images.githubusercontent.com/60328238/201410287-8e312088-9d38-453f-863e-c09c482f6cbd.png)

We see that *these* logs are still coming, meaning we've sucessfully excluded logs from a certain container :)

To revert to receiving **all** logs again, simply comment out the exclusion configuration in your values.yaml file and redeploy the Agent. Similar steps can be taken if one would like to exclude other things from containers, such as their metrics. 



  env:
