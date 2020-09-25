# Creating Managed Server Applications in Azure





## Exercise 1: Create Azure Kubernetes Service (AKS) cluster

#### Task 1: Open the Azure Portal

1. On the Taskbar, click the **Microsoft Edge** icon.

1. In the open browser window, navigate to the **Azure Portal** (<https://portal.azure.com>).

1. If prompted, authenticate with the user account account that has the owner role in the Azure subscription you will be using in this lab.

#### Task 2: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open a new shell instance.

   
#### Task 3: Create an AKS cluster by using Cloud Shell

1. At the **Cloud Shell** command prompt at the bottom of the portal, type in the following command and press **Enter** to create a variable which value designates the name of the resource group you will use in this task:

    ```sh
    RESOURCE_GROUP='AADesignLab0402-RG'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a variable which value designates the Azure region you will use for the deployment (replace the placeholder `<Azure region>` with the name of the Azure region to which you intend to deploy resources in this lab. `az account list-locations` will list all available locations for your subscription.):

    ```sh
    LOCATION='<Azure region>'
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new resource group:

    ```sh
    az group create --name $RESOURCE_GROUP --location $LOCATION
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to create a new AKS cluster:

    ```sh
    az aks create --resource-group $RESOURCE_GROUP --name aad0402-akscluster --node-count 1 --node-vm-size Standard_D2s_v3 --generate-ssh-keys
    ```

    > **Note**: If you receive an error message regarding availability of the VM size which value is represented by the `--node-vm-size` parameter, review the message and try other suggested VM sizes.

    > **Note**: Alternatively, in **PowerShell** on **Cloud Shell**  you can identify VM sizes available in your subscription in a given region by running the following command and reviewing the values in the **Restriction** column (make sure to replace the `region` placeholder with the name of the target region):

    ```pwsh
    Get-AzComputeResourceSku | where {$_.Locations -icontains "region"} | Where-Object {($_.ResourceType -ilike "virtualMachines")}
    ```

    > **Note**: The **Restriction** column will contain the value **NotAvailableForSubscription** for VM sizes that are not available in your subscription.

1. Wait for the deployment to complete before you proceed to the next task.

    > **Note**: This operation can take up to 10 minutes.


#### Task 4: Connect to the AKS cluster.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to retrieve the credentials to access the AKS cluster:

    ```sh
    az aks get-credentials --resource-group $RESOURCE_GROUP --name aad0402-akscluster
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify connectivity to the AKS cluster:

    ```
    kubectl get nodes
    ```

1. At the **Cloud Shell** command prompt, review the output and verify that the node is reporting the **Ready** status. Rerun the command until the correct status is shown.

> **Result**: After you complete this exercise, you should have successfully deployed a new AKS cluster.




## Exercise 2: Autoscaling pods in an AKS cluster

#### Task 1: Deploy a Kubernetes pod by using a **.yaml** file.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to download a sample containerized application:

    ```
    git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to navigate to the location of the downloaded app:

    ```sh
    cd azure-voting-app-redis
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to list the content of the application **.yaml** file:

    ```sh
    cat azure-vote-all-in-one-redis.yaml
    ```

1. Review the output of the command and verify that the pod defninition includes requests and limits in the followng format:

    ```yaml
    resources:
      requests:
        cpu: 250m
      limits:
        cpu: 500m
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to deploy the application based on the **.yaml** file:

    ```
    kubectl apply -f azure-vote-all-in-one-redis.yaml
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to to verify that a Kubernetes pod has been created:

    ```
    kubectl get pods
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to identify whether the public IP address for the containerized application has been provisioned:

    ```
    kubectl get service azure-vote-front --watch
    ```

1. Wait until the value in the **EXTERNAL-IP** column for the **azure-vote-front** entry changes from **\<pending\>** to a public IP address, then press **Ctrl-C** key combination. Note the public IP address in the **EXTERNAL-IP** column for **azure-vote-front**.

1. Start Microsoft Edge and browse to the IP address you obtained in the previous step. Verify that Microsoft Edge displays a web page with the **Azure Voting App** message.

#### Task 2: Autoscale Kubernetes pods.

1. At the **Cloud Shell** command prompt, type in the following commands and press **Enter** after each to change the current directory and download a sample containerized application:

    ```
    cd ..
    git clone https://github.com/kubernetes-incubator/metrics-server.git
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to install **Metrics Server**:

    ```
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to configure autoscaling for the **azure-vote-front** deployment:

    ```
    kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=3 --max=10
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to view the status of autoscaling:

    ```
    kubectl get hpa
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to view the pods:

    ```
    kubectl get pods
    ```

    > **Note**: Verify that the number of replicas increased to 3. If that is not the case, wait one minute and rerun the two previous steps.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the deployment:

    ```
    kubectl delete deployment azure-vote-front
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the deployment:

    ```
    kubectl delete deployment azure-vote-back
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to verify that the commands you ran in the previous steps completed successfully:

    ```
    kubectl get pods
    ```

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the AKS cluster:

    ```
    az aks delete --resource-group AADesignLab0402-RG --name aad0402-akscluster --yes --no-wait
    ```

1. Close the **Cloud Shell** pane.

> **Review**: In this exercise, you implemented autoscaling of pods in an AKS cluster



## Exercise 4: Remove lab resources

#### Task 1: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open the Cloud Shell pane.

1. At the **Cloud Shell** command prompt at the bottom of the portal, type in the following command and press **Enter** to list all resource groups you created in this lab:

    ```sh
    az group list --query "[?starts_with(name,'AADesignLab04')]".name --output tsv
    ```

1. Verify that the output contains only the resource groups you created in this lab. These groups will be deleted in the next task.

#### Task 2: Delete resource groups

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the resource groups you created in this lab

    ```sh
    az group list --query "[?starts_with(name,'AADesignLab04')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

1. Close the **Cloud Shell** prompt at the bottom of the portal.


> **Review**: In this exercise, you removed the resources used in this lab.
