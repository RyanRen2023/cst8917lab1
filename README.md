# CST8917 Lab 1 - Azure Functions with Queue and SQL Database

This project demonstrates the implementation of two Azure Functions with different output bindings:
1. HTTP Trigger with Queue Storage binding - Accepts a name and adds it to a queue
2. HTTP Trigger with SQL Database binding - Creates ToDo items in a database

## Setup Instructions

### 1. Azure Resources Setup on Cloud

#### Resource Group, Function App, and Storage Account
1. Create a resource group: `cst8917lab1-rg`
2. Create an Azure Function App (this will automatically create a Storage Account):
   - Runtime stack: Python
   - Version: 3.12
   - Operating System: Windows
   - Plan type: Consumption (Serverless)
   - A new storage account will be created automatically

![Create Function on Azure](images/01-create%20functuon%20on%20auzre.png)

![Set up Function Storage](images/02-set%20up%20function%20storage.png)

3. Review and create the function:

![Function Review 1](images/03-Create%20Function%20Review%201.png)
![Function Review 2](images/03-Create%20Function%20Review%202.png)
![Function Review 3](images/03-Create%20Function%20Review%203.png)

#### SQL Database Setup
1. Create an Azure SQL Server:
   - Server name: cst8917lab1dbserver
   - Authentication: SQL authentication
   - Admin username: azureuser
   - Admin password: [Your secure password]

![Create Database on Azure](images/04-create%20database%20on%20azure.png)

2. Create a database named `mySampleDatabase`

3. Configure SQL Server firewall:
   - Allow Azure services: Yes

![Setup DB Networking](images/05-setup%20db%20networking.png)

4. Create the ToDo table using SQL query:
   ```sql
   CREATE TABLE dbo.ToDo (
       Id NVARCHAR(450) PRIMARY KEY,
       title NVARCHAR(MAX),
       completed BIT,
       url NVARCHAR(MAX)
   );
   ```

![Create Table on DB](images/06-create%20table%20on%20db.png)

### 2. Local Development Setup

#### Prerequisites
- Python 3.12
- Visual Studio Code
- Azure Functions Core Tools

#### Configuration Steps

1. Using Azure Function: create a project to create a new Azure function project. 

2. Using "Azure Functions: Download Remote Settings..." to download remote settings from azure

3. Register binding extensions

4. Implement Functions

#### 4.1 Queue Output Binding

1. Add the output binding decorator
```python
# Important: Queue output binding decorator
@app.queue_output(arg_name="msg", queue_name="outqueue", connection="AzureWebJobsStorage")
```

2. Implement the function code
```python 
import azure.functions as func
import logging

# Initialize function app with anonymous auth
app = func.FunctionApp(http_auth_level=func.AuthLevel.ANONYMOUS)

# Important: Route and Queue binding decorators
@app.route(route="HttpExample")  # HTTP trigger route
@app.queue_output(arg_name="msg", queue_name="outqueue", connection="AzureWebJobsStorage")  # Queue output binding
def HttpExample(req: func.HttpRequest, msg: func.Out [func.QueueMessage]) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    # Get name from query parameter or request body
    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        # Important: Write message to queue
        msg.set(name)  # This line sends the message to the queue
        return func.HttpResponse(f"Hello, {name}. This HTTP triggered function executed successfully.")
    else:
        return func.HttpResponse(
             "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.",
             status_code=200
        )
```

3. Run the function locally (Using F5)

4. Test and check data in Azure Storage Explorer

#### 4.2 SQL Database Output Binding

1. Add the output binding decorator
```python
# Important: SQL Database output binding decorator
@app.generic_output_binding(
    arg_name="toDoItems",
    type="sql",
    CommandText="dbo.ToDo",
    ConnectionStringSetting="SqlConnectionString",
    data_type=DataType.STRING
)
```

2. Implement the function code
```python 
import azure.functions as func
import logging
from azure.functions.decorators.core import DataType
import uuid

app = func.FunctionApp()

@app.function_name(name="HttpTrigger1")
@app.route(route="hello", auth_level=func.AuthLevel.ANONYMOUS)
@app.generic_output_binding(
    arg_name="toDoItems",
    type="sql",
    CommandText="dbo.ToDo",
    ConnectionStringSetting="SqlConnectionString",
    data_type=DataType.STRING
)
def test_function(req: func.HttpRequest, toDoItems: func.Out[func.SqlRow]) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')
    
    # Get name from request body
    name = req.get_json().get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        # Important: Create and insert ToDo item
        toDoItems.set(func.SqlRow({
            "Id": str(uuid.uuid4()),
            "title": name,
            "completed": False,
            "url": ""
        }))
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
            "Please pass a name on the query string or in the request body",
            status_code=400
        )
```

3. Run the function locally (Using F5)

4. Test and verify data in SQL Database

![Test Function on VSCode](images/07-test%20function%20on%20vscode.png)

## Demo Video

[Demo Video](https://youtu.be/Xts_jhCJbmY)

The demo video covers:
1. Function Demonstrations
   - Queue Storage Function
   - SQL Database Function
2. Code Walkthrough

## Project Structure
- `function_app.py`: Contains both function implementations
- `requirements.txt`: Python dependencies
- `host.json`: Function app configuration
- `local.settings.json`: Local development settings






