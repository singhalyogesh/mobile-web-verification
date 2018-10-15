# Truecaller SDK verification for Mobile Web

## Preconditions

- You have a mobile web application that allow users to sign up /sign in / verify using mobile number to use your services
- You have identified the benefits that Truecaller can provide you, by allowing your users one click sign up / verification / checkout using their already verified Truecaller profile
- You have set up a callback endpoint on your server, that we will use to post the access token, once the user has approved your app to use their Truecaller verified profile
- This documentation to efficiently pass through all the steps of the process.

## Get started

### Already a Truecaller Developer?

- Login to your account to create a new app. Add your App Name and the Callback URL, where we'll post your access token. Make sure you follow the [guidelines for the Callback URL](#guidelines-for-the-callbackurl).
- Once we've got the info about your account and your app, we will provide you with a unique "appKey" for that application. You'll use this key in the header, for us to be able to authorize your requests.

### New to Truecaller Devs?

To ensure the authenticity of the interactions between your app and Truecaller, you need to sign up for an account and add the information about your application.

- Add your email, password, and personal info to create the account. These will be your credentials to login to your Truecaller Developer account.
- Now add your Application. Insert your App Name, your App Domain and the Callback URL, where we'll post the access tokens. Make sure you follow the [guidelines for the Callback URL](#guidelines-for-the-callbackurl).

Once we've got the info about your account and your app, we will provide you with a unique "appKey" for that application.

## So how can Truecaller users verify quickly on your mobile web app?

This is how it looks in a glance:

![Diagram](https://github.com/singhalyogesh/web-login/blob/master/documentation/images/mweb-flow.png)

But let's get down to the details.

### Initiate the user verification flow

To initiate the user verification, you need to trigger a deep link in the format mentioned below. You can initiate the user verification at multiple touch points in your user flow journey ( for exmple - login, registration, checkout, verification etc. )

"truecaller://truesdk/web_verify?requestNonce=UNIQUE_REQUEST_ID&partnerKey=YOUR_APP_KEY&partnerName=YOUR_APP_NAME"

Here, requestNonce should be a unique requestID that you need to associate with every verification request you trigger, so as to do the requisite mapping of the access token which we post to your callback URL once the user shares his / her consent.
Add the partner key which you generated from your developer portal account, and the app name that you want users to see in the truecaller profile dialog.

### Fetch User Profile

Once the user approves the sign up to your app with their Truecaller profile, we'll immediately post the accessToken and the requestID to your Callback endpoint. To be able to fetch the user's profile information you should use the following endpoint.

**Endpoint:**  
https://profile4.truecaller.com/v1/default

**Header Authorization Parameters:**  

| **Parameter [Type]** | **Required** | **Description**  | **Example**                             |
| -------------------  | ------------ | ---------------- | --------------------------------------- |
| Authorization        | yes          | Bearer {token}   | Bearer WcB~aSJYbCr5yla5z0CdAGfyj3Rruk~8 |

**Get User Profile**  
```bash
curl -X GET -H "Authorization: Bearer a3sAB0KnGANg4VZwIXfhUyFmPbzoONofl4FjIItac0JQSODp6niW8oBr33uOI-u7" -H "Cache-Control: no-cache" "https://profile4.truecaller.com/v1/default"
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
