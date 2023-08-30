# Adobe Sign Connector module for Mendix

Adobe Acrobat Sign is a comprehensive electronic signature and digital document solution. It enables secure document sending, signing, tracking, and management, eliminating paper-based processes.

Capabilities of the Adobe Acrobat Sign API:
- Create and manage agreements
- Request electronic and conventional signatures
- Retrieve signed documents
- Embed a signing UI in your app
- Send reminders
- Create widgets
- Build reusable library documents
- Batch send documents in bulk
- Download audit trails
- Archive signed documents
- Leverage enterprise workflows in Acrobat across any device

## Adobe Acrobat Sign Connector advantage:

This Mendix connector for Adobe Acrobat Sign seamlessly integrates both platforms. It empowers Mendix apps with Adobe Sign's capabilities by providing a generic connector for the Adobe Acrobat Sign API that - due to its genericity - supports each API Operation supported by Adobe. You can visit the [Adobe Acrobat Sign API Documentation](https://secure.eu1.adobesign.com/public/docs/restapi/v6) to gain further information on its capabilities.

## Typical usage scenario

For Mendix developers seeking to enhance their Mendix applications with automated document handling and electronic signature capabilities, the Mendix connector for Adobe Sign offers a valuable solution. This connector enables integration of Adobe Sign's features, allowing users to seamlessly send, track, and collect electronic signatures within their Mendix apps. Whether for automating approval workflows, digitizing contract management, or simplifying form submissions, the Mendix connector for Adobe Sign provides the tools needed to optimize document processes and elevate user experiences.

## Features

- Authenticate users via their Adobe Sign acoount
- Access to all the features of the Adobe Sign API
- Logging for all access attempts
- CSRF protection
- Adobe Sign Token management including encryption

## Preparation

In order to use the Adobe Sign Connector you need access to the Adobe Acrobat Sign API. This is usually granted via a developer account. [See here](https://opensource.adobe.com/acrobat-sign/developer_guide/index.html) on how to get started. If you are working at a bigger company, you might have corporate access and licenses. In that case, your way to a developer account might differ from the usual one.

[Create an application](https://opensource.adobe.com/acrobat-sign/developer_guide/gstarted.html) registration in the web UI and configure OAuth for that application. 

Now, in the OAuth Settings of the application registration, set the Redirect URI to ```https://[your-mendix-application-domain.com]/adobesign/callback```. You can also configure multiple callback URIs if you want to use that particular application registration for multiple Mendix environments. Be aware though, that you can only add https-URIs and it is not possible to add ```http://localhost:8080/adobesign/callback```. [See below](#how-to-test-your-integration-locally) on how to test you integration locally anyway.

Additionally, set the scopes you want to use with this application registration. Details can be found [here](https://opensource.adobe.com/acrobat-sign/developer_guide/gstarted.html).

## Installation

1. Install the Dependencies from the Mendix Marketplace:
   - Nanoflow Commons
   - Community Commons
   - Encryption
2. Set the EncryptionKey constant of the Encryption module.
3. Install the Adobe Sign Connector from the Mendix Marketplace
4. Assign the module roles to your projects user roles. Administrator can configure the integrations, Users can use them.
5. Add the SNPT_AdobeSign_Configuration Snippet to a page you can access. Alternatively you can use the AdobeSign_Configuration page.

## Configuration

1. Open the page with the configuration snippet (see last step of installation)
2. Click on "New Authorization" and fill the creation form as follows. [See below](#how-to-find-out-your-access-points) on how to find out what you access points are.
   - web_access_point: the URI of the authorization service (e.g. ```https://secure.eu1.adobesign.com/```)
   - api_access_point: the URI of the API instance to be used (e.g. ```https://api.eu1.adobesign.com/```)
   - response_type: currently only ```code``` is supported
   - client_id: the client id of your Adobe Sign application registration
   - client_secret: the secret id of your Adobe Sign application registration
   - scopes: the scopes you want to use. Set them acoording to you Adobe Sign applications registration OAuth configuration.
3. After saving the input, you can request access tokens via the button with the "open lock" icon. This will redirect you to Adobe Sign. If you configured both the application registration in the Adobe Sign web UI and the Authorization object correctly, you will be asked to confirm the autorization. After doing this, you will be redirected to you Mendix application.
4. You might want to set the Authorization object you created as default using the "Mark as default" button. If you do so, you can use the ```RTR_AccessToken_Default``` Microflow from the Adobe Sign Connector Module to directly retrieve the current access token of the default authorization. However, is is possible to maintain and use multiple Authorizations simultaneously.

## Usage

After [preparation](#preparation), [installation](#installation) and [configuration](#configuration) you can now use the ```DELETE```, ```GET```, ```POST_FILE```, ```POST_JSON``` and ```PUT``` Microflows of the Adobe Sign Connector Module by passing the required parameters to it. The provided API request functions expect (exported) JSON strings and return HttpResponse objects that still have to be imported in order to save them in objects. When using this module **without** any extensions, you are expected to make yourself familiar with the Adobe Sign API in order to pass the correct parameters and handle the responses correctly. Particularly, what you will have to do is to create the domain model objects as well as the import and export mappings that refer to the resources you want to use.  That additional effort is the drawback of the genericity of this module. 

I currently plan to develop another Module that builds on the Adobe Sign Connector and provides the most popular API calls in a much more convenient way. That module will of course then lack the genericity of this one.
   
## How to find out your access points

To find your access points you can make a request against the Adobe Sign API. To do so, follow these steps:

1. Open the [Adobe Sign API Documentation](https://secure.adobesign.com/public/docs/restapi/v6#!/baseUris/getBaseUris)
2. Scroll to the first operation which is ```GET /baseUris```
3. Click on the ```OAUTH ACCESS-TOKEN``` Button on the right.
4. A popup will open and ask you to select the scopes for the token. Select ```user_login``` here and click on the ```Authorize``` button.
4. You will be redirected to Adobe in a new window. Just log in as you normally would and click on ```Allow Access``` afterwards.
5. The window will close with the ```Authorization``` field of the operation now bein filled. Click on ```Try it out!```.
6. There you go, your access points will be given as the response body.



## How to test your integration locally

If you want to test the integration locally, you might face the issue that Mendix locally runs on http, but you cannot add http-URIs as redirect URIs in your Adobe Sign application registrations OAuth configuration. For me, the easiest solution was to create a local NodeJS server that serves as a https proxy with a self-signed SSL certificate and redirects to your Mendix runtime.

1. Create a self signed SSL certificate. You can use openssl in your terminal for that. The command below generates a certificate ```cert.pem``` and the respective key ```key.pem```.
```bash
# non-interactive and 10 years expiration
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 3650 -nodes -subj "/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName   /CN=CommonNameOrHostname"
```
2. Create a ```server.js``` file and place the certificate and its key in a ```./ssl``` folder next to it. Not that the code below expects your Mendix runtime to be reached on ```http://localhost:8080/```.  If your Mendix runtime runs on a different port, adjust the code accordingly.
```js
let http = require("http");
let httpProxy = require("http-proxy");
let fs = require("fs");

// Create the HTTPS proxy server in front of a HTTP server

console.log("Mendix (run local) HTTPS proxy service");
console.log(
    "Listening on https://localhost:44383/ and passing requests thru to http://localhost:8080/ (Mendix runtime)"
);

httpProxy
    .createServer({
        target: { host: "localhost", port: 8080 },
        ssl: {
            key: fs.readFileSync("./ssl/key.pem", "utf8"),
            cert: fs.readFileSync("./ssl/cert.pem", "utf8"),
        },
    })
    .listen(44383);
```
3. Install the dependencies (e.g. using npm)
4. Start the server using the command ```node server.js```.
5. If you configured everything correctly and your Mendix app is running locally, you can reach it on ```https://localhost:44383/```.
6. Add ```https://localhost:44383/``` to the Redirect URIs of the Adobe Sign application registration in the Adobe Sign web UI.
