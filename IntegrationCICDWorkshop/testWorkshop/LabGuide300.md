Updated Sep 18 2019


# Lab 300: Understanding Our Build Script

Note: these scripts are editable and these are not necessarily the best scripts possible. The scripts can be done in any coding language, the following script is a mixture of bash and node js.

All APIs can be found in this documentation(Curl commands are API calls):https://docs.oracle.com/en/cloud/paas/integration-cloud/rest-api/index.html

## Lets break down our script

### Initial Step
 Our first lines here allow us to access our integration environment files inside of our CONFIGS folder. chmod changes the permissions and source allows us to use the variable stored in our files. the ${xyz} and $xyz notation is how you access a variable in the shell script.
```bash
chmod 755 CONFIGS/env-${OIC_SOURCE_ENV}
source ./CONFIGS/env-${OIC_SOURCE_ENV}
```
### Get Integrations
We will start with a loop that goes through all of our integrations to export. The reason we use a loop that increments by two each time is to grab the integration ID and the version number as the variable looks as follows: 

ex. GET_INTEGRATIONS=(integrationtest 01.00.0000 integrationtest2 01.01.0020)
where integrationtest is the ID of our integration and 01.00.0000 is the version.
```bash
#loop through integrations to pull
for ((integration=0; integration<${#GET_INTEGRATIONS[@]}; integration=integration+2))
do
  curl -u ${OIC_USERNAME}:${OIC_PASSWORD} -H “Content-Type:octet-stream” -X GET ${OIC_BASE_URL}/ic/api/integration/v1/integrations/${GET_INTEGRATIONS[$integration]}\|${GET_INTEGRATIONS[$integration+1]}/archive -v --output ${GET_INTEGRATIONS[$integration]}.iar -v
done
```
As you can see base on the curl command we access our parameters from our build , username, password and base url. We then make an api call to our environment to pull each integration and output the data into an iar file of the same name. Our $integration variable is our iterator in this case.

### Get Packages

Exporting packages is a full reset. This means that you don't specify what integrations and connectors to pull but pull all referenced in that package. 

First we will look at all the packages we have listed in our config file and loop through them. For each package we we will enter a directory by their name (if it exists). If it doesn't exist we will create a directory for it and then enter.
```bash
#loop through packages to export
for package in ${GET_PACKAGES}
do

  #if package does not exist in our directory, create the directory
  if [ ! -d "${package}_package" ]
  then
      mkdir ${package}_package
  fi
  #enter the repository
  cd ${package}_package
```

