<a  href="https://www.twilio.com">
<img  src="./public/convos-title.png"  alt="Twilio"  width="250"  />
</a>

# Twilio Webchat React App + Serverless

_Twilio Webchat React App + Serverless_ is an extension of the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app) application that demonstrates how you can build a website chat widget using [Twilio Conversations](https://www.twilio.com/docs/conversations) powered with a backend hosted on [Twilio Functions & Assets](https://www.twilio.com/docs/serverless/functions-assets).

The frontend application is identical to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app), with the main difference being which endpoint gets called from the UI (serverless functions; located in the `serverless` folder).

**TLDR;** the original [server](https://github.com/twilio/twilio-webchat-react-app/tree/main/server) folder has been replaced by the [serverless](./serverless/) folder, providing a deployable option for endpoints and static site hosting.

---

1. [Getting started](#Getting-started)
    1. [Setup](#Setup)
    2. [Working locally](#working-locally)
2. [Features](#Features)
3. [Project structure](#Project-structure)
    1. [React app](#react-app)
    2. [Serverless functions](#serverless-functions)
4. [Deployment Steps](#deployment-steps)
5. [Browser support](#Browser-support)
6. [Accessibility](#Accessibility)
7. [FAQs](#faqs)

---

# Getting Started

## Setup

### Install Dependencies

1. [Install the yarn package manager](https://classic.yarnpkg.com/lang/en/docs/install/), which is needed before performing the next steps.

1. **Install React App Dependencies** - run the following command at the root of the repository:

    ```
    yarn
    ```

1. **Install Serverless Dependencies** - navigate to the `/serverless` directory:

    ```
    cd serverless && yarn
    ```

### Populate `.env` Files

There are two `.env` to populate; one at the root of the repository and one in the `./serverless` directory. We provide a handy `bootstrap` script to set up the environment variables required for you.

```shell
yarn bootstrap \
accountSid=YOUR_ACCOUNT_SID \
authToken=YOUR_AUTH_TOKEN \
apiKey=YOUR_API_KEY_SID \
apiSecret=YOUR_API_SECRET \
addressSid=YOUR_ADDRESS_SID \
conversationsServiceSid=YOUR_CONVERSATIONS_SERVICE_SID
```

You can find your **Account Sid** and **Auth Token** on the main [Twilio Console page](https://console.twilio.com/).

For more info on how to create an **API key** and an **API secret**, please check the [documentation](https://www.twilio.com/docs/glossary/what-is-an-api-key#how-can-i-create-api-keys).

You can find your **Conversations Service Sid** on the [services page](https://console.twilio.com/us1/develop/conversations/manage/services?frameUrl=%2Fconsole%2Fconversations%2Fservices%3Fx-target-region%3Dus1). Make sure to pick the one linked to your Flex Account — usually it is named `Flex Chat Service` and it starts with `IS`

For the **Address Sid**, first ensure you have a valid chat address configured as described [here](https://www.twilio.com/docs/flex/admin-guide/setup/conversations/manage-conversations-chat-addresses). Click on the edit button of your chat address on the [messaging conversations addresses page](https://console.twilio.com/us1/develop/flex/manage/messaging/conversations) and the edit screen will contain Address Sid. Note this Sid starts with `IG`.

The environment variables associated with enabling and configuring customer transcripts can be found in the `.env.default` file, as well as the `.env` file after running the bootstrap script. Their use is covered [in the Twilio Webchat React readme](https://github.com/twilio/twilio-webchat-react-app/#chat-transcripts).

## Working Locally

While developing locally, you will need to run the backend and frontend servers at the same time, which can both be started from the root directory with the following commands.

### Start the Local Backend Server

The following command will run the serverless functions locally:

```shell
yarn server
```

Your functions will be served at `http://localhost:3003/`.

### Start the Local React App Server

The following will run the React App locally:

```shell
yarn start
```

Your app will be served at `http://localhost:3002/`.

# Features

For a feature overview of the application, please refer to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app) README.

# Project Structure

This repository includes two packages:

1. A React app
2. Serverless functions for securely invoking the Twilio Webchat APIs

## React App

For an overview of the React app, please refer to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app) README.

### Configuration

For an overview of all configuration options, please refer to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app#configuration) README.

This version has an additional optional configuration parameter, `sessionData`. If populated, properties within will be included in the `pre_engagement_data` of the conversation. This example adds two properties: `userId` and `entryPoint`.

```javascript
window.addEventListener("DOMContentLoaded", () => {
    Twilio.initWebchat({
        sessionData: {
            userId: "123",
            entryPoint: "view-product-456"
        }
    });
});
```

## Serverless functions

As mentioned before, Twilio Webchat App requires a backend to hit in order to work correctly.
This backend functionality — found in the `serverless` folder — exposes two main endpoints:

### InitWebchat

This first endpoints, hit by the application when the pre-engagement form is submitted, takes care of a few things:

1. Contacts Twilio Webchats endpoint to create a conversation and get a `conversationSid` and a participant `identity`
2. Creates a token with the correct grants for the provided participant identity
3. (optional) Programmatically send a message in behalf of the user with their query and then a welcome message

#### **A note about the pre-engagement form data**

-   By default, this endpoint takes the `friendlyName` field of the form and uses it to set the customer User's name via the webchat orchestration endpoint.
-   In addition to that, all the fields (including `friendlyName`) will be saved as the conversation `attributes`, under the `pre_engagement_data` key. You can find additional information on the Conversation object [here](https://www.twilio.com/docs/conversations/api/conversation-resource#conversation-properties).

### RefreshToken

This second endpoint is in charge of refreshing a token that is about to expire. If the token is invalid or already expired, it will fail.

# Deployment Steps

In order to use a deployed version of this widget you will need to follow these three steps:

1. Build and compile minimized React App code.
2. Build and deploy the serverless function endpoints
3. (Optional) Update your website template.

The first two steps can be performed automatically via the included GitHub Actions workflow. This workflow requires configuring the following repository secrets, using the values from the Getting Started section above:

-   TWILIO_API_KEY
-   TWILIO_API_SECRET
-   ADDRESS_SID
-   CONVERSATIONS_SERVICE_SID

You may also follow the steps below to manually deploy.

## Build and compile minimized React App code

The first step is to compile a build of the Webchat React App, which will eventually be hosted via Twilio Assets. This is important to do first, as the app needs to be compiled and copied to the serverless `assets` directory in order to be deployed. The following command will build the React app and place the minimized build in the `assets` folder of our Twilio Serverless function:

```shell
yarn build
```

## Build and deploy the serverless function endpoints

The next step is to build and deploy the serverless functions and assets.

Since Typescript was used in development, the `.ts` files are compiled to `.js` files and copied to a `dist/` folder which are then used for deployment.

1. To build the serverless functions, run the following from the root directory:

    ```
    yarn build-server
    ```

2. Next, deploy the functions with the following:
    ```
    yarn deploy-server
    ```

After successful deployment, copy the `domain` of the deployed functions, which will be needed in the next step.

## Update Your Website Template (optional)

Once the bundle is uploaded, you can have it loaded in your website page, as per one of the two examples below:

### Embedded JavaScript (recommended)

First, add the following script tag, replacing `[serverless domain]` with the same domain output from the deployment steps above:

```html
<script defer src="https://[serverless domain]/static/js/main.js"></script>
```

Next, declare the root element that the webchat widget will be rendered into:

```html
<div id="twilio-webchat-widget-root"></div>
```

Finally, add the code to initialize the webchat app as per following example.

For more information about the available options, please check the [Configuration section](#configuration).

```html
<script>
    window.addEventListener("DOMContentLoaded", () => {
        Twilio.initWebchat({
            theme: {
                isLight: true
            }
        });
    });
</script>
```

### Embedded iframe

Add the following, replacing `[serverless domain]` with the same domain output from the deployment steps above:

```html
<iframe src="https://[serverless domain]/index.html"></iframe>
```

# Browser Support

For Browser Support, please refer to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app) README.

# Accessibility

For Accessibility, please refer to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app) README..

# FAQs

### As a developer, how do I clear an ongoing chat?

Open your browser console, run `localStorage.clear()` and refresh the page to start anew.
Alternatively, you can simply wrap up/complete the corresponding task as an agent from your Flex UI instance.

### Can I use npm?

Currently there is a known issue with installing dependencies for this project using npm. We are investigating this and will publish a fix as soon as possible. We recommend using `yarn` instead.

# License

MIT © Twilio Inc.
