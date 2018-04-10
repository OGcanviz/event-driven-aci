# Even Driven ACI

### Components

1. web-server: This is the host for the dashboard and the api. The dashboard will let you add work to the queue. When work is added a yellow pending container will show up. This is a default place holder until the actual ACI instance comes up and adds it's InProgress state to the DB to be read. Once the container has reached InProgress, the UI will chagne the default pending to Blue with the container's name in place. You can also hit the "More Details" Button to open the model with that containers ID. Currently the modal is calling to the metrics api endpoint for that specific container. 

2. go-worker: This is the container to be spawned by the azure function watching the service bus queue. Its takes the enviorment variables: MESSAGE: The message from the queue, CONTAINER_NAME: container group name, DATABASE_URI: CosmosDb mongo connection string. *IMPORTANT:* You must remove the ssl=true url param from the connection string or it will break. When the container is started, it will add a state entry to the DB with its name and InProgress. It then waits a set number of seconds and Adds Done as its state to the DB. 

3. Spawner: This will watch the service bus queue and spawner ACI instances. The plan is to retire this and create an Azure Function instead. 

### Sales Pitch

#### Do YOU have things to be computed?!?

#### Do YOU wish they were computed?!?

#### Are YOU tired of having to deal with VMs to get those things computed?!?

Well, here at Serverless-Buzzwordland we have just the solution. Just add your string based data to a queue and our 
helpful little watcher will automagicly spin up container based compute for each of your tasks.

But Serverless Buzzworldland, why do I care? 

Well Jimmy, let me tell you

1. This serverless based architecture takes seconds to start up.

2. It will only run while your task is being computed then shut down.

3. And best of all **YOU PAY PER SECOND**, thats like 60 times less than a minute!


#### Deployment Steps

1. Clone the repo.
   ```console
   git clone https://github.com/OGcanviz/event-driven-aci.git

   cd event-driven-aci
   ```

2. Create resource group.
   ```console
    #create resource group
    az group create -l westus -n <resource group name>
   ```

2. Create service principal.
   ```console
    az ad sp create-for-rbac -n <resource group name> --role contributor
    ```
    Output sample:
    ```
    #return value as following
    #{
    #  "appId": "fb7c4111-2144-4489-8fd9-XXXXXXXXX",
    #  "displayName": "msazure-aciaks-demo",
    #  "name": "http://msazure-aciaks-demo",
    #  "password": "0fa91eda-261e-47ad-bb65-XXXXXXXX",
    #  "tenant": "3dad2b09-9e66-4eb8-9bef-XXXXXXX"
    #}
    ```

3. Update the **azuredeploy.parameters.json** in the folder **arm** with the service principal credential created above

4. Deploy the Azure resources with the ARM template.
   ```console
    cd arm
    
    #deploy resource group
    az group deployment create --template-file azuredeploy.json --parameters @azuredeploy.parameters.json
    ```
    >Note: The output **fqdn** is the URL of the ACI dashboard.

4. Download NPM packages.
   ```console
    cd ../spawner-functions

    npm install

5. Compress the files inside the **spawner-functions** folder as a .zip file.

6. Deploy the .zip file to the Azure function.
   ```console
    az functionapp deployment source config-zip  -g <resource group name> -n <app_name> --src <zip_file>
   ```
