#Reproduction Steps

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
