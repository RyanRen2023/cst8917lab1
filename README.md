


 az account list-locations 

 "Canada Central"

 az group create --name AzureFunctionsCst8917lab1-rg --location  "Canada Central"

{
  "id": "/subscriptions/cc6178d9-393a-40b4-8355-d26737d0be44/resourceGroups/AzureFunctionsCst8917lab1-rg",
  "location": "canadacentral",
  "managedBy": null,
  "name": "AzureFunctionsCst8917lab1-rg",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}


az storage account create --name cst8917storageaccoutn --location "Canada Central" --resource-group AzureFunctionsCst8917lab1-rg --sku Standard_LRS


az functionapp create --resource-group AzureFunctionsCst8917lab1-rg --consumption-plan-location canadacentral --runtime python --runtime-version 3.12 --functions-version 4 --name cst8917lab1-azure --os-type linux --storage-account cst8917storageaccoutn


step 2:

# Variable block
let "randomIdentifier=cst8917lab1"
location="Canada Central"
resourceGroup="AzureFunctionsCst8917lab1-rg"
tag="create-and-configure-database"
server="msdocs-azuresql-server-$randomIdentifier"
database="msdocsazuresqldb$randomIdentifier"
login="azureuser"
password="Pa$$w0rD-$randomIdentifier"
# Specify appropriate IP address values for your environment
# to limit access to the SQL Database server
startIp=0.0.0.0
endIp=0.0.0.0

echo "Using resource group $resourceGroup with login: $login, password: $password..."


echo "Creating $server in $location..."
az sql server create --name $server --resource-group $resourceGroup --location "$location" --admin-user azure --admin-password Password12@3!

echo "Configuring firewall..."
az sql server firewall-rule create --resource-group $resourceGroup --server $server -n AllowYourIp --start-ip-address $startIp --end-ip-address $endIp


echo "Creating $database in serverless tier"
az sql db create --resource-group $resourceGroup --server $server --name $database --sample-name AdventureWorksLT --edition GeneralPurpose --compute-model Serverless --family Gen5 --capacity 2


1. create resource group `cst8917lab1-rg`
2. create Functions in Azure with stroageaccount
3. config my local environment
4. Azure: Sign In
5. `Functions:create new project`
6. Press F1 to open the command palette, then search for and run the command Azure `Functions: Download Remote Settings....`
7. set up queue
8. Press F5 to Run the fuction
9. Navigate to Azure extention , in workspace, choose local project to Test Azure function
10. in to storage explorer to check data
11. Deploy Function to Azure

step 2
1. create a sql server with database
2. set up database network to public access
3. set up `SqlConnectionString` in vs code comes from database connection string
4. initial database table 
5. download remote setting again.

SqlConnectionString

Server=tcp:cst8917lab1dbserver.database.windows.net,1433;Initial Catalog=mySampleDatabase;Persist Security Info=False;User ID=azureuser;Password=Password123!;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;





