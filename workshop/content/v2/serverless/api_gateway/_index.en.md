+++
title = "Enable API Gateway Authentication"
weight = 35
+++

Amazon API Gateway can use the JSON Web tokens (JWT) returned by Cognito User Pools to authenticate API calls. In this step, you'll configure an authorizer for your API to use the user pool you created in module 1.

Since Cognito User Pools implements [OpenID](https://en.wikipedia.org/wiki/OpenID_Connect) Connect JSON web tokens, API Gateway is able to compare the signature of an access or identity token against the known public keys of the Cognito User Pool which allows verification and authentication to happen without having to write additional code in your application.

### High-Level Instructions

In the Amazon API Gateway console, create a new Cognito user pool authorizer for your API. Configure it to use the user pool that you created in the previous module. You can test the configuration in the console by copying and pasting the identity token printed to the console after you log in via the `/signin` path of your current website. Once setup, you will change your application's code to send the proper JSON web token with its API requests to authenticate.

{{% expand "Step-by-step instructions (expand for details)" %}}

1. In the AWS Management Console choose ***Services*** then select ***API Gateway*** under Networking and Content Delivery.

1. Choose the API named **WildRydes**.

1. Under your newly created API, choose ***Authorization***.

1. Then click ***Manage authorizers*** tab.

1. Click ***Create***.

    ![Authorizers](../../images/apigatewayv2-authorizer-settings.png)


1. Confirm ***JWT*** is selected for ***Authorizer Type***

1. Enter `WildRydes` for the Authorizer name.

1. Confirm ***Identity Source*** is ***$request.header.Authorization***

1. Enter Issuer URL as `https://cognito-idp.us-east-1.amazonaws.com/userPoolID` for Issuer URL. If you are working in another region, be sure to update the URL with the correct region. 

1. Click Add Audience and in the audience box paste the ***Client Id*** for the App Client. 

    ![Authorizer final](../../images/apigatewayv2-create-user-pool-authorizer.png)
    
1. Click ***Save and Attach***

1. On the main Authorization page, click ***Deploy***

#### Configure your Wild Rydes web app to authenticate API requests

Now that you've deployed the new authorizer configuration to production, all API requests must be authenticated to be processed.

1. Return to your Wild Rydes app, sign in at **/signin** if necessary, and attempt to request a ride.

1. You should receive an Error finding unicorn. If you open the developer console, you will see that we received a HTTP 401 error, which means it was an unauthorized request. To authenticate our requests properly, we need to send an Authorization header.

    > If you at first still that you requests go through without any errors, try requesting a ride again in 30-60 seconds to allow the API Gateway changes to fully propagate.

1. Go back to Cloud9 and open the **/website/src/pages/MainApp.js** files.

1. Browse down to the **getData** method you previously updated. You will notice that the headers for the request currently include a blank **Authorization** header.

1. Replace your current **getData** method with the following code which sends your user's Cognito identity token, encoded as a JSON web token, in the **Authorization** header with every request.

    ```javascript
    async getData(pin) {
        const apiRequest = {
          body: {
            PickupLocation: {
              Longitude: pin.longitude,
              Latitude: pin.latitude
            }
          },
          headers: {
            'Authorization': this.state.idToken,
            'Content-Type': 'application/json'
          }
        };
        console.log('API Request:', apiRequest);
        return await API.post(apiName, apiPath, apiRequest);
    }
    ```

1. Allow the application to refresh, sign-in again, and request a ride.

1. The unicorn ride request should be fulfilled as before now. To see the full request headers which were sent, look at the developer console for an **API Request** informational message which includes the API Request details once expanded, including the full headers and body of the request.

    ![API Request Details](../../images/cognito-authorizer-request-console-log.png)
    
{{% /expand %}}

---

If the API now invokes correctly and application funcions as expected summoning unicorns, you may proceed to complete either:

Optional module 2 extension with Fine-grained IAM-based authorization with API Gateway

OR

To proceed to the module 3 without completing the optional module extension, choose **3. AWS integration with IAM-based AuthZ** on the left side menu.