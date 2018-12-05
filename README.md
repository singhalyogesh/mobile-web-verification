# Truecaller SDK verification for Mobile Web

## Preconditions

- You have a mobile web application that allow users to sign up /sign in / verify using mobile number to use your services
- You have identified the benefits that Truecaller can provide you, by allowing your users one click sign up / verification / checkout using their already verified Truecaller profile
- You have set up a callback endpoint on your server, that we will use to post the access token, once the user has approved your app to use their Truecaller verified profile
- This documentation to efficiently pass through all the steps of the process.

## Get started

### Already a Truecaller Developer?

- Login to your account to create a new app. Add your App Name and the Callback URL, where we'll post your access token. Make sure you follow the [guidelines for the Callback URL](#guidelines-for-the-callback-url).
- Once we've got the info about your account and your app, we will provide you with a unique "appKey" for that application. You'll use this key in the header, for us to be able to authorize your requests.

### New to Truecaller Devs?

To ensure the authenticity of the interactions between your app and Truecaller, you need to sign up for an account and add the information about your application.

- Add your email, password, and personal info to create the account. These will be your credentials to login to your Truecaller Developer account.
- Now add your Application. Insert your App Name, your App Domain and the Callback URL, where we'll post the access tokens. Make sure you follow the [guidelines for the Callback URL](#guidelines-for-the-callback-url).

Once we've got the info about your account and your app, we will provide you with a unique "appKey" for that application.

## So how can Truecaller users verify quickly on your mobile web app?

This is how it looks in a glance:

