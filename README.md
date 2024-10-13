= RestTestCases

RestTestCases is a simple and extensible tool to test a REST API.
Its focus is to define the test cases of a REST API in a declarative way, for example using JSON or YAML, reducing verbosity and making test definitions easier to understand.

This version is developed in Java, allowing integration with applications in this language using JUnit 5. However, its core concept also supports implementation in other technologies.

== Motivation

The main motivation was to have a testing tool that allows for creating REST API test cases more clearly.

Creating tests with RestAssured in Java, requires understanding the RestAssured tool and its functions. The response must be validated programmatically, which can be laborious when the service returns a very extensive JSON.

*Example with RestAssured*:

[source,java]
----
    public void testGetUserByQueryParam() {
    
        RestAssured.baseURI = "http://api.example.com";

        Response response = given()
                                .param("name", "John Doe")  
                                .param("age", 30)           
                                .when()
                                .get("/users")
                                .then()
                                .extract().response();

       
        response.then().statusCode(200);

        response.then().body("name", hasItem("John Doe"));
        response.then().body("age", hasItem(30));
        response.then().body("birthdate", hasItem("15/05/1980"));
        response.then().body("country", hasItem("Canada"));
    }
----

The idea was to create a way to write tests in a declarative manner, where the programmer only needs to specify the request and the expected response. Using the RestTestCases, we have an easily understandable YAML file.

In the example below, we define a test that executes a request and validates some data in the response. The key advantage is that we can validate the entire response more easily than if we were to do it programmatically. retorno completo mais facilmente que se fizessemos de forma programática.

*Example with RestTestCases*:

[source,yaml]
----
testGetUserById:
    request:
         GET: http://api.example.com/users/
         queryParams:
             name: John Doe
            age: 30
         response:
            status: 200
            bodyJson: !
                [
                    {    
                        "id":1;
                        "name":"John Doe",
                        "age": 30,
                        "birthdate":"15/05/1980",
                        "country":"Canada"
                    }
                ]
----



== Functionality


With RestTestCases, tests are defined in a YAML or JSON file format. The tool converts this JSON tree into a tree of NodeDef components, which executes a small step of the testing process. The default components are configured in the internal config.yaml file of the library (io.github.resttestcases.config.yaml), but the tool can be extended to add new functionalities, such as preparing the application by loading a specific precondition of your system.

----
testGetUserById:                                # starts a test
    request:                                    # prepares the request for RestAssured
         GET: http://api.example.com/users/     # configures the method and URL
         queryParams:                          # configures the parameters
            name: John Doe
            age: 30
         response:                              # handles the response
            status: 200                    # checks the status code
            body: !                             # checks if the body matches the expected
                [
                    {   
                        "id":1,
                        "name":"John Doe",
                        "age": 30
                    }
                ]
----


In the template below, we have all the default declarations of the tool with some notes:

[source,yaml]
----                   
test<id optional>:                             #Required. Define a test or a test step; each file may contain multiple tests.
    name:<Optional Test Name>                  #Optional. Test Name
    description:<description>,                 #Optional. Test description
    
    request<id optional>:                      #Optional. Define a request; a test can have multiple requests as long as different IDs are assigned to each of them.
        (GET|POST|PUT|DELETE): <path> (*)      #Requerid. Define the request method (e.g., GET, POST) and the request path (e.g., /api/v1/resources)
                                               #Optional. Below are the defined parameters of the request.
        contentType: <value>                   
        accept: <value>
        headers:
            <paramNameN>: <paramValueN>
        pathParams: 
            <paramNameN>: <paramValueN>
        queryParams: 
            <paramNameN>: <paramValueN>
        formParams:
            <paramNameN>: <paramValueN>
        multiPart:
            <paramNameN>: <paramValueN>
        multiPartFiles:
            <paramNameN>: <path do arquivo do resource>
        body: <conteudo>

        response: (*)                           #Optional, verify the response
            status: <http status code>,     #Optional, verify the status code
            headers:                            #Optional, verify response headers
                <paramNameN>: <paramValueN>                
            body: < body>                       #Optional, verify the body of the response in text format
            bodyJson: <json body>               #Optional, verify the body of the response in json format, more details below."
            
            
----


*The jsonBody definition*

The jsonBody definition allows for more semantic validation of the JSON compared to a simple string comparison.

The next example, define a standard JSON. The request body must contain the same properties with the same values; the list must contain the same items, but the order of the properties and lists may vary.
 
[source,json]
----
{
    "name":"John",
    "department":{
        "id":"1",
        "name":"Sales"
    },
    "roles":[
        {
           "nome":"product",
           "valor":"write" 
        },
        {
           "nome":"customer",
           "valor":"read" 
        }
    ]
}
----

The next example includes handling for some special cases:

[source,json]
----
{
    "id":"?"                        #Wildcard "?" accept any value. Requires the id field, but it can have any value.
    "name":"John",
    "department":{
        "id":"1",
        "name":"Sales"
        "..."                       #Wildcard "..." accepts any items or fields undeclared. Ignore additional properties
    },
    "roles":[
        "!inOrder;length:18",       #Wildcard "!inOrder force list in order, wildcard "!length" verify length array  
                                    #Requires the items to be in the same order and list length is 18
        {
           "name":"product",
           "value":"write" 
        },
        {
           "name":"customer",
           "value":"read" 
        }
        "..."                       #Ignore additional itens, in example:  "roles":["..."] checks only if the field is a array 
    ]
}
----

The wildcard "..." allows checking, for example, the first and last item of a list:


[source,json]
----
[
    "!inOrder"   #Wildcard "!inOrder" force list in order
    "first"
    "..."
    "tenth"
    "..."
    "last"
]
----


== How to use

The current version was developed for use in Java projects and has dependencies on JUnit and RestAssured. To execute your tests with RestTestCases, you should follow these steps:

* *1.* Add maven dependency:

[source,xml]
----
    <dependency>
        <groupId>io.github.resttestcases</groupId>
        <artifactId>rest-test-cases-java</artifactId>
        <version>1.0.0</version>
        <scope>test</scope>
    </dependency>
----

* *2.* Create Tests Cases in yalm files.

----
testGetUserById:                                # starts a test
    request:                                    # prepares the request for RestAssured
         GET: http://api.example.com/users/     # configures the method and URL
         queryParams:                          # configures the parameters
            name: John Doe
            age: 30
         response:                              # handles the response
            status: 200                    # checks the status code
            body: !                             # checks if the body matches the expected
                [
                    {   
                        "id":1,
                        "name":"John Doe",
                        "age": 30
                    }
                ]
----


* *3.* Create file resttestcases.conf.yaml to config RestTestCases

[source,yalm]
----
#RestTestCases config file

#Tests URL Base 
baseURI: http://localhost:8080

----

* *4.* Create JunitTestSuite using TestFactory

[source,java]
----
import io.github.resttestcases.core.junit.DynamicTestFactory;

public class TestSuite {

    @TestFactory
    public Iterable<? extends DynamicNode> load() throws IOException {
        return DynamicTestFactory.createDynamicTestNodes(new File("src/test/resources/rest-test-cases"), new Properties());
    }
}
----

* *6.* When executing *TestSuite*, all tests defined in *src/test/resources/yaml-tests* will be run


== Extending the tool

The tool is based on converting each JSON/YAML node into a component of type NodeStatement, which implements *io.github.resttestcases.core.statements.NodeStatement* that defines the following methods:

[source,java]
----
    /**
     * Configures the component based on JSON node representation 
     * @param path
     * @param parent
     * @param node
     */
    void config(NodePath path, NodeStatement parent, JsonNode jsonNode);
    
    /**
     * Valide JSON node 
     * @param path
     * @param node
     */
    void valide(NodePath path, JsonNode jsonNode);
    
    
    /**
     * Execute the function associated with this definition
     * @param context
     */
    void execute(Context context);

----


Each component is capable of being configured and is responsible for initiating the configuration of its children.
The Factory class assists in this creation and configuration and uses the files io.github.resttestcases.config.yaml and resttestcases.conf.yaml files to configure its components.
To create your component, you should follow these steps:


* *1.* Create a new NodeStatement:

[source,java]
----
public class SystemDateStatement extends NodeStatement {

    @Override
    public void execute(Context context) {
        //set new date to database
    }
}        
----          

* *2.* Add in resttestcases.conf.yaml: 
  

[source,yaml]
----
#RestTestCases config file

#Tests URL Base 
baseURI: http://localhost:8080

statements : 
    -   name: systemDate
        className: my.project.test.def.SystemDateStatement
        parents:
        - io.github.resttestcases.statements.TestStatement
----

* *3.* use in test cases: 
  

[source,yaml]
----
testGetUserById: 
    systemDate: 2020-01-01 00:00:00                           
    request:                                    
         GET: http://api.example.com/users/     
         queryParams:                          
            name: John Doe
            age: 30
         response:                            
            status: 200
----

For better understanding, please check the project's source code.