We first download the .par file itself which can be uploaded directly to a integration environment. The issue here is that we must maintain all of our connector data as well. As mentioned previously the API to get integrations or packages does not maintain all connector data, so an api to pull connector data must be used for all connectors in our integration asset.
```bash

  #Download the Par file for package
  curl -u ${OIC_USERNAME}:${OIC_PASSWORD} -H “Content-Type:octet-stream” -X GET ${OIC_BASE_URL}/ic/api/integration/v1/packages/${package}/archive -v --output ${package}.par 
```
We then use an api to get data about our package to get a list of integrations inside of our package. We then loop through each integration in our package and loop through each connector in our integrations to pull all of our connector data. We use node in this chunk of code in order to manipulate the json elements coming back from our connectors. This gets rid of any metadata we do not need in our connector json file as it is not necessary for uploading our connector back to a new integration environment.
```bash

  #get Package Data
  curl -u ${OIC_USERNAME}:${OIC_PASSWORD} -H “Content-Type:octet-stream” -X GET ${OIC_BASE_URL}/ic/api/integration/v1/packages/${package} -v --output ${package}.json
  
  #Loop through each integration in package
  max=$(node -pe 'JSON.parse(process.argv[1]).countOfIntegrations' "$(cat "${package}".json)")
  for ((i=0;i<max;i++));
  do
    integration=$(node -pe 'JSON.parse(process.argv[1]).integrations['${i}'].id' "$(cat "${package}".json)")
    curl -u ${OIC_USERNAME}:${OIC_PASSWORD} -H “Content-Type:octet-stream” -X GET ${OIC_BASE_URL}/ic/api/integration/v1/integrations/${integration} --output integration.json
    len=$(node -pe 'JSON.parse(process.argv[1]).dependencies.connections.length' "$(cat integration.json)")
    
    #loop through each connection for all integrations and download their Jsons
    for ((j=0;j<len;j++));
    do
        
        connector=$(node -pe 'JSON.parse(process.argv[1]).dependencies.connections['${j}'].id' "$(cat integration.json)")
        
        curl -u ${OIC_USERNAME}:${OIC_PASSWORD} -H “Content-Type:octet-stream” -X GET ${OIC_BASE_URL}/ic/api/integration/v1/connections/${connector} -v --output connection.json
        
        #Edit Jsons to be immediately configurable
        sed '5!d' connection.json > ${connector}.json
            
node<<EOF
```
```js

        const fs = require('fs');
        const http = require("https");
        const data = require('./${connector}.json')
        var auth = 'Basic ' + Buffer.from('${OIC_USERNAME}:${OIC_PASSWORD}').toString('base64');
        delete data.adapterType.type
        var dataString=JSON.stringify(data)
        fs.writeFileSync('${connector}.json',dataString)
        
        if(data.connectionProperties != undefined){
          if(data.connectionProperties[0].attachment != undefined){
            attachmentName = data.connectionProperties[0].attachment.attachmentName
            propertyName = data.connectionProperties[0].attachment.propertyName
            console.log("attach: " + attachmentName + "\nprop: " + propertyName)
            //Write to json to make sure it is able to be immediately re-uploaded
//            var dataString=JSON.stringify(data)
            console.log('${connector}.json')

        
            //API call to get attachment
            var options = {
              "method": "GET",
              "hostname":"integration-orasenatdpltintegration02.integration.ocp.oraclecloud.com",
              "port": "443",
              "path": "/ic/api/integration/v1/connections/${connector}/attachments/"+propertyName,
              "headers": {
                  "Authorization": auth,
                  "Accept": "*/*",
                  "Host": "integration-orasenatdpltintegration02.integration.ocp.oraclecloud.com:443",
              }
            };
            var req = http.request(options, function (res) {
              var chunks = [];
              res.on("data", function (chunk) {
                  chunks.push(chunk);
              });
              res.on("end", function () {
                  var body = Buffer.concat(chunks);
                  fs.writeFileSync(attachmentName,body.toString())
              });
            });
            req.end();
           }
          }
```          
```bash
EOF
    done
done
cd ..
done
```
### Get Connections
Now we loop through any connectors that are specified in the config files and pull down their necessary data. We do this separately from our full refresh so that we can modularly do CICD.
```bash
#loop through connectors to pull
for connector in ${GET_CONNECTIONS}
do
  curl -u ${OIC_USERNAME}:${OIC_PASSWORD} -H “Content-Type:octet-stream” -X GET \
  ${OIC_BASE_URL}/ic/api/integration/v1/connections/$connector -v --output connector.json
  sed '5!d' connector.json > $connector.json
   
node<<EOF

        const fs = require('fs');
        const http = require("https");
        const data = require('./${connector}.json')
        var auth = 'Basic ' + Buffer.from('${OIC_USERNAME}:${OIC_PASSWORD}').toString('base64');
        delete data.adapterType.type
        var dataString=JSON.stringify(data)
        fs.writeFileSync('$connector.json',dataString)

        if(data.connectionProperties != undefined){
          if(data.connectionProperties[0].attachment != undefined){
            attachmentName = data.connectionProperties[0].attachment.attachmentName
            propertyName = data.connectionProperties[0].attachment.propertyName
            console.log("attach: " + attachmentName + "\nprop: " + propertyName)
            //Write to json to make sure it is able to be immediately re-uploaded
            
            //API call to get attachment
            var options = {
              "method": "GET",
              "hostname":"integration-orasenatdpltintegration02.integration.ocp.oraclecloud.com",
              "port": "443",
              "path": "/ic/api/integration/v1/connections/${connector}/attachments/"+propertyName,
              "headers": {
                  "Authorization": auth,
                  "Accept": "*/*",
                  "Host": "integration-orasenatdpltintegration02.integration.ocp.oraclecloud.com:443",
              }
            };
            var req = http.request(options, function (res) {
              var chunks = [];
              res.on("data", function (chunk) {
                  chunks.push(chunk);
              });
              res.on("end", function () {
                  var body = Buffer.concat(chunks);
                  fs.writeFileSync(attachmentName,body.toString())
              });
            });
            req.end();
           }
          }   
EOF
done
```
Lastly we will pull our Process Automation projects specified in the config file and push everything we have pulled down into our current directory.
```bash

#Pull down relevant process Applications
for project in ${PROJECT_NAME}
  do
  curl -u ${OIC_USERNAME}:${OIC_PASSWORD} -H “Content-Type:octet-stream” -X GET https://integration-orasenatdpltintegration02.integration.ocp.oraclecloud.com:443/ic/api/process/v1/spaces/${GET_SPACEID}/projects/${GET_PROJECTID}/exp -v --output $project.exp
done

git add .
git commit -m wip
git push

```

This is a brief overview of what the code does. Please feel free to edit your script to make it work for your use cases this is just a simple outline that can be built off of.

For lab conclusions please see lab 400.