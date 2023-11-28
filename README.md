# SAP UI5 Demo Mock Server Configuration

This system is the so-called back-end system that we will now simulate with an SAPUI5 feature called mock server. It serves local files, but it simulates a back-end system more realistically than just loading the local data. We will also change the model instantiation part so that the model is configured in the descriptor and instantiated automatically by SAPUI5. This way, we do not need to take care of the model instantiation in the code.

### Folder Structure

```
📂 webapp
  📂 controller
  📂 i18n
  📂 localservice
    📂 mockdata
      📄 Invoices.json
    📄 metadata.xml
    📄 mockserver.js
  📂 test
    📄 initMockServer.js
    📄 mockServer.html
  📂 view
  📄 index.html
  📄 Component.js
  📄 manifest.json
```

The folder structure of our app project is clearly separating test and productive files after this step. The new test folder now contains a new HTML page mockServer.html which will launch our application in test mode without calling the real service.

The new localService folder contains a metadata.xml service description file for OData, the mockserver.js file that simulates a real service with local data, and the mockdata subfolder that contains the local test data (Invoices.json).

### Code Explaination

Refer to [/webapp/test/mockServer.html](https://github.com/VaibhavMojidra/SAP-UI5---Demo-Mock-Server-Configuration/blob/master/webapp/test/mockServer.html "mockServer.html")

We copy the index.html to a separate file in the webapp/test folder and name it mockServer.html. We will now use this file to run our app in test mode with mock data loaded from a JSON file. Test pages should not be placed in the application root folder but in a subfolder called test to clearly separate productive and test coding.

From this point on, you have two different entry pages: One for the real “connected” app (index.html) and one for local testing (mockServer.html). You can freely decide if you want to do the next steps on the real service data or on the local data within the app.

We modify the mockServer.html file and change the page title to distinguish it from the productive start page. In the bootstrap, the data-sap-ui-resourceroots property is also changed. The namespace now points to the folder above ("../"), because the mockServer.html file is now in a subfolder of the webapp folder. Instead of loading the app component directly, we now call a script initMockServer.js.

Refer to [/webapp/test/initMockServer.js](https://github.com/VaibhavMojidra/SAP-UI5---Demo-Mock-Server-Configuration/blob/master/webapp/test/initMockServer.js "initMockServer.js")

The first dependency is a file called mockserver.js that will be located in the localService folder later.

The mockserver depencency that we are about to implement is our local test server. Its init method is immediately called before we load the component. This way we can catch all requests that would go to the "real" service and process them locally by our test server when launching the app with the mockServer.html file. The component itself does not "know" that it will now run in test mode.

Refer to [/webapp/localservice/metadata.xml](https://github.com/VaibhavMojidra/SAP-UI5---Demo-Mock-Server-Configuration/blob/master/webapp/localservice/metadata.xml "metadata.xml")

The metadata file contains information about the service interface and does not need to be written manually. It can be accessed directly from the “real” service by calling the service URL and adding $metadata at the end (e.g. in our case http://services.odata.org/V2/Northwind/Northwind.svc/$metadata). The mock server will read this file to simulate the real OData service, and will return the results from our local source files in the proper format so that it can be consumed by the app (either in XML or in JSON format).

For simplicity, we have removed all content from the original Northwind OData metadata document that we do not need in our scenario. We have also added the status field to the metadata since it is not available in the real Northwind service.

Refer to [/webapp/localservice/mockserver.js](https://github.com/VaibhavMojidra/SAP-UI5---Demo-Mock-Server-Configuration/blob/master/webapp/localservice/mockserver.js "mockserver.js")

Now that we have added the OData service description file metadata.xml file, we can write the code to initialize the mock server which will then simulate any OData request to the real Northwind server.

We load the standard SAPUI5 MockServer module as a dependency and create a helper object that defines an init method to start the server. This method is called before the component initialization in the mockServer.html file above. The init method creates a MockServer instance with the same URL as the real service calls.

The URL in the rootUri configuration parameter has to point to the same URL as defined in the uri property of the data source in the manifest.json descriptor file. In the manifest.json, UI5 automatically interprets a relative URL as being relative to the application namespace. In the JavaScript code, you can ensure this by using the sap.ui.require.toUrl method. The sap/ui/core/util/MockServer then catches every request to the real service and returns a response. Next, we set two global configuration settings that tell the server to respond automatically and introduce a delay of 500 ms to imitate a typical server response time. Otherwise, we would have to call the respond method on the MockServer manually to simulate the call.

To simulate a service, we can simply call the simulate method on the MockServer instance with the path to our newly created metadata.xml. This will read the test data from our local file system and set up the URL patterns that will mimic the real service.

The oUrlParams object is used to retrieve the value of the serverDelay parameter from the query string. The get method is called on the oUrlParams object to retrieve the value of the serverDelay parameter. If the serverDelay parameter is not present in the query string, the get method returns null. If Null than 500 ms delay.

Finally, we call start on oMockServer. From this point, each request to the URL pattern rootUri will be processed by the MockServer. If you switch from the index.html file to the mockServer.html file in the browser, you can now see that the test data is displayed from the local sources again, but with a short delay. The delay can be specified with the URI parameter serverDelay.

This approach is perfect for local testing, even without any network connection. This way your development does not depend on the availability of a remote server, i.e. to run your tests.

Try calling the app with the index.html file and the mockServer.html file to see the difference.

---

[![Vaibhav Mojidra - 1.jpeg](https://raw.githubusercontent.com/VaibhavMojidra/SAP-UI5---Demo-Mock-Server-Configuration/master/screenshot/1.jpeg "Vaibhav Mojidra")](https://vaibhavmojidra.github.io/site/)