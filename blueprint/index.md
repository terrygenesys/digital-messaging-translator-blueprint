---
title: Build a messaging translation assistant with the AWS Translate service
author: agnes.corpuz
indextype: blueprint
icon: blueprint
image: images/banner.png
category: 6
summary: |
  This Genesys Cloud Developer Blueprint provides instructions for building a translation assistant which uses the AWS Translate service to allow customers and agents to chat in their preferred languages. The translation assistant automatically translates everything in the integration window in real-time, including canned responses. This supports both web chat and web messaging interactions. All the components used in this solution can be deployed using Terraform Genesys Cloud CX as Code provider.
---

This Genesys Cloud Developer Blueprint provides instructions for building a translation assistant which uses the AWS Translate service to allow customers and agents to chat in their preferred languages. The translation assistant automatically translates everything in the integration window in real-time, including canned responses. This supports both web chat and web messaging interactions. All the components used in this solution can be deployed using Terraform Genesys Cloud CX as Code provider.

![Digital Messaging translation assistant](images/overview.png "Digital Messaging translation assistant")

## Scenario

An organization wants to provide a real-time translation for web chat and web messaging that would allow both their customers and agents have a conversation using their preferred language. To do this, the organization wants to:

1. **The customer sends a message using their preferred language.** The agent will receive an incoming interaction and will see the customer's message.

2. **Have the agent open the Message Translator interaction widget.** The agent opens the widget for the interaction and have the widget display the message translation based on their configured preferred language in Genesys Cloud.

3. **Allow the agent to send messages using their preferred language.** The agent types and sends a response in the widget chat box using their preferred language.

4. **Customer receives the agent's reposonse translated in their language.** The customer will see a translated response from the agent.

## Content

* [Solution](#solution "Goes to the Solution section")
* [Solution components](#solution-components "Goes to the Solution components section")
* [Requirements](#requirements "Goes to the Requirements section")
* [Implementation steps](#implementation-steps "Goes to the Implementation steps section")
* [Additional resources](#additional-resources "Goes to the Additional resources section")

## Solution

* **Architect inbound message flow** - Provides the routing layer that gets the customer to the right queue.
* **Interaction widget integration** - The Genesys Cloud integration that enables web apps to be embedded in an iframe within Genesys Cloud. The iframe only appears on specified interaction types and to specified agents. For this solution, Genesys Cloud uses the Interaction Widget integration to show translated web chat and web messages to the customer.
* **Web messenger** - Allows developers to create and configure a JavaScript web messenger that deploys to their organization's website where customers can interact with it.

## Solution components

* **Genesys Cloud CX** - A suite of Genesys cloud services for enterprise-grade communications, collaboration, and contact center management. In this solution, you use an Architect inbound message flow, and a Genesys Cloud queues, web message configuration and web message deployment.
* **CX as Code** - A Genesys Cloud Terraform provider that provides an interface for declaring core Genesys Cloud objects.
* **AWS IAM** - Identity and Access Management that controls access to AWS resources such as services or features. In this solution, you set the permissions to allow the Messaging Translator to access Amazon Translate and the AWS SDK.
* **Amazon Translate** - A translation service that enables cross-lingual communication between users of an application. Amazon Translate is the translation service used in the Messaging Translator solution.

### Software development kits (SDKs)

* **Genesys Cloud Platform API SDK** - Client libraries used to simplify application integration with Genesys Cloud by handling low-level HTTP requests. This SDK is used for the initial chat and messaging interaction between agent and customer.
* **AWS for JavaScript SDK** - This SDK enables developers to build and deploy applications that use AWS services. This solution uses the JavaScript API to enable the Messaging Translator in an agent's browser and it uses the inside Node.js applications to enable the Messaging Translator on the server where Genesys Cloud runs.

## Prerequisites

### Specialized knowledge

* Administrator-level knowledge of Genesys Cloud
* AWS Cloud Practitioner-level knowledge of AWS IAM, AWS Translate, and AWS for JavaScript SDK
* Experience using the Genesys Cloud Platform API
* Experience with Terraform

### Genesys Cloud account

* A Genesys Cloud license. For more information, see [Genesys Cloud Pricing](https://www.genesys.com/pricing "Opens the Genesys Cloud pricing page") in the Genesys website.
* The Master Admin role. For more information, see [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Opens the Roles and permissions overview article") in the Genesys Cloud Resource Center.
* CX as Code. For more information see, [CX as Code](https://developer.genesys.cloud/devapps/cx-as-code/ "Goes to the CX as Code page") in the Genesys Cloud Developer Center.

### AWS account

* A user account with AdministratorAccess permission and full access to the following services:
  * IAM service
  * Translate service

### Development tools running in your local environment

* Terraform (the latest binary). For more information, see [Download Terraform](https://www.terraform.io/downloads.html "Goes to the Download Terraform page") on the Terraform website.
* Golang 1.16 or higher. For more information, see [Downloads](https://go.dev/dl/ "Goes to the Downloads page") on the Go website.

## Implementation steps

* [Download the repository containing the project files](#download-the-repository-containing-the-project-files "Goes to the Download the repository containing the project files section")
* [Set up AWS Translate](#set-up-aws-translate "Goes to the Set up AWS Translate section")
* [Set up Genesys Cloud](#set-up-genesys-cloud "Goes to the Set up Genesys Cloud section")
* [Configure your Terraform build](#configure-your-terraform-build "Goes to the Configure your Terraform build section")
* [Run Terraform](#run-terraform "Goes to the Run Terraform section")
* [Additional configurations](#additional-configurations "Goes to the Additional configurations section")
* [Host and run the Node.js app server](#host-and-run-the-node-js-app-server "Goes to the Host and run the Node.js app server section")
* [Test the solution](#test-the-solution "Goes to the Test the solution section")

### Download the repository containing the project files

1. Clone the [digital-messaging-translator-blueprint repository](https://github.com/GenesysCloudBlueprints/digital-messaging-translator-blueprint "Opens the digital-messaging-translator-blueprint repository in GitHub").

### Set up AWS Translate

1. Create an IAM user for the application. For more information, see [IAM users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html "Opens IAM users") in the AWS documentation.
2. Add a policy to the IAM that grants full access to the AWS Translate service. For more information, see [Managing IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage.html "Opens Managing IAM policies") in the AWS documentation.
3. Create an access key for the IAM user. For more information, see [Managing access keys for IAM users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html "Opens Managing access keys for IAM users") in the AWS documentation.
4. Write down the access key and secret.
5. Create an .env file in the directory folder and provide values for the following variables: `AWS_REGION`, `AWS_ACCESS_KEY_ID`, and `AWS_SECRET_ACCESS_KEY`.

  :::primary
  **Tip**: Start with the sample.env file for this blueprint, rename it to `.env` and provide your org-specific details.
  :::

### Set up Genesys Cloud

1. To run this project using the Terraform provider, open a terminal window and set the following environment variables:

 * `GENESYSCLOUD_OAUTHCLIENT_ID` - This is the Genesys Cloud client credential grant Id that CX as Code executes against. 
 * `GENESYSCLOUD_OAUTHCLIENT_SECRET` - This is the Genesys Cloud client credential secret that CX as Code executes against. 
 * `GENESYSCLOUD_REGION` - This is the Genesys Cloud region in your organization.

2. Run Terraform in the terminal window where the environment variables are set. 

### Configure your Terraform build

You must define several values that are specific to your Genesys Cloud organization. 

In the blueprint/terraform/dev.auto.tfvars file, set the following values:

* `environment` - This is a free-form field that combines with the prefix value to define the name of various Genesys Cloud artifacts. For example, if you set the environment name to be `dev` and the prefix to be `web-messaging` your Genesys Cloud group, queue, messenger configuration, messenger deployment, etc.will all begin with `dev-web-messaging`.
* `prefix`- This a free-form field that combines with the environment value to define the name of various Genesys Cloud artifacts.
* `email` - your email used in Genesys Cloud. This will bee used to assign you to the Genesys Cloud group and queue.

The following is an example of the dev.auto.tfvars file that was created by the author of this blueprint.

```
environment            = "dev"
prefix                 = "web-messaging"
email                  = "test-email@company.com"
```

### Run Terraform

You are now ready to run this blueprint solution for your organization. 

1. Change to the docs/terraform folder and issue these commands:

* `terraform plan` - This executes a trial run against your Genesys Cloud organization and shows you a list of all the Genesys Cloud resources created. Review this list and make sure you are comfortable with the activity being undertake before continuing to the second step.

* `terraform apply --auto-approve` - This does the actual object creation and deployment against your Genesys Cloud account. The --auto--approve flag steps the approval step required before creating the objects.

After the `terraform apply --auto-approve` command has completed, you should see the output of the entire run along with the number of objects successfully created by Terraform. Keep these points in mind:

*  This project assumes you are running using a local Terraform backing state. This means that the `tfstate` files will be created in the same folder where you ran the project. Terraform does not recommend using local Terraform backing state files unless you run from a desktop and are comfortable with the deleted files.

* As long as your local Terraform backing state projects are kept, you can tear down the blueprint in question by changing to the `docs/terraform` folder and issuing a `terraform destroy --auto-approve` command. This destroys all objects currently managed by the local Terraform backing state.

### Additional configurations

After you have created the Genesys Cloud objects, you can now use the Messenger Deployment to add a Messenger to your website.

1. Navigate to **Admin** > **Message** > **Messenger Deployments** > **dev-web-messaging-deployment**.
2. Under **Deploy your snippet**, click **Copy to Clipboard** to copy the snippet. Paste the snippet to the `<head>` tag of all you webpages.

You also neeed to update the config file found in /docs/scripts/config.js to use the OAuth client.

1. Navigate to **Admin** > **Integrations** > **OAuth** > **Web Messages Implicit Client**.
2. Add the client ID from your OAuth client and specify the region where your Genesys Cloud organization is located, for example, `mypurecloud.ie` or `mypurecloud.com.au`.

### Host and run the Node.js app server

1. At a command line, verify that you are running Node.js v14.15.4 or later. Open a command line tool and type `node-v`.
  * To upgrade, type `nvm install 14.15.4`.
  * To install the latest version, type `npm install -g n latest`.

2. Switch to the directory where the files for your Messaging Translator project are located and install the dependencies in the local node-modules folder. In the command line, type `npm install`.
3. To run the server locally, in the command line type `node run-local.js`.

### Test the solution

#### Translate web message interactions

1. Go to your website and start a web message.
   ![Start web message interaction](images/start-web-message.png "Start web message interaction")
2. To answer the message as an agent, in your Genesys Cloud organization change your status to **On Queue** and then answer the incoming interaction.
3. To open the Messaging Translator, click the **Messaging Translator** button, which appears in the agent's toolbar.
   ![Web message interaction](images/web-message-interaction.png "Incoming web message interaction")
4. Practice sending and receiving messages in different languages. When you type a message, the Messaging Translator automatically translates it into the language that the customer is using.
  :::primary
  **Important**: Make sure to type on the right side of the interaction for the Messaging Translator app to successfully translate the agent's message.
  :::
  ![Translated message](images/web-message-translate.png "Translated message")
  ![Customer view](images/web-message-translate-customer.png "Customer view")
5. To send a translated canned response, click **Open Canned Responses** and select a canned response.  
  ![Translated canned response](images/translate-canned-response.png "Translated canned response")

#### Translate web chat interactions

1. Create a Genesys web chat widget. For more information, see [Create a widget for web chat](https://help.mypurecloud.com/?p=195772 "Opens the Create a widget for web chat article") in the Genesys Cloud Resource Center.

  :::primary
  **Important**: If you will use the Genesys Cloud developer tools to test this solution, then make sure that under **Widget Type** you select **Version 2**, **Version 1.1**, or **Third Party**. For more information see [About widgets for web chat](https://help.mypurecloud.com/articles/?p=194115 "Opens the About widgets for web chat article") in the Genesys Cloud Resource Center.
  :::

2. Open the [Web Chat developer tool](https://developer.mypurecloud.com/developer-tools/#/webchat "Opens the Web Chat developer tool").

  :::primary
  **Important**: Make sure the Developer Center URL matches the region where your Genesys Cloud organization is located. For more information, see the [Access the developer tools](https://developer.mypurecloud.com/gettingstarted/developer-tools-intro.html#accessTools "Goes to the Access the developer tools section on the Developer tools quick start page") section on the Developer tools quick start page.
  :::

3. Go to the [Chat Configuration page in the Genesys Cloud Developer Center](https://developer.mypurecloud.com/developer-tools/#/webchat "Opens the Chat Configuration page in the Genesys Cloud Developer Center").
4. Click **Populate Fields**.
5. To start a chat as a customer, click **Start Chat**.
6. To answer the chat as an agent, in your Genesys Cloud organization change your status to **On Queue** and then answer the incoming interaction.
  ![Chat interaction](images/chat-interaction.png "Incoming chat interaction")
7. To open the Messaging Translator, click the **Messaging Translator** button, which appears in the agent's toolbar.
8. Practice sending and receiving chats in different languages. When you type a chat, the Messaging Translator automatically translates it into the language that the customer is using.
  :::primary
  **Important**: Make sure to type on the right side of the interaction for the Messaging Translator app to successfully translate the agent's message.
  :::
  ![Translated chat](images/chat-translate.png "Translated chat")
9. To send a translated canned response, click **Open Canned Responses** and select a canned response.  
  ![Translated canned response](images/translate-canned-response-chat.png "Translated canned response")

## Additional resources

* [Genesys Cloud Platform Client SDK](https://developer.mypurecloud.com/api/rest/client-libraries/ "Opens the Genesys Cloud Platform Client SDK page")
* [About web messaging](https://help.mypurecloud.com/articles/about-web-messaging/ "Opens the About Web Messaging page")
* [Amazon Translate](https://aws.amazon.com/translate/ "Opens Amazon Translate page") in the AWS documentation
* [digital-messaging-translator-blueprint repository](https://github.com/GenesysCloudBlueprints/digital-messaging-translator-blueprint "Opens the chat-translator-blueprint repository in GitHub")