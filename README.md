README
==============================================================================
pre-requisites

1. Download WSO2 Micro-gateway from     for installation.
2. Download WSO2 Micro-gateway toolkit from  and follow    for installation.
3. Install ballerina from   . For this demo ballerina - 0.991.0 version is used.
4. Create a private docker registry if you want to push the docker image in ops process.(optional)
5. Install Docker, Kubernetes and kubectl.

Deploying micro-services in Kubernetes

1. After installing ballerina, run following commands for each service to build and create kubernetes resources. You can find the two ballerina services () in micro-services folder. 

ballerina build books_get_service.bal

This command would create the relevant kubernetes resources and push the docker image to your specified docker registry. Once this command is executed, it would provide you a link to the kubernetes resources as below. Copy that command and run it.


Similarly, execute the same commands for books_search_service.bal
ballerina build books_search_service.bal

2. Now you can execute below command, and it will display the kubernetes services created.
kubectl get svc


Updating API-definitions (swagger file) with Open-API vendor extension values

  Under api-definitions folder, you can find  ' booklistAPI.yaml' which contains an OpenAPI 3 - swagger definition with book-list resource defined. Please update the endpoint url with your IP and port accordingly. 

3.Try out developer workflow 
  3.1 Create a micro-gateway project called bookstore with following command. This would create a project structure as below.

  micro-gw init bookstore 

  ├── api_definitions
├── conf
│   └── deployment-config.toml
├── extensions
│   ├── extension_filter.bal
│   ├── startup_extension.bal
│   └── token_revocation_extension.bal
├── interceptors
├── policies.yaml
├── services
│   ├── authorize_endpoint.bal
│   ├── revoke_endpoint.bal
│   ├── token_endpoint.bal
│   └── user_info_endpoint.bal
└── target
    └── gen

  3.2  Copy the updated swagger definition from  step 3 to your newly created micro-gw project bookstore/api-definitions folder.

  3.3 Run below command to build the artifacts to expose your API. This would create an executable (.balx) file which you can use as input to the micro-gw runtime. 

      micro-gw build  bookstore

 3.3 Mount the created API to WSO2 micro-gw docker image and expose the API by running below command. If you need to use the micro-gw binary please follow [1] to understand the flow. Replace project_target_path with your bookstore/target folder.
 
 docker run -d -v <project_target_path>:/home/exec/ -p 9095:9095 -p 9090:9090 -e project="bookstore"  wso2/wso2micro-gw:3.0.1

 If you execute a docker ps command, you can find a micro-gw runtime running.

 3.4 After the APIs are exposed via WSO2 API Microgateway, you can invoke the API with a valid JWT token or an opaque access token. In order to use JWT tokens, WSO2 API Microgateway should be presented with a JWT signed by a trusted OAuth2 service.For this demo you could set the token by running below command through the terminal.

 TOKEN=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5UQXhabU14TkRNeVpEZzNNVFUxWkdNME16RXpPREpoWldJNE5ETmxaRFUxT0dGa05qRmlNUSJ9.eyJhdWQiOiJodHRwOlwvXC9vcmcud3NvMi5hcGltZ3RcL2dhdGV3YXkiLCJzdWIiOiJhZG1pbiIsImFwcGxpY2F0aW9uIjp7ImlkIjoyLCJuYW1lIjoiSldUX0FQUCIsInRpZXIiOiJVbmxpbWl0ZWQiLCJvd25lciI6ImFkbWluIn0sInNjb3BlIjoiYW1fYXBwbGljYXRpb25fc2NvcGUgZGVmYXVsdCIsImlzcyI6Imh0dHBzOlwvXC9sb2NhbGhvc3Q6OTQ0M1wvb2F1dGgyXC90b2tlbiIsImtleXR5cGUiOiJQUk9EVUNUSU9OIiwic3Vic2NyaWJlZEFQSXMiOltdLCJjb25zdW1lcktleSI6Ilg5TGJ1bm9oODNLcDhLUFAxbFNfcXF5QnRjY2EiLCJleHAiOjM3MDMzOTIzNTMsImlhdCI6MTU1NTkwODcwNjk2MSwianRpIjoiMjI0MTMxYzQtM2Q2MS00MjZkLTgyNzktOWYyYzg5MWI4MmEzIn0=.b_0E0ohoWpmX5C-M1fSYTkT9X4FN--_n7-bEdhC3YoEEk6v8So6gVsTe3gxC0VjdkwVyNPSFX6FFvJavsUvzTkq528mserS3ch-TFLYiquuzeaKAPrnsFMh0Hop6CFMOOiYGInWKSKPgI-VOBtKb1pJLEa3HvIxT-69X9CyAkwajJVssmo0rvn95IJLoiNiqzH8r7PRRgV_iu305WAT3cymtejVWH9dhaXqENwu879EVNFF9udMRlG4l57qa2AaeyrEguAyVtibAsO0Hd-DFy5MW14S6XSkZsis8aHHYBlcBhpy2RqcP51xRog12zOb-WcROy6uvhuCsv-hje_41WQ==

3.4 Test the API locally with below command. 

curl -X GET "https://IP:9095/bookstore/v1/books/list" -k -H "Authorization:Bearer $TOKEN"

3.5 Similarly, you could add the second resource 'book_search' where we will add a response interceptor to send a custom response back. Update the swagger definition you copied to bookstore/api-definitions with the content of git repository api-definitions/'fullbooklist.yaml'.

3.6 Add interceptors/interceptor.bal to booklist/interceptors folder.Build the bookstore project to create the new artifacts with below command.

micro-gw build bookstore

3.7 Stop the running micro-gw docker runtime with following commands. docker ps command would list the container id.

docker stop CONTAINER_ID

docker remove CONTAINER_ID

3.8 Mount the new API artifacts to micro-gw runtime docker image and run it.
docker run -d -v <project_target_path>:/home/exec/ -p 9095:9095 -p 9090:9090 -e project="bookstore"  wso2/wso2micro-gw:3.0.1

3.9 Now you can test it as below.
curl -X GET "https://IP:9095/bookstore/v1/books/search/Java" -k -H "Authorization:Bearer $TOKEN"

Use below command to  get the customer response from the  response interceptor.
curl -X GET "https://IP:9095/bookstore/v1/books/search/123" -k -H "Authorization:Bearer $TOKEN"

In a potential CI process you could push this project to a source repository such as git and integrate a cotinuous build with CI server such as Jenkins.

4. Try out operations workflow

4.1 You could take the bookstore micro-gw project that was created in developer workflow.

4.2. Create a new directory called 'dev' inside bookstore/conf folder of your project.

4.3 Copy dep.toml file from git project's deployment-configs folder and add it to bookstore/conf/dev folder. 

4.4. Make sure to set appropriate values to below properties in your dep.toml file.
image = 'IMAGE_NAME'
ballerinaConf = 'PATH_TO_MGW_RUNTIME/conf/micro-gw.conf'

4.5 Now you can build the bookstore project by pointing to the new deployment config file , which would create your kubernetes resources. 

micro-gw build bookstore/ -d bookstore/conf/dev/dep.toml

4.6 This would provide you a link to deploy your kubernetes resources. Please run that command.

4.7 Do a kubectl 'get svc' command and get the relevant node ports and you can try invoking the resources in kubernetes. 

Sample command
curl -X GET "https://IP:NODE_PORT/bookstore/v1/books/list" -k -H "Authorization:Bearer $TOKEN"

In a potential CD process you can auto push the docker image to a docker registry.You can enhance this dev-ops flow by creating separate deployment.toml files for each environment such as test,staging,production etc. 



