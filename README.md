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

![Diagram](https://github.com/singhalyogesh/web-login/blob/master/documentation/images/mweb-flow.png)

But let's get down to the details.

### Initiate the user verification flow

To initiate the user verification, you need to trigger a deep link in the format mentioned below. You can initiate the user verification at multiple touch points in your user flow journey ( for exmple - login, registration, checkout, verification etc. )

```java
"truecaller://truesdk/web_verify?requestNonce=UNIQUE_REQUEST_ID&partnerKey=YOUR_APP_KEY&partnerName=YOUR_APP_NAME"
```

Here, requestNonce should be a unique requestID that you need to associate with every verification request you trigger, so as to do the requisite mapping of the access token which we post to your callback URL once the user shares his / her consent.
Add the partner key which you generated from your developer portal account, and the app name that you want users to see in the truecaller profile dialog.

Please note that in case Truecaller app is not present on the user's device, the deep link won't trigger anything. To effectively handle this case, you should use the deep link as an 'href' link, and open the link by setting the targer as 'blank'. Please refer below example -

```java
<a href="truecaller://truesdk/web_verify?requestNonce=UNIQUE_REQUEST_ID&partnerKey=YOUR_PARTNER_KEY&partnerName=YOUR_APP_NAME" target="_blank">
```

This would open the deeplink in a new window, and open the user's Truecaller profile if the app is present on the device. And in case the app is not present, then the new blank window will open. Using javascript, add an event listenner for a new window and just close the window after a certain timeout ( few milliseconds ).
 

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
 "name": {
  "first": "first_name_field_value3",
  "last": "last_name_field_value",
 },
 "aboutMe": "status_message_field_value",
 "phoneNumbers": [467601234567],
 "onlineIdentity": {
  "facebookId": "facebook_id_field_value",
  "twitterId": "twitter_id_field_value",
  "email": "mail@gmail.com",
  "url": "url_field_value"
 },
 "companyName": "w_company_name_field_value",
 "jobTitle": "w_title_field_value",
 "addresses": [{
  "countryCode": "se",
  "city": "city_field_value",
  "street": "street_field_value",
  "zipcode": "1234567"
 }],
 "gender": "Female"
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
| state [String]       | no            | This parameter will exist in json if it is sent in "ask for user's number" endpoint. | ne4_cRtzx73-alui_XDvzS5h                                          |

**Expected Response Codes:**

- 2** OK
