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
    1. [React App](#react-app)
    2. [Local backend server](#local-backend-server)
4. [Working in production](#working-in-production)
5. [Browser support](#Browser-support)
6. [Accessibility](#Accessibility)
7. [FAQs](#faqs)

---

# Getting Started

## Setup

### Install Dependencies

1. **Install React App Dependencies** - run the following command at the root of the repository:

    ```
    yarn
    ```

2. **Install Serverless Dependencies** - navigate to the `/serverless` directory:

    ```
    cd serverless && yarn
    ```

### Populate `.env` Files

There are two `.env` to populate; one at the root of the respository and one in the `./serverless` directory.

1. Run the following to make a copy of the `.env.sample` file at the root, then populate the values:

    ```
    cp .env.sample .env
    ```

    ```
    ACCOUNT_SID=xxxxxxxxxxxxxxxx
    AUTH_TOKEN=xxxxxxxxxxxxxxxx
    API_KEY=xxxxxxxxxxxxxxxx
    API_SECRET=xxxxxxxxxxxxxxxx
    ADDRESS_SID=xxxxxxxxxxxxxxxx
    CONVERSATIONS_SERVICE_SID=xxxxxxxxxxxxxxxx

    # ALLOWED_ORIGINS should be a comma-separated list of origins
    ALLOWED_ORIGINS=http://localhost:3000

    ## base endpoint of your local server. By default it is "http://localhost:3003"
    REACT_APP_LOCAL_SERVER_URL=http://localhost:3003

    # Used to enable/disable downloading/emailing of chat transcripts for the customer. Enable by setting the variable to true.
    DOWNLOAD_TRANSCRIPT_ENABLED=false
    EMAIL_TRANSCRIPT_ENABLED=false

    # The environment variable below are optional and should only be filled if enabling email chat transcripts.
    SENDGRID_API_KEY=xxxxxxxxxxxxxxxx
    FROM_EMAIL=xxxxxxxxxxxxxxxx
    ```

2. Change directories to the `./serverless` directory and generate an `.env` file from the sample, then populate the values:

    ```
    cp .env.sample .env
    ```

    ```
    ACCOUNT_SID=xxxxxxxxxxxxxxxxx
    AUTH_TOKEN=xxxxxxxxxxxxxxxxx
    ADDRESS_SID=xxxxxxxxxxxxxxxxx
    API_KEY=xxxxxxxxxxxxxxxxx
    API_SECRET=xxxxxxxxxxxxxxxxx
    CONVERSATIONS_SERVICE_SID=xxxxxxxxxxxxxxxxx
    ```

You can find your **Account Sid** and **Auth Token** on the main [Twilio Console page](https://console.twilio.com/).

For more info on how to create an **API key** and an **API secret**, please check the [documentation](https://www.twilio.com/docs/glossary/what-is-an-api-key#how-can-i-create-api-keys).

You can find your **Conversations Service Sid** on the [services page](https://console.twilio.com/us1/develop/conversations/manage/services?frameUrl=%2Fconsole%2Fconversations%2Fservices%3Fx-target-region%3Dus1). Make sure to pick the one linked to your Flex Account — usually it is named `Flex Chat Service` and it starts with `IS`

For the Address Sid, click on the edit button of your address and the edit screen will contain Address Sid. Note this Sid starts with `IG`.

The environment variables associated with enabling and configuring customer transcripts can be found in the `.env.sample` file and their use will be covered [here](#chat-transcripts).

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

Twilio Webchat React App is an open source repository that includes:

1. A React App
2. A local backend server

## React App

For an overview of the React app, please refer to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app) README.

## Local Backend Server

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

1. Build and deploy the serverless function endpoints
2. Build and compile minimised React App code.
3. (Optional) Update your website template.

## Build and deploy the serverless function endpoints

The first step is to build and deploy the endpoints. This is important to do first since we need the endpoint URL of our function to update the React App code with before we build that.

Since Typescript was used in development, the `.ts` files are compiled to `.js` files and copied to a `dist/` folder which are then used for deployment.

1. To build the serverless functions, run the following from the root directory:

    ```
    yarn build-server
    ```

2. Next, deploy the functions with the following:
    ```
    yarn deploy-server
    ```

After successful deployment, copy the `URL` of the deployed function, which will be needed in the next step.

## Build and compile minimised React App code.

The next step is to compile a build of the Webchat React App, which will eventually be hosted via Twilio Assets.

1. Take the output `URL` of the deployed function and paste it into the `.env` file where the `REACT_APP_HOSTED_SERVER_URL` placeholder string is (e.g. `https://xxxx-xxxxx-####-dev.twil.io`):

    ```
    ## base endpoint of your hosted server (twilio serverless functions)
    REACT_APP_HOSTED_SERVER_URLhttps://xxxx-xxxxx-####-dev.twil.io
    ```

2. The following command will build the React app and place the minimized build in the `assets` folder of our Twilio Serverless function:

    ```shell
    yarn build
    ```

3. Lastly, deploy the function again with the following:
    ```
    yarn deploy-server
    ```

## Update Your Website Template (optional)

Once the bundle is uploaded, you can have it loaded in your website page, as per example below:

```html
<iframe src="https://[...]index.html"></iframe>
```

# Browser Support

For Browser Support, please refer to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app) README.

# Accessibility

For Accessbility, please refer to the [_Twilio Webchat React_](https://github.com/twilio/twilio-webchat-react-app) README..

# FAQs

### As a developer, how do I clear an ongoing chat?

Open your browser console, run `localStorage.clear()` and refresh the page to start anew.
Alternatively, you can simply wrap up/complete the corresponding task as an agent from your Flex UI instance.

### Can I use npm?

Currently there is a known issue with installing dependencies for this project using npm. We are investigating this and will publish a fix as soon as possible. We recommend using `yarn` instead.

# License

MIT © Twilio Inc.
