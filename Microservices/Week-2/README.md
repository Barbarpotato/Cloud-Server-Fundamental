# Table Of Contents
- [CRUD Operation in Flask](#flask)
- [Swagger Documentation for RESTAPI](#Swagger)

# Creating Flask REST API
1. Open a new Terminal in the lab environment.
2. Now you will write a Python code that will act as a web application serving REST API endpoint that returns the string Hello World. Run the following command to create a file named server.py and the click the button below to edit the content of the file..
```
touch /home/project/server.py
```
3. Paste the following code in server.py and save the file.
```python
from flask import Flask

app = Flask("My Hello World Application")

@app.route("/")
def hello():
    return "Hello World!"

if __name__=="__main__":
    app.run(debug=True)
# When no port is specified, starts at default port 5000
```
4. Before you run this web application, you need to install the packages that are required for the application to run. On the terminal, run the following command to install the packages.
```
python3 -m pip install flask
```
5. Once the flask package is installed, run server.py to start a server which is listening at port 5000 and serving a REST API endpoint at the root / that returns the string Hello World!.
```
python3 server.py
```

<a name="flask"></a>

# CRUD operations with Flask
In this lab, you will learn how to create a Products's list using a Flask server. Your application should allow you to add a product, retrieve the products, retrieve a specific product with its id, update a specific product with its id, and delete a product with its id. All these operations will be achieved through the REST API endpoints in your Flask server.

You will create an application with API endpoints to perform Create, Retrieve, Update, and Delete operations on the above data using a Flask server.

## Set-up : Create application
1. Run the following command to clone the Git repository that contains the starter code needed for this lab, if it doesn't already exist.
```
[ ! -d 'jmgdo-microservices' ] && git clone https://github.com/ibm-developer-skills-network/jmgdo-microservices.git
```
2. Change to the directory jmgdo-microservices/CRUD to start working on the lab.
```
cd jmgdo-microservices/CRUD
```
3. Run the following command on the terminal to install the packages that are required.
```
python3 -m pip install flask flask_cors
```

## Exercise 1: Understand the server application
1. You first need to import the packages required to create REST APIs with Flask.
```
from flask import Flask, jsonify, request
import json
```
2. You can then create the Flask application, which will service all the REST APIs for adding, retrieving, updating, and deleting products.
```
app = Flask("Product Server")
```
3. The code has precreated products added to the list. These are defined in the following code.
```python
products = [
    {'id': 143, 'name': 'Notebook', 'price': 5.49},
    {'id': 144, 'name': 'Black Marker', 'price': 1.99}
]
```
4. Now that the server is defined, you will create the REST API endpoints and define the routes or paths, one for each of the following operations.
    - Retrieve all the products - GET Request Method
    - Retrieve a product by its id - GET Request Method
    - Add a product - POST Request Method
    - Update a product by its id - PUT Request Method
    - Delete a product by its id - DELETE Request Method
    Add the following code to products.py in the space provided.
    ```python
    # Example request - http://localhost:5000/products
    @app.route('/products', methods=['GET'])
    def get_products():
        return jsonify(products)

    # Example request - http://localhost:5000/products/144 - with method GET
    @app.route('/products/<id>', methods=['GET'])
    def get_product(id):
        id = int(id)
        product = [x for x in products if x["id"] == id][0]
        return jsonify(product)

    # Example request - http://localhost:5000/products - with method POST
    @app.route('/products', methods=['POST'])
    def add_product():
        products.append(request.get_json())
        return '', 201

    # Example request - http://localhost:5000/products/144 - with method PUT
    @app.route('/products/<id>', methods=['PUT'])
    def update_product(id):
        id = int(id)
        updated_product = json.loads(request.data)
        product = [x for x in products if x["id"] == id][0]
        for key, value in updated_product.items():
            product[key] = value
        return '', 204

    # Example request - http://localhost:5000/products/144 - with method DELETE
    @app.route('/products/<id>', methods=['DELETE'])
    def remove_product(id):
        id = int(id)
        product = [x for x in products if x["id"] == id][0]
        products.remove(product)
        return '', 204
    ```

## Run and test the server with cURL
1. In the terminal, run the python server using the following command.
```
python3 products.py
```
The server starts running and listening at port 5000.
2. In the new terminal, run the following command to access the http://localhost:5000/products API endpoint. curl command stands for Client URL and is used as command line interfacing with the server serving REST API endpoints. It is, by default, a GET request.
```
curl http://localhost:5000/products
```
This returns a JSON with the products that have been preloaded.
```json
[
  {
    "id": 143,
    "name": "Notebook",
    "price": 5.49
  },
  {
    "id": 144,
    "name": "Black Marker",
    "price": 1.99
  }
]
```

4. In the terminal, run the following command to add a product to the list. This will be a POST request to which you will pass the product parameter as a JSON.
```
curl -X POST -H "Content-Type: application/json" \
    -d '{"id": 145, "name": "Pen", "price": 2.5}' \
    http://localhost:5000/products
```
5. Verify if the product is added by running the following command.
```
curl http://localhost:5000/products/145
```

<a name="Swagger"></a>

# Creating a Swagger documentation for REST API
In this lab, you will understand how to create a Swagger documentation for your REST APIs.

## Task 1: Getting your application started
1. In the terminal, clone the repository which has the Swagger documentation and the REST API code ready by pasting the following command. The repository that you clone has code that will run a REST API application which can be used to organize tasks.
```
git clone https://github.com/ibm-developer-skills-network/jmgdo-microservices.git
```
2. Change the working directory to jmgdo-microservices/swagger_example by running the following command.
```
cd jmgdo-microservices/swagger_example
```
3. Run the following commands to install the required packages.
```
python3 -m pip install flask_cors
```
4. Now start the application which serves the REST API on port number 5000.
```
python3 app.py
```
5. From the top menu, choose Launch Application and enter the port number as 5000. This will open a new browser page, which accesses the application you just ran.
6. Copy the url on the address bar.
7. From the file menu, go to jmgdo-microservices/swagger_example/swagger_config.json to view the file on the file editor.
    ![swagger.json](/images/swagger_json.png)<br>
8. In the file editor, paste the application URL that you copied where it says **<Your application URL\>** without the protocol (https://) and save the file.
9. Copy the entire content of the file swagger_config.json. You will need this copied content to generate SwaggerUI.
10. Click on this link <a href="https://editor.swagger.io/>https://editor.swagger.io/</a> to go to the Swagger Editor.
11. Paste the content you copied from swagger_config.json on the left side. You will get a prompt which says Would you like to convert your JSON into YAML? . Press Cancel to paste the content.
12. You will see that the UI is automatically populated on the right.
    ![swagger interface](/images/swagger_editor.png)<br>
13. Now you can test each of the endpoints. Four tasks have been already added for you, when the application was started. Click on the down arrow next to GET /tasks.
14. Click on Try it out. This will allow you to try your REST API endpoint.
    ![tryitout](/images/click_tryitout.png)<br>

## Task 2: Creating Swagger Documentation and Generating Server code
1. Now you will create a REST API with Swagger documentation. To start with, let’s define your application.
2. Copy and paste the following JSON in the Swagger Editor. You will get a prompt which says Would you like to convert your JSON into YAML? . Press Cancel to paste the content.
```json
{
    "swagger": "2.0",
    "info": {
      "version": "1.0",
      "title": "Our first generated REST API",
      "description": "<h2>This is a sample server code the is generated from Swagger Documenation with Swagger Editor</h2>"
    },
    "paths": {
      "/greetings": {
        "get": {
          "summary": "Returns a list of Greetings",
          "tags": ["Hello in Different Languages"],
          "description": "Returns greetings in different languages",
          "produces": [
            "application/json"
          ],
          "responses": {
            "200": {
              "description": "OK"
            }
          }
        }
      }
    }
}
```
You will see the Swagger UI automatically appearing on the right. You cannot test it yet as your application is not defined and running yet.
![generated swagger](/images/ourfirstgenerate_api.png)<br>
3. From the menu on top, click on Generate Server and select python-flask. This will automatically generate the server code as a zip file named python-flask-server-generated.zip. Download the zip file to your system
![generate servercode](/images/generate_servercode.png)<br>
4. In your lab envrionment, click on the PROJECT folder and drag and drop the zip file there.
![dragndrop](/images/dragdrop_flaskapp.png)<br>
5. On the terminal go to the /home/project directory.
```
cd /home/project
```
6. Check to see if the zip file that you just dragged and dropped, exists.
7. Unzip the contents of the zip file into a directory named python-flask-server-generated by running the following command.
```
unzip python-flask-server-generated.zip -d python-flask-server-generated/
```
8. Change to the python-flask-server folder inside the folder you just extracted the zip file into.
```
cd python-flask-server-generated/python-flask-server
```
9. The entire server setup along with endpoint is done for you already. Let's build the server code.
```
docker build . -t mynewserver
```
10. Run the docker application now by running the following command. The server generated code automatically is configured to run on port 8080.
```
docker run -dp 8080:8080 mynewserver
```
11. If you got some problem. Considering take this actions
    - First inspect the Docker images you've built by executing the following command:
    ```
    docker images
    ```
    - This command will provide a list of Docker images, with their respective IMAGE IDs. Then delete the Docker image of mynewserver by using the following command:
    ```
    docker rmi -f <IMAGE ID>
    ```
    - Open the "requirements.txt" file which present inside the unzipped folder named "python-flask-server-generated" and update the connexion version to "connexion >= 2.7.0, < 3.0.0" as shown in the screenshot below.
    ```
    connexion >= 2.7.0, < 3.0.0
    connexion[swagger-ui] >= 2.6.0
    python_dateutil == 2.6.0
    setuptools >= 21.0.0
    swagger-ui-bundle >= 0.0.2
    ```
    Once the image is deleted, proceed with rebuilding the docker image and allowing some time before running the docker application.

12. To confirm that the service is running and your REST API works, execute the following command.
```
curl localhost:8080/greetings
```
13. Now you should stop the server. For this you need the docker container id. Run the following command and copy the container id.
```
docker ps | grep mynewserver
```
14. To stop the container you need to kill the instance referring to the container id you copied in the last step.
```
docker kill <container_id>
```
15. In the file explorer go to, python-flask-server-generated/python-flask-server/swagger_server/controllers/hello_in_different_languages_controller.py. This is where you need to implement your actual response for the REST API.
![serverview](/images/view_servercode.png)<br> 
16. Replace return 'do some magic!' with the following code. As this is the python code and the indentation in Python is very important, make sure you check the indentations error.
```python
hellos = {
  "English": "hello",
  "Hindi": "namastey",
  "Spanish": "hola",
  "French": "bonjour",
  "German": "guten tag",
  "Italian": "salve",
  "Chinese": "nǐn hǎo",
  "Portuguese": "olá",
  "Arabic": "asalaam alaikum",
  "Japanese": "konnichiwa",
  "Korean": "anyoung haseyo",
  "Russian": "Zdravstvuyte"
}

return hellos
```

17. Build the docker container again to ensure the changed code is taken in.
```
docker build  . -t mynewserver
```
Run the container now with the following command. You may notice that you are using -p instead of -dp. This is to ensure the server is not running in discreet mode and you are able to see errors if any.
```
docker run -p 8080:8080 mynewserver
```
18. Now click on Launch Application and enter the port number 8080. This will open a browser window. Append the path /greetings to the URL. You should see the greetings in the page.
![output](/images/newserver_output.png)<br>
