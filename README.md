# Building real-time applications with iOS, GraphQL & AWS AppSync

In this workshop we'll learn how to build cloud-enabled native iOS Swift apps with [AWS Amplify](https://aws-amplify.github.io/) and connect our apps to a GraphQL API via AWS AppSync.

We'll start from a new Xcode project, add categories such as `API` and `Auth` using the Amplify Framework to provision cloud resources such as a hosted GraphQL API via AWS AppSync, Amazon DynamoDB as a data source, or an Identity Provider via Amazon Cognito to provide basic authentication. We'll start simple and work our way up to a fully connected mobile app. Please provide any feedback you have in the 'issues' and I'll take a look. Let's get started!

![](./amplify-swift-logo-cli.png)

### Topics we'll be covering:

- [GraphQL API with AWS AppSync](https://github.com/wizage/aws-appsync-ios-workshop#getting-started---create-an-xcode-project)
- [Deleting the resources](https://github.com/wizage/aws-appsync-ios-workshop#removing-categories-from-amplify-project)


#### SwiftUI for the brave
- [(Alpha) Using SwiftUI](https://github.com/wizage/aws-appsync-ios-workshop/tree/SwiftUI)

## Getting Started - Create an Xcode project

To get started, create a new Xcode project for iOS Swift & save as: `ios-amplify-app`

From a Mac Terminal, change into the new app directory & prepare to install and congigure the Amplify CLI.

### Install and Configure the Amplify CLI - Just Once

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```

After installation, configure the CLI with your developer credentials:

Note: If you already have the AWS CLI installed and use a named profile, you can skip the `amplify configure` step.
`Amplify configure` is going to have you launch the AWS Management Console, create a new IAM User, asign an IAM Policy, and collect the programmatic credentials to craate a CLI profile that will be used to provision AWS resources for each project in future steps.

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __us-east-1__
- Specify the username of the new IAM user: __amplify-workshop-user__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
? accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
? secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __amplify-workshop-user__

### Initializing A New Amplify Project
From the root of your Xcode project folder:
```bash
amplify init
```

- Enter a name for the project: __iosamplifyapp__
- Enter a name for the environment: __master__
- Choose your default editor: __Visual Studio Code (or your default editor)__   
- Please choose the type of app that you're building __ios__     
- Do you want to use an AWS profile? __Y__
- Please choose the profile you want to use: __amplify-workshop-user__

AWS Amplify CLI will iniatilize a new project & you'll see a new folder: __amplify__ & a new file called `awsconfiguration.json` in the root directory. These files hold your Amplify project configuration.

To view the status of the amplify project at any time, you can run the Amplify `status` command:

```sh
amplify status
```

## Adding a GraphQL API
In this section we'll add a new GraphQL API via AWS AppSync to our iOS project backend. 
To add a GraphQL API, we can use the following command:

```sh
amplify add api
```

Answer the following questions:

- Please select from one of the above mentioned services __GraphQL__   
- Provide API name: __ConferenceAPI__   
- Choose an authorization type for the API __API key__   
- Do you have an annotated GraphQL schema? __N__   
- Do you want a guided schema creation? __Y__   
- What best describes your project: __Single object with fields (e.g. “Todo” with ID, name, description)__   
- Do you want to edit the schema now? (Y/n) __Y__   

When prompted and the default schema launches in your favorite editor, update the default schema to the following:   

```graphql
type Talk @model {
  id: ID!
  clientId: ID
  name: String!
  description: String!
  speakerName: String!
  speakerBio: String!
}
```

Next, let's deploy the GraphQL API into our account:
This step take the local CloudFormation templates and deployes them to the AWS Cloud for provisioning of the services you enabled via the `add API` category.
```bash
amplify push
```

- Do you want to generate code for your newly created GraphQL API __Y__
- Enter the file name pattern of graphql queries, mutations and subscriptions: __(graphql/**/*.graphql)__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions? __Y__
- Enter maximum statement depth [increase from default if your schema is deeply nested] __2__
- Enter the file name for the generated code __API.swift__

> To view the new AWS AppSync API at any time after its creation, go to the dashboard at [https://console.aws.amazon.com/appsync](https://console.aws.amazon.com/appsync). Also be sure that your region is set correctly.

### Performing mutations from within the AWS AppSync Console

In the AWS AppSync console, open your API & then click on Queries.

Execute the following mutation to create a new talk in the API:

```bash
mutation createTalk {
  createTalk(input: {
    name: "Monetize your Mobile Apps"
    description: "4 ways to make money as a mobile app developer"
    speakerName: "Sam Patzer"
    speakerBio: "Mobile Quickie Developer"
  }) 
  {
      id 
      name 
      description 
      speakerName 
      speakerBio
  }
}
```

Now, let's query for the talk:

```bash
query listTalks {
  listTalks {
    items {
      id
      name
      description
      speakerName
      speakerBio
    }
  }
}
```

We can even add search / filter capabilities when querying:

```bash
query listTalks {
  listTalks(filter: {
    description: {
      contains: "money"
    }
  }) {
    items {
      id
      name
      description
      speakerName
      speakerBio
    }
  }
}
```

### Configuring the iOS applicaion - AppSync iOS Client SDK

Our backend resources have been created and we just verified mutations and queries in the AppSync Console. Let's move onto the mobile client!

To configure the app, we'll use [Cocoapods](https://cocoapods.org/) to install the AWS SDK for iOS and AWS AppSync Client dependencies.
In the root project folder, run the following command to initialize Cocoapods.

```js
pod init
```

This will create a new `Podfile`. Open up the `Podfile` in your favorite editor and add the following dependency for adding the AWSAppSync SDK to your app:

```swift
target 'ios-amplify-app' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for ios-amplify-app
  pod 'AWSAppSync'
  
end
```

Install the AppSync iOS SDK by running:
```js
pod install --repo-update
```

## Add the `awsconfiguration.json` and `API.swift` files to your Xcode project

We need to configure our iOS Swift application to be aware of our new AWS Amplify project. We do this by referencing the auto-generated `awsconfiguration.json` and `API.Swift` files in the root of your Xcode project folder.

Launch Xcode using the .xcworkspace from now on as we are using Cocoapods.
```js
$ open ios-amplify-app.xcworkspace/
```

In Xcode, right-click on the project folder and choose `"Add Files to ..."` and add the `awsconfiguration.json` and the `API.Swift` files to your project. When the Options dialog box that appears, do the following:

* Clear the Copy items if needed check box.
* Choose Create groups, and then choose Next.

Build the project (Command-B) to make sure we don't have any compile errors.

## Initialize the AppSync iOS Client
#### Update ViewController.swift
Add the folowing four numbered code snippets to your `ViewController.swift` class:

```swift
import AWSAppSync // #1

class ViewController: UIViewController {
    
    // Reference AppSync client
    var appSyncClient: AWSAppSyncClient? // #2
    
    //...
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //...

        initializeAppSync() // #3
        
        //...
    }

    // #4
    // Use this code when setting up AppSync with API_key auth mode. This mode is used for testing our GraphQL mutations, queries, and subscriptions with an API key.
    func initializeAppSync() {
        do {
        // Initialize the AWS AppSync configuration
        let appSyncConfig = try AWSAppSyncClientConfiguration(appSyncServiceConfig: AWSAppSyncServiceConfig(),
        cacheConfiguration: AWSAppSyncCacheConfiguration())
        
        // Initialize the AWS AppSync client
        appSyncClient = try AWSAppSyncClient(appSyncConfig: appSyncConfig)
        } catch {
        print("Error initializing appsync client. \(error)")
        }
    }
    // End #4
}
```

## Add GraphQL Mutation

Add the following mutation function to your `ViewController.swift` class:

```swift
// GraphQL Mutation - Add a new talk
func createTalkMutation(){
    let conferenceInput = CreateTalkInput(name: "Monetize your iOS app", description: "How to make dough as an iOS developer", speakerName: "Steve Jobs", speakerBio: "I do cool stuff at Apple")
    appSyncClient?.perform(mutation: CreateTalkMutation(input: conferenceInput))
    { (result, error) in
        if let error = error as? AWSAppSyncClientError {
            print("Error occurred: \(error.localizedDescription )")
        }
        if let resultError = result?.errors {
            print("Error saving conf talk: \(resultError)")
            return
        }
        
        guard let result = result?.data else { return }
        
        print("Talk created: \(String(describing: result.createTalk?.id))")
    }
}
```

## Add GraphQL Query

Add the following query function to your `ViewController.swift` class:

```swift
// GraphQL Query - List all talks
func getTalksQuery(){
    appSyncClient?.fetch(query: ListTalksQuery(), cachePolicy: .returnCacheDataAndFetch) { (result, error) in
        if error != nil {
            print(error?.localizedDescription ?? "")
            return
        }
        
        guard let talks = result?.data?.listTalks?.items else { return }
        talks.forEach{ print(("Title: " + ($0?.name)!) + "\nSpeaker: " + (($0?.speakerName)! + "\n")) }
    }
}
```

### Run App and Invoke Mutation and Query Functions

In order to execute the mutation and/or query functions above, we can invoke those functions from `ViewDidLoad()` in the `ViewController.swift` class:
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    //...

    createTalkMutation()
    getTalksQuery()

    //...
}
```
Build (Command-B) and run your app.

### Add GraphQL Subscriptions

GraphQL subscriptions give us the real-time updates when data changes in our API. 
First, set the discard variable at the `ViewController.swift` class level:

```swift
// Set a discard variable at the class level
var discard: Cancellable?
```

Now add this new `subscribeToTalks()` function to your `ViewController.swift` class: 

```swift
func subscribeToTalks() {
    do {
        discard = try appSyncClient?.subscribe(subscription: OnCreateTalkSubscription(), resultHandler: { (result, transaction, error) in
            if let result = result {
                print("Subscription triggered! " + result.data!.onCreateTalk!.name + " " + result.data!.onCreateTalk!.speakerName)
            } else if let error = error {
                print(error.localizedDescription)
            }
        })
    } catch {
        print("Error starting subscription.")
    }
}
```

Finally, call the `subscribeToTalks()` from `ViewDidLoad()` in your `ViewController.swift` class:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    // ...
    
    // invoke to subscribe to newly created talks
    subscribeToTalks()
}
```
> Don't forget to comment out the calls to createTalkMutation() getTalksQuery() in the ViewDidLoad() or the app will create a new talk each time it loads.

Run the mobile app and it'll then subscribe to any new talk creation. To test the real-time subscription: Leave the app running and then create a new talk via the AppSync Console through a Mutation and you should see the iOS app log the new talk via a subscription!


## Removing Categories from Amplify Project

If at any time, or at the end of this workshop, you would like to delete a category from your project & your account, you can do this by running the `amplify remove` command:

```sh
amplify remove api

amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.

## Deleting the Amplify Project

```sh
amplify delete
```
