+++
date = "2016-10-12T10:31:18-07:00"
title = "Configuring Facebook Connector in Dex"
description = "This blog post describes setting up Facebook connector in Dex"
draft = true
tags = [
  "go",
  "golang",
  "templates",
  "themes",
  "development",
]
categories = [
  "Development",
  "golang",
]
image = "/img/Configuring-Facebook-Connector-in-Dex/social.jpg"

+++

## Dex

`dex` is a federated identity management solution. It provides OpenID Connect and OAuth 2.0 to end users. It can act as a proxy for multiple identity providers as well can act as a local identity provider. Connecting multiple identity providers is achieved through connectors. It has connectors for well-known identity providers such as GitHub and Bitbucket. But connector for well known social platform Facebook was missing . I wanted to implement connector for facebook and contribute back to the community.I have implemented and sent a pull request. It was recently merged into the project .In rest of the blog, i am going to describe on how to configure and test dex facebook connector.

## Pre requists

* Setup `dex` with sample app using master branch.
Please follow the [getting started guide](https://github.com/coreos/dex/blob/master/Documentation/getting-started.md) to setup `dex`. At the end, you will have

        * dex-worker running on [http://localhost:5556]
        * dex-overlord running on [https://localhost:5557]
        * a sample app running on [http://localhost:5555]

* A valid Facebook account

## Registering Facebook App

To configure the facebook connector for `dex` we need the following information.
 
* clientID: a `string`. The Facebook App ID.
* clientSecret: a `string`. The Facebook App Secret.

In this section, I am going to guide you get above details from Facebook

#### Step 1 : Logging into to Facebook Developer Portal

Goto [https://developers.facebook.com/](https://developers.facebook.com/) and log in using your credentials. If you have already created a Facebook developer account by agreeing to the Facebook terms and conditions, you are good to follow the step 2. Otherwise, you need to follow [facebook apps registering guide](https://developers.facebook.com/docs/apps/register) to register the Facebook app. Especially you need to follow step 2 in above guide.

#### Step 2 : Creating a Facebook App

After successfully login to the Facebook developer portal now we are ready to create a Facebook app. Click on **My Apps** on the top right corner of the developer portal header then there wil be a pop over window with list of apps already created and link to **Create a New App**. Click on **Create a New App** link to create new Facebook app then there will be a popover window to create new app id. Fill the Display name , contact email and category and then click **Create App Id**  button. You need to fill a captcha before proceeding to create the app. 

#### Step 3: Getting ClientID and Client Secret

After successfully creating the app you will be landing on a dashboard for an app that you just created. Click on **Dashboard** tab in right side of the dashboard window. Now you can view app id and app secret that we needed for testing Facebook login with dex. The app secret is hidden by default you need to click on **show** button to see the app secret. You may need to enter your Facebook credential after clicking the show button. Now we have

* clientID: App ID.
* clientSecret: App Secret.

Even though we have the required details to configure dex facebook connector we need to do some more configuration in facebook developer portal to continue.

#### Step 4 : Configuring App Domain and Platform for Facebook App

On the left menu click **Settings** to go to app settings page. Here we need to add the app domain and platform. App domain is the hostname of dex worker . Here we are running dex worker on localhost .Enter the `localhost` in app domain field and press enter . Click **Add Platform** then there will be a popover window to select a platform. Select website as the platform and there will be a new panel called **Website**. In the website panel, enter dex URL in **Site URL** field.Finally, click save button on the right bottom to save all the changes.

#### Step 5 : Enable OAuth2 Login for Facebook App

`dex` Facebook connector uses OAuth2 protocol to authenticate users. We need to enable OAuth 2 setting in Facebook App.Click **Add Product** link from left panel then you will be presented with the list of products available.From the list select **Facebook Login** product.Configure following items in the window.

   * `Client OAuth Login` should be set to `Yes`. 
   * `Web OAuth Login` should be set to `Yes`. 
   * `Valid OAuth redirect URIs` should be set to in following format.

```
$ISSUER_URL/auth/$CONNECTOR_ID/callback
```

For example runnning a connector with ID `"facebook"` and an issuer URL of `"http://localhost:5556/"` the redirect would be.

```
http://localhost:5556/auth/facebook/callback
```

Now we are ready to test the Facebook login functionality.

## Configuring Facebook Connector

Make sure the dex worker and dex overlord are up and running. We need to create a JSON connector config and use dexctl command line tool to apply the configuration.

Here's an example of a `facebook` connector configuration; the clientID and clientSecret should be replaced by App ID and App Secret values from Step 3.

```
    {
        "type": "facebook",
        "id": "facebook",
        "clientID": "$DEX_FACEBOOK_CLIENT_ID",
        "clientSecret": "$DEX_FACEBOOK_CLIENT_SECRET"
    }
``` 
Use the following command to apply the config.
```
cat << EOF > /tmp/dex_connectors.json
[
	{
		"type": "local",
		"id": "local"
	},
	{
         "type": "facebook",
         "id": "facebook",
         "clientID": "$DEX_FACEBOOK_CLIENT_ID",
         "clientSecret": "$DEX_FACEBOOK_CLIENT_SECRET"	
       } 
]
EOF
./bin/dexctl --db-url=$DEX_DB_URL set-connector-configs /tmp/dex_connectors.json
```
## Testing Facebook Login in Sample App

Open [http://localhost:5555](http://localhost:5555) in a browser. Click `register`and  choose`facebook`.Enter your facebook credentials and authorize dex to retrieve your email address. After successful authentication, you should be able to view your claims returned by dex. It means you are now successfully logged into sample app through your Facebook account!!!