![Diagram](https://github.com/singhalyogesh/mobile-web-verification/blob/master/documentation/images/mweb%20doc%20flow%20(1).png)

But let's get down to the details.

### Initiate the user verification flow

To initiate the user verification, you need to trigger a deep link in the format mentioned below. You can initiate the user verification at multiple touch points in your user flow journey ( for exmple - login, registration, checkout, verification etc. )

```java
"truecallersdk://truesdk/web_verify?requestNonce=UNIQUE_REQUEST_ID&partnerKey=YOUR_APP_KEY&partnerName=YOUR_APP_NAME&lang=LANGUAGE_LOCALE&title=TITLE_STRING_OPTION"
```

Here, requestNonce should be a unique requestID that you need to associate with every verification request you trigger, so as to do the requisite mapping of the access token which we post to your callback URL once the user shares his / her consent. Please note that the minimun length og the request ID parameter should be 8 characters.
Add the partner key which you generated from your developer portal account, and the app name that you want users to see in the truecaller profile dialog.
The language locale needs to be the locale string corresponding to the language that you wish the user to see the profile dialog in ( For example - 'en' for English ). Currently supported languages include -

     - English
     - Hindi
     - Bhojpuri
     - Rajasthani
     - Haryanvi
     - Marathi
     - Telugu
     - Malayalam
     - Gujarati
     - Punjabi
     - Tamil
     - Bengali
     - Kannada
     - Odia
     - Assamese
     

Similarly, the title string option can be any one of the following parameters depending on what contextual title string you want to show to the user -

     - logIn
     - signUp
     - signIn
     - verify
     - register
     - getStarted
     
Please note that in case Truecaller app is not present on the user's device, the deep link won't trigger anything. To effectively handle this case, you should use the deep link in a new window. Please refer below example -

```java

var wnd = window.open("truecallersdk://truesdk/web_verify?requestNonce=UNIQUE_REQUEST_ID&partnerKey=YOUR_PARTNER_KEY&partnerName=YOUR_APP_NAME&lang=LANGUAGE_LOCALE&title=TITLE_STRING_OPTION");

setTimeout(function(){
  if(wnd.location.href === 'about:blank'){
     // Truecaller app not present on the device and you redirect the user 
     // to your alternate verification page
     wnd.close();
 }else{
   // Truecaller app present on the device and the profile overlay opens
   // The user clicks on verify and you'll receive the user's access token to fetch the profile on 
   // your callback URL - post which, you can refresh the session at your frontend and complete the user verification
 }
},200)
```

This would open the deeplink in a new window, and open the user's Truecaller profile if the app is present on the device. And in case the app is not present, then the new blank window will stay. Using javascript timeout function, you can immediately close the window after a certain timeout ( say 200 milliseconds, so that JS can appropriately detect in case the overlay opened or the blank window, by using the code snippet as above ) and redirect the user to your alternate OTP flow.
 

### Fetch User Profile

Once the user approves the sign up to your app with their Truecaller profile, we'll immediately post the accessToken and the requestID to your Callback endpoint. The sample response format would look like below -

```java
{"requestId":"RL8YZ41FQMt5Jiak2sc_Ys0OgQA=","accessToken":"a1asX--8_yw-OF--E6Gj_DPyKelJIGUUeYB9U9MJhyeu4hOCbrl","endpoint":"https://profile4-noneu.truecaller.com/v1/default"}
```

To be able to fetch the user's profile information you should use the following endpoint.

**Endpoint:**  
Use the endpoint that you receive in the above response ( once the user shares his truecaller profile ) to fetch the user profile.

**Header Authorization Parameters:**  

| **Parameter [Type]** | **Required** | **Description**  | **Example**                             |
| -------------------  | ------------ | ---------------- | --------------------------------------- |
| Authorization        | yes          | Bearer {token}   | Bearer WcB~aSJYbCr5yla5z0CdAGfyj3Rruk~8 |

**Get User Profile**  
```bash
curl -X GET -H "Authorization: Bearer a3sAB0KnGANg4VZwIXfhUyFmPbzoONofl4FjIItac0JQSODp6niW8oBr33uOI-u7" -H "Cache-Control: no-cache" "https://profile4-noneu.truecaller.com/v1/default"
```

**Sample User Profile Response**

```json
{
  "phoneNumbers": ["919999999999"],
  "addresses": [
    {
       "countryCode": "in",
       "city": "city_field_value",
       "street": "street_field_value",
       "zipcode": "1234567"
    }
  ], 
  "avatarUrl": "https://s3-eu-west-1.amazonaws.com/images1.truecaller.com/myview/1/15a999e9806gh73834c87aaa0498020d/3", 
  "aboutMe":"About me",
  "jobTitle": "CEO", 
  "companyName": "ABC",  
  "history": {
    "name": 
    {
      "updateTime": "1508089888000"
    }
  }, 
  "isActive": "True", 
  "gender": "Male", 
  "createdTime": "1379314068000", 
  "onlineIdentities": {
    "url": "https://www.truecaller.com", 
    "email": "y.s@truecaller.com",
    "facebookId":"105056625245",
  }, 
  "type": "Personal", 
  "id": "655574719", 
  "userId":"1319413476",
  "badges": ["verified", "premium"], 
  "name": {
    "last": "Kapoor", 
    "first": "Rajat"
  }
}
```

**Response Codes**

- 200 OK
- 401 Unauthorized - **If your credentials are not valid**
- 5xx Server error - **Any other error**

Please note, in case the user doesn't shares his truecaller profile ( user dismissed the profile dialog by pressing the back button ), you'll receive a user reject error response on your callback endpoint. The sample format for the same would look as below -

```java
{"requestId":"WZqlS6PqY0ycO3mKlEuI=","status":"user_rejected"}
```

## Guidelines for the Callback URL

The Callback URL should correspond to an endpoint where we will post the access token for you to fetch the user's profile. Every access token can be used to fetch the profile only of the related user granting the authorization to your app. The access token has a time-to-live (10 minutes) and if not used within the TTL, the user needs to re-trigger the authorization process from the beginning.

When setting up the service, please consider the following:

**Method:**  
All access token requests will be submitted as POST request. Make sure your service is async, since we only expect that the message is accepted from your side. The service should respond within maximum 3 seconds upon receiving the request.

**Security:**  
To ensure security and privacy, HTTPS should be used. Make sure your certificate is always valid.

**Request Params:**

| **Param [String]**   | **Mandatory** | **Description**                                                                      | **Example value**                                                 |
| -------------------- | ------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| requestId [String]   | yes           |                                                                                      | vXbyFPwqiCAHZyxAldA9M9DDXKk=                                      |
| accessToken [String] | yes           |                                                                                      | a3sAB0KnGANg4VZwIXfhUyFmPbzoONofl4FjIItac0JQSODp6niW8oBr33uOI-u7  |

**Expected Response Codes:**

- 2** OK
