
This is my attempt to explain various **OAuth2 Grant Types**

## Definitions

* **Resource Owner** - User or owner of the resource 
* **Resource Server** - Application where the resource is stored
* **Authorization Server** - Application that authenticate the client
* **Client** - Requestor


## Authorization Code Grant

This grant type is meant for applications that can store **secret key** securely e.g. Web Application. Mobile Application or SPA Application should not use this as the secret key can be extracted easily from the source code.

In this grant type, there are three steps

* User Authentication
* Client Authentication
* Resource Access

### Step 1 (User Authentication)

Client send a request to the **Authorization Server** with the following url

		https://oauth2server.com/auth?response_type=code&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=photos 
		
    * **response_type** is **code** which means **Authorization Code**
    * **client_id** id of the client (this is the id that is obtained once a client is registered with the Authorization Server)
    * **redirect_uri** This is the url that is redirected by Authorization Server once authentication is done
    * **scope** This is the access level requested by client

Assuming the user is authenticated successfully and user accepts the requested access, the response will be

	    https://oauth2client.com/cb?code=AUTH_CODE_HERE

otherwise it will be

	   https://oauth2client.com/cberror=access_denied

### Step 2 (Client Authentication)

Client send a **HTTP Post** request to the **Authorization Server** with the following body (note the authorization code and client_secret) 

	    POST https://api.oauth2server.com/token
	     grant_type=authorization_code&
	     code=AUTH_CODE_HERE&
	     redirect_uri=REDIRECT_URI&
	     client_id=CLIENT_ID&
	     client_secret=CLIENT_SECRET
	     
Assuming the **client_id**, **client secret** and **code** is valid then the response will be

		{
		  "access_token": "ff16372e-38a7-4e29-88c2-1fb92897f558",
		  "token_type": "bearer",
		  "refresh_token": "f554d386-0b0a-461b-bdb2-292831cecd57",
		  "expires_in": 43199,
		  "scope": "photos"
		}

otherwise

		{
		    "error":"invalid_request"
		}

**refresh token** can be used by the client to request for new **access code** from **Authorization Server** once the old **access code** expires 

    POST https://api.oauth2server.com/token
	     grant_type=refresh_token&
	     client_id=CLIENT_ID&
	     client_secret=CLIENT_SECRET&
	     refresh_token=REFRESH_TOKEN
		
### Step 3 (Resource Access)

Client calls the **Resource Server** with the **access_token** as part of the header `Authorization: Bearer <access_token>`

## Implicit Grant

This grant type is meant for application that cannot keep secret such as SPA, Mobile Applications as it is possible to extract the secret key after decompiling the source code as such in this grant type there are only two steps (Step 2 of Authorization Grant Type is not there)

### Step 1 (User Authentication)

Client send a request to the **Authorization Server** with the following url 

		https://oauth2server.com/auth?response_type=token&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=photos 
	
notice that the difference is the **response_type** which is **token** and there is no **client_secret**

Once the **access token** is received the resource can be accessed as per step 3 in **Authorization Code Grant**


## Client Credentials Grant

Client Credentials Grant type is meant for the client itself and is not related to any specific user as such there is no need of user authentication. There are only two steps here same as **Implict Grant**

### Step 1 (Client Authentication)

Client send a HTTP Post request to the **Authorization Server** with the following url 

		POST https://api.oauth2server.com/token
    			grant_type=client_credentials&
    			client_id=CLIENT_ID&
    			client_secret=CLIENT_SECRET 
	
Once the **access token** is received the resource can be accessed as per step 3 in **Authorization Code Grant**. One thing to note that for **client credentials** there is no **refresh token** [spec](https://tools.ietf.org/html/rfc6749#section-4.4.3) 

### Password

OAuth2 has a grant type where a client can pass username and password in order to get a token. Since this requires the client to collect username and password as such this grant type should only be used for client that is an extension of the web application e.g. reddit mobile app to connect to reddit itself.

### Step 1 (User Authentication)

Client send a HTTP Post request to the **Authorization Server** with the following url 

		POST https://api.oauth2server.com/token
    			grant_type=password&
    			username=USERNAME&
    			password=PASSWORD&
    			client_id=CLIENT_ID

Once the **access token** is received the resource can be accessed as per step 3 in **Authorization Code Grant**. Note that there is no ** client secret** as the assumption here is that client that uses this grant type is a application (such as mobile application) that cannot keep secret.

## Resources

* https://aaronparecki.com/2012/07/29/2/oauth2-simplified
* http://salesforce.stackexchange.com/questions/14009/whats-the-benefit-of-the-client-secret-in-oauth2
* http://www.slideshare.net/rohitsghatol/oauth-20-simplified
* http://www.slideshare.net/aaronpk/an-introduction-to-oauth-2/31-Server_exchanges_authcode_for_an

