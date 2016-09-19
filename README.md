# Dropbox for Swift

The Official Dropbox Swift SDK for integrating with Dropbox [API v2](https://www.dropbox.com/developers/documentation/http/documentation) on iOS or macOS.

Full documentation [here](http://dropbox.github.io/SwiftyDropbox/api-docs/latest/).

---

## Table of Contents

* [System Requirements](#system-requirements)
  * [Swift 3 Keychain bug](#swift-3-keychain-bug)
* [Get Started](#get-started)
  * [Register your application](#register-your-application)
  * [Obtain an OAuth2 token](#obtain-an-oauth2-token)
* [SDK Distribution](#sdk-distribution)
  * [CocoaPods](#cocoapods)
  * [Carthage](#carthage)
  * [Manually add subproject](#manually-add-subproject)
  * [Swift 2.3](#swift-23)
* [Configure your project](#configure-your-project)
  * [Application `.plist` file](#application-plist-file)
  * [Handling the authorization flow](#handling-the-authorization-flow)
    * [Initialize a `DropboxClient` instance](#initialize-a-dropboxclient-instance)
    * [Begin the authorization flow](#begin-the-authorization-flow)
    * [Handle redirect back into SDK](#handle-redirect-back-into-sdk)
* [Try some API requests](#try-some-api-requests)
  * [Dropbox client instance](#dropbox-client-instance)
  * [Handle the API response](#handle-the-api-response)
  * [Request types](#request-types)
    * [RPC-style request](#rpc-style-request)
    * [Upload-style request](#upload-style-request)
    * [Download-style request](#download-style-request)
  * [Handling responses and errors](#handling-responses-and-errors)
    * [Route-specific errors](#route-specific-errors)
    * [Generic network request errors](#generic-network-request-errors)
    * [Response handling edge cases](#response-handling-edge-cases)
  * [Customizing network calls](#customizing-network-calls)
    * [Specify global response queue](#specify-global-response-queue)
    * [Specify API call response queue](#specify-api-call-response-queue)
  * [`DropboxClientsManager` class](#dropboxclientsmanager-class)
    * [Single Dropbox user case](#single-dropbox-user-case)
    * [Multiple Dropbox user case](#multiple-dropbox-user-case)
* [Examples](#examples)
* [Documentation](#documentation)
* [Stone](#stone)
* [Modifications](#modifications)
* [Bugs](#bugs)

---

## System Requirements

- iOS 9.0+
- macOS 10.11+
- Xcode 8.0+

### Swift 3 Keychain bug

> SwiftyDropbox currently supports Swift 3. However, there appears to be a bug with the Keychain in the iOS simulator environment where data is not persistently saved to the Keychain.
>
> As a temporary workaround, in the Project Navigator, select **your project** > **Capabilities** > **Keychain Sharing** > **ON**.
>
> You can read more about the bug [here](https://forums.developer.apple.com/message/170381#170381).

---

## Get Started

### Register your application

Before using this SDK, you should register your application in the [Dropbox App Console](https://dropbox.com/developers/apps). This creates a record of your app with Dropbox that will be associated with the API calls you make.

### Obtain an OAuth2 token

All requests need to be made with an OAuth2 access token. An OAuth token represents an authenticated link between a Dropbox app and
a Dropbox user account or team.

Once you've created an app, you can go to the App Console and manually generate an access token to authorize your app to access your own Dropbox account.
Otherwise, you can obtain an OAuth token programmatically using the SDK's pre-defined auth flow. For more information, [see below](https://github.com/dropbox/SwiftyDropbox#handling-authorization-flow).

---

## SDK Distribution

You can integrate the Dropbox Swift SDK into your project using one of several methods.

### CocoaPods

To use [CocoaPods](http://cocoapods.org), a dependency manager for Cocoa projects, you should first install it using the following command:

```bash
$ gem install cocoapods
```

Then navigate to the directory that contains your project and create a new file called `Podfile`. You can do this either with `pod init`, or open an existing Podfile, and then add `pod 'SwiftyDropbox'` to the main loop. Your Podfile should look something like this:

```ruby
use_frameworks!

target '<YOUR_PROJECT_NAME>' do
    pod 'SwiftyDropbox'
end
```

Then, run the following command to install the dependency:

```bash
$ pod install
```

**Note**: If you are encountering `Use Legacy Swift Language Version` and/or `Always Embed Swift Standard Libraries` warnings after running `pod install`, then add the following to the end of your Podfile:

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['SWIFT_VERSION'] = '3.0'
      config.build_settings['ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES'] = 'YES'
    end
  end
end
```

Once your project is integrated with the Dropbox Swift SDK, you can pull SDK updates using the following command:

```bash
$ pod update
```

---

### Carthage

You can also integrate the Dropbox Swift SDK into your project using [Carthage](https://github.com/Carthage/Carthage), a decentralized dependency manager for Cocoa. Carthage offers more flexibility than CocoaPods, but requires some additional work. You can install Carthage (with Xcode 7+) via [Homebrew](http://brew.sh/):

```bash
brew update
brew install carthage
```

 To install the Dropbox Swift SDK via Carthage, you need to create a `Cartfile` in your project with the following contents:

```
# SwiftyDropbox
github "https://github.com/dropbox/SwiftyDropbox" ~> 4.0.3
```

Then, run the following command to install the dependency to checkout and build the Dropbox Swift SDK repository:

##### iOS

```bash
carthage update --platform iOS
```

In the Project Navigator in Xcode, select your project, and then navigate to **General** > **Linked Frameworks and Libraries**, then drag and drop `SwiftyDropbox.framework` (from `Carthage/Build/iOS`).

Then, navigate to **Build Phases** > **+** > **New Run Script Phase**. In the newly-created **Run Script** section, add the following code to the script body area (beneath the "Shell" box):

```
/usr/local/bin/carthage copy-frameworks
```

Then, navigate to the **Input Files** section and add the following path:

```
$(SRCROOT)/Carthage/Build/iOS/SwiftyDropbox.framework
```

##### macOS
```bash
carthage update --platform Mac
```

In the Project Navigator in Xcode, select your project, and then navigate to **General** > **Embedded Binaries**, then drag and drop `SwiftyDropbox_macOS.framework` (from `Carthage/Build/Mac`).

Then navigate to **Build Phases** > **+** > **New Copy Files Phase**. In the newly-created **Copy Files** section, click the **Destination** drop-down menu and select **Products Directory**, then drag and drop `SwiftyDropbox_macOS.framework.dSYM` (from `Carthage/Build/Mac`).

---

### Manually add subproject

Finally, you can also integrate the Dropbox Swift SDK into your project manually without using a dependency manager.

Drag the `Source/SwiftyDropbox.xcodeproj` project into your project as a subproject.

Then, in the Project Navigator in Xcode, select your project, and then navigate to your project's build target > **General** > **Embedded Binaries** > **+** and then add `SwiftyDropbox.framework`.

---

## Swift 2.3

SwiftyDropbox currently supports only Swift 3+. However, we have a Swift 2.3 compatible branch, if necessary. To access it, you can either pull the `swift_2_3` branch from the repo, or using one of the following distribution channels: 

#### CocoaPods
```ruby
use_frameworks!

target '<YOUR_PROJECT_NAME>' do
    pod 'SwiftyDropbox', :git => 'https://github.com/dropbox/SwiftyDropbox', :branch => 'swift_2_3'
end
```

#### Carthage
```
# SwiftyDropbox
github "https://github.com/dropbox/SwiftyDropbox" ~> 3.4.0
```

---

## Configure your project

Once you have integrated the Dropbox Swift SDK into your project, there are a few additional steps to take before you can begin making API calls.

### Application `.plist` file

If you are compiling on iOS SDK 9.0, you will need to modify your application's `.plist` to handle Apple's [new security changes](https://developer.apple.com/videos/wwdc/2015/?id=703) to the `canOpenURL` function. You should
add the following code to your application's `.plist` file:

```
<key>LSApplicationQueriesSchemes</key>
    <array>
        <string>dbapi-8-emm</string>
        <string>dbapi-2</string>
    </array>
```
This allows the Swift SDK to determine if the official Dropbox iOS app is installed on the current device. If it is installed, then the official Dropbox iOS app can be used to programmatically obtain an OAuth2 access token.

Additionally, your application needs to register to handle a unique Dropbox URL scheme for redirect following completion of the OAuth2 authorization flow. This URL scheme should have the format `db-<APP_KEY>`, where `<APP_KEY>` is your
Dropbox app's app key, which can be found in the [App Console](https://dropbox.com/developers/apps).

You should add the following code to your `.plist` file (but be sure to replace `<APP_KEY>` with your app's app key):

```
<key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>db-<APP_KEY></string>
            </array>
            <key>CFBundleURLName</key>
            <string></string>
        </dict>
    </array>
```

After you've made the above changes, your application's `.plist` file should look something like this:

<p align="center">
  <img src="https://github.com/dropbox/SwiftyDropbox/blob/master/Images/InfoPlistExample.png?raw=true" alt="Info .plist Example"/>
</p>

---

### Handling the authorization flow

There are three methods to programmatically retrieve an OAuth2 access token:

* **Direct auth** (iOS only): This launches the official Dropbox iOS app (if installed), authenticates via the official app, then redirects back into the SDK
* **In-app webview auth** (iOS, macOS): This opens a pre-built in-app webview for authenticating via the Dropbox authorization page. This is convenient because the user is never redirected outside of your app.
* **External browser auth** (iOS, macOS): This launches the platform's default browser for authenticating via the Dropbox authorization page. This is desirable because it is safer for the end-user, and pre-existing session data can be used to avoid requiring the user to re-enter their Dropbox credentials.

To facilitate the above authorization flows, you should take the following steps:

---

#### Initialize a `DropboxClient` instance

From your application delegate: 

##### iOS

```Swift
import SwiftyDropbox

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    DropboxClientsManager.setupWithAppKey("<APP_KEY>")
    return true
}
```

##### macOS

```Swift
import SwiftyDropbox

func applicationDidFinishLaunching(_ aNotification: Notification) {
    DropboxClientsManager.setupWithAppKeyDesktop("<APP_KEY>")
}
```

---

#### Begin the authorization flow

You can commence the auth flow by calling `authorizeFromController:controller:openURL:browserAuth` method in your application's
view controller. If you wish to authenticate via the in-app webview, then set `browserAuth` to `NO`. Otherwise, authentication will be done via an external web browser.

From your view controller: 

##### iOS

```Swift
import SwiftyDropbox

func myButtonInControllerPressed() {
    DropboxClientsManager.authorizeFromController(UIApplication.shared,
                                                  controller: self,
                                                  openURL: { (url: URL) -> Void in
                                                    UIApplication.shared.openURL(url)
                                                  })
}

```

##### macOS

```Swift
import SwiftyDropbox

func myButtonInControllerPressed() {
    DropboxClientsManager.authorizeFromController(sharedWorkspace: NSWorkspace.shared(),
                                                  controller: self,
                                                  openURL: { (url: URL) -> Void in
                                                    NSWorkspace.shared().open(url)
                                                  })
}
```

Beginning the authentication flow via in-app webview will launch a window like this:


<p align="center">
  <img src="https://github.com/dropbox/SwiftyDropbox/blob/master/Images/OAuthFlowInit.png?raw=true" alt="Auth Flow Init Example"/>
</p>

---

#### Handle redirect back into SDK

To handle the redirection back into the Swift SDK once the authentication flow is complete, you should add the following code in your application's delegate:

##### iOS

```Swift
import SwiftyDropbox

func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
    if let authResult = DropboxClientsManager.handleRedirectURL(url) {
        switch authResult {
        case .success:
            print("Success! User is logged into Dropbox.")
        case .cancel:
            print("Authorization flow was manually canceled by user!")
        case .error(_, let description):
            print("Error: \(description)")
        }
    }
    return true
}

```

##### macOS

```Swift
import SwiftyDropbox

func handleGetURLEvent(_ event: NSAppleEventDescriptor?, replyEvent: NSAppleEventDescriptor?) {
    if let aeEventDescriptor = event?.paramDescriptor(forKeyword: AEKeyword(keyDirectObject)) {
        if let urlStr = aeEventDescriptor.stringValue {
            let url = URL(string: urlStr)!
            if let authResult = DropboxClientsManager.handleRedirectURL(url) {
                switch authResult {
                case .success:
                    print("Success! User is logged into Dropbox.")
                case .cancel:
                    print("Authorization flow was manually canceled by user!")
                case .error(_, let description):
                    print("Error: \(description)")
                }
            }
        }
    }
```

After the end user signs in with their Dropbox login credentials via the in-app webview, they will see a window like this:


<p align="center">
  <img src="https://github.com/dropbox/SwiftyDropbox/blob/master/Images/OAuthFlowApproval.png?raw=true" alt="Auth Flow Approval Example"/>
</p>

If they press **Allow** or **Cancel**, the `db-<APP_KEY>` redirect URL will be launched from the webview, and will be handled in your application
delegate's `application:handleOpenURL` method, from which the result of the authorization can be parsed.

Now you're ready to begin making API requests!

---

## Try some API requests

Once you have obtained an OAuth2 token, you can try some API v2 calls using the Swift SDK.

### Dropbox client instance

Start by creating a reference to the `DropboxClient` or `DropboxTeamClient` instance that you will use to make your API calls.

```Swift
import SwiftyDropbox

// Reference after programmatic auth flow
let client = DropboxClientsManager.authorizedClient
```

or

```Swift
import SwiftyDropbox

// Initialize with manually retrieved auth token
let client = DropboxClient(accessToken: "<MY_ACCESS_TOKEN>")
```

---

### Handle the API response

The Dropbox [User API](https://www.dropbox.com/developers/documentation/http/documentation) and [Business API](https://www.dropbox.com/developers/documentation/http/teams) have three types of requests: RPC, Upload and Download.

The response handlers for each request type are similar to one another. The arguments for the handler blocks are as follows:
* **route result type** (`Void` if the route does not have a return type)
* **network error** (either a route-specific error or generic network error)
* **output content** (`URL` / `Data` reference to downloaded output for Download-style endpoints only)

Note: Response handlers are required for all endpoints. Progress handlers, on the other hand, are optional for all endpoints.

---

### Request types

#### RPC-style request
```Swift
_ = tester.files.createFolder(path: "/test/path/in/Dropbox/account").response { response, error in
    if let response = response {
        print(response)
    } else if let error = error {
        print(error)
    }
}
```

---

#### Upload-style request
```Swift
let fileData = "testing data example".data(using: String.Encoding.utf8, allowLossyConversion: false)!

let request = tester.files.upload(path: "/test/path/in/Dropbox/account", input: TestData.fileData)
    .response { response, error in
        if let response = response {
            print(response)
        } else if let error = error {
            print(error)
        }
    }
    .progress { progressData in
        print(progressData)
    }

// in case you want to cancel the request
if someConditionIsSatisfied {
    request.cancel()
}
```

---

#### Download-style request
```Swift
// Download to URL
let fileManager = FileManager.default
let directoryURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
let destURL = directoryURL.appendingPathComponent("myTestFile")
let destination: (URL, HTTPURLResponse) -> URL = { temporaryURL, response in
    return destURL
}
_ = tester.files.download(path: "/test/path/in/Dropbox/account", overwrite: true, destination: destination)
    .response { response, error in
        if let response = response {
            print(response)
        } else if let error = error {
            print(error)
        }
    }
    .progress { progressData in
        print(progressData)
    }


// Download to Data
_ = tester.files.download(path: "/test/path/in/Dropbox/account")
    .response { response, error in
        if let response = response {
            let responseMetadata = response.0
            print(responseMetadata)
            let fileContents = response.1
            print(fileContents)
        } else if let error = error {
            print(error)
        }
    }
    .progress { progressData in
        print(progressData)
    }
```

---

### Handling responses and errors

Dropbox API v2 deals largely with two data types: **structs** and **unions**. Broadly speaking, most route **arguments** are struct types and most route **errors** are union types.

**NOTE:** In this context, "structs" and "unions" are terms specific to the Dropbox API, and not to any of the languages that are used to query the API, so you should avoid thinking of them in terms of their Swift definitions.

**Struct types** are "traditional" object types, that is, composite types made up of a collection of one or more instance fields. All public instance fields are accessible at runtime, regardless of runtime state.

**Union types**, on the other hand, represent a single value that can take on multiple value types, depending on state. We capture all of these different type scenarios under one "union object", but that object will exist only as one type at runtime. Each union state type, or **tag**, may have an associated value (if it doesn't, the union state type is said to be **void**). Associated value types can either be primitives, structs or unions. Although the Swift SDK represents union types as objects with multiple instance fields, at most one instance field is accessible at runtime, depending on the tag state of the union.

For example, the [/delete](https://www.dropbox.com/developers/documentation/http/documentation#files-delete) endpoint returns an error, `Files.DeleteError`, which is a union type. The `Files.DeleteError` union can take on two different tag states: `path_lookup`
(if there is a problem looking up the path) or `path_write` (if there is a problem writing -- or in this case deleting -- to the path). Here, both tag states have non-void associated values (of types `Files.LookupError` and `Files.WriteError`, respectively).

In this way, one union object is able to capture a multitude of scenarios, each of which has their own value type.

To properly handle union types, you should pass each union through a switch statement, and check each possible tag state associated with the union. Once you have determined the current tag state of the union, you can then access the value associated with that tag state (provided there exists an associated value type, i.e., it's not **void**).

---

#### Route-specific errors
```Swift
_ = tester.files.delete(path: "/test/path/in/Dropbox/account").response { response, error in
    if let response = response {
        print(response)
    } else if let error = error {
        switch error as CallError {
        case .routeError(let boxed, let requestId):
            print("RouteError[\(requestId)]:")
            
            switch boxed.unboxed as Files.DeleteError {
            case .pathLookup(let lookupError):
                switch lookupError {
                case .notFound:
                    print("There is nothing at the given path.")
                case .notFile:
                    print("We were expecting a file, but the given path refers to something that isn't a file.")
                case .notFolder:
                    print("We were expecting a folder, but the given path refers to something that isn't a folder.")
                case .restrictedContent:
                    print("The file cannot be transferred because the content is restricted...")
                case .malformedPath(let malformedPath):
                    print("Malformed path: \(malformedPath)")
                case .other:
                    print("Unknown")
                }
            case .pathWrite(let writeError):
                print("WriteError: \(writeError)")
                // you can handle each `WriteError` case like the `DeleteError` cases above
            case .other:
                print("Unknown")
            }
        case .internalServerError(let code, let message, let requestId):
            ....
            ....
            // a not route-specific error occured
        ....
        ....
        ....
        }
    }
}
```

---

#### Generic network request errors

In the case of a network error, errors are either specific to the endpoint (as shown above) or more generic errors. 

To determine if an error is route-specific or not, the error object should be cast as a `CallError`, and depending on the type of error, handled in the appropriate switch statement. 

```Swift
_ = tester.files.delete(path: "/test/path/in/Dropbox/account").response { response, error in
    if let response = response {
        print(response)
    } else if let error = error {
        switch error as CallError {
        case .routeError(let boxed, let requestId):
            // a route-specific error occured
            // see handling above
            ....
            ....
            ....
        case .internalServerError(let code, let message, let requestId):
            print("InternalServerError[\(requestId)]: \(code): \(message)")
        case .badInputError(let message, let requestId):
            print("BadInputError[\(requestId)]: \(message)")
        case .authError(let authError, let requestId):
            print("AuthError[\(requestId)]: \(authError)")
        case .rateLimitError(let rateLimitError, let requestId):
            print("RateLimitError[\(requestId)]: \(rateLimitError)")
        case .httpError(let code, let message, let requestId):
            print("HTTPError[\(requestId)]: \(code): \(message)")
        case .clientError(let error):
            print("ClientError: \(error)")
        }
    }
}
```

---

#### Response handling edge cases

Some routes return union types as result types, so you should be prepared to handle these results in the same way that you handle union route errors. Please consult the [documentation](https://www.dropbox.com/developers/documentation/http/documentation)
for each endpoint that you use to ensure you are properly handling the route's response type.

A few routes return result types that are **datatypes with subtypes**, that is, structs that can take on multiple state types like unions.

For example, the [/delete](https://www.dropbox.com/developers/documentation/http/documentation#files-delete) endpoint returns a generic `Metadata` type, which can exist either as a `FileMetadata` struct, a `FolderMetadata` struct, or a `DeletedMetadata` struct.
To determine at runtime which subtype the `Metadata` type exists as, pass the object through a switch statement, and check for each possible class, with the result casted accordingly. See below:

```Swift
_ = tester.files.delete(path: "/test/path/in/Dropbox/account").response { response, error in
    if let response = response {
        switch response {
        case let fileMetadata as Files.FileMetadata:
            print("File metadata: \(fileMetadata)")
        case let folderMetadata as Files.FolderMetadata:
            print("Folder metadata: \(folderMetadata)")
        case let deletedMetadata as Files.DeletedMetadata:
            print("Deleted entity's metadata: \(deletedMetadata)")
        }
    } else if let error = error {
        switch error as CallError {
        case .routeError(let boxed, let requestId):
            // a route-specific error occured
            // see handling above
        case .internalServerError(let code, let message, let requestId):
            ....
            ....
            // a not route-specific error occured
            // see handling above
        ....
        ....
        ....
        }
    }
}
```

This `Metadata` object is known as a **datatype with subtypes** in our API v2 documentation.

Datatypes with subtypes are a way combining structs and unions. Datatypes with subtypes are struct objects that contain a tag, which specifies which subtype the object exists as at runtime. The reason we have this construct, as with unions, is so we can capture a multitude of scenarios with one object.

In the above example, the `Metadata` type can exists as `FileMetadata`, `FolderMetadata` or `DeleteMetadata`. Each of these types have common instances fields like "name" (the name for the file, folder or deleted type), but also instance fields that are specific to the particular subtype. In order to leverage inheritance, we set a common supertype called `Metadata` which captures all of the common instance fields, but also has a tag instance field, which specifies which subtype the object currently exists as.

In this way, datatypes with subtypes are a hybrid of structs and unions. Only a few routes return result types like this.

---

### Customizing network calls

#### Specify global response queue

By default, all response handler code is executed via the main queue (which makes UI updating convenient). However, if additional customization is necessary
(like handling responses on a custom queue), you can initialize your `DropboxClient` with a customized `DropboxTransportClient` in your application delegate. See below:

##### iOS
```Swift
import SwiftyDropbox

let transportClient = DropboxTransportClient(accessToken: "<MY_ACCESS_TOKEN>",
                                             baseHosts: nil,
                                             userAgent: "CustomUserAgent",
                                             selectUser: nil)
DropboxClientsManager.setupWithAppKey("<APP_KEY>", transportClient: transportClient)
```

##### macOS
```Swift
import SwiftyDropbox

let transportClient = DropboxTransportClient(accessToken: "<MY_ACCESS_TOKEN>",
                                             baseHosts: nil,
                                             userAgent: "CustomUserAgent",
                                             selectUser: nil)
DropboxClientsManager.setupWithAppKeyDesktop("<APP_KEY>", transportClient: transportClient)
```

#### Specify API call response queue

You can also set custom response queues on a API call-by-call basis in the response handler:

```Swift
let client = DropboxClientsManager.authorizedClient!

_ = client.files.listFolder(path: "").response(queue: DispatchQueue(label: "MyCustomSerialQueue")) { response, error in
    if let result = response {
        print(Thread.current)  // Output: <NSThread: 0x61000007bec0>{number = 4, name = (null)}
        print(Thread.main)     // Output: <NSThread: 0x608000070100>{number = 1, name = (null)}
        print(result)
    }
}
```

---

### `DropboxClientsManager` class

The Swift SDK includes a convenience class, `DropboxClientsManager`, for integrating the different functions of the SDK into one class.

#### Single Dropbox user case

For most apps, it is reasonable to assume that only one Dropbox account (and access token) needs to be managed at a time. In this case, the `DropboxClientsManager` flow looks like this: 

* call `setupWithAppKey`/`setupWithAppKeyDesktop` (or `setupWithTeamAppKey`/`setupWithTeamAppKeyDesktop`) in integrating app's app delegate
* client manager determines whether any access tokens are stored -- if any exist, one token is arbitrarily chosen to use
* if no token is found, call `authorizeFromController`/`authorizeFromControllerDesktop` to initiate the OAuth flow
* if auth flow is initiated, call `handleRedirectURL` (or `handleRedirectURLTeam`) in integrating app's app delegate to handle auth redirect back into the app and store the retrieved access token (using a `DropboxOAuthManager` instance)
* client manager instantiates a `DropboxTransportClient` (if not supplied by the user)
* client manager instantiates a `DropboxClient` (or `DropboxTeamClient`) with the transport client as a field

The `DropboxClient` (or `DropboxTeamClient`) is then used to make all of the desired API calls.

* call `unlinkClients` to logout Dropbox user and clear all access tokens

#### Multiple Dropbox user case

For some apps, it is necessary to manage more than one Dropbox account (and access token) at a time. In this case, the `DropboxClientsManager` flow looks like this: 

* access token uids are managed by the app that is integrating with the SDK for later lookup
* call `setupWithAppKeyMultiUser`/`setupWithAppKeyMultiUserDesktop` (or `setupWithTeamAppKeyMultiUser`/`setupWithTeamAppKeyMultiUserDesktop`) in integrating app's app delegate
* client manager determines whether an access token is stored with the`tokenUid` as a key -- if one exists, this token is chosen to use
* if no token is found, call `authorizeFromController`/`authorizeFromControllerDesktop` to initiate the OAuth flow
* if auth flow is initiated, call `handleRedirectURL` (or `handleRedirectURLTeam`) in integrating app's app delegate to handle auth redirect back into the app and store the retrieved access token (using a `DropboxOAuthManager` instance)
* at this point, the app that is integrating with the SDK should persistently save the `tokenUid` from the `DropboxAccessToken` field of the `DropboxOAuthResult` object returned from the `handleRedirectURL` (or `handleRedirectURLTeam`) method
* `tokenUid` can be reused either to authorize a new user mid-way through an app's lifecycle via `reauthorizeClient` (or `reauthorizeTeamClient`) or when the app initially launches via `setupWithAppKeyMultiUser`/`setupWithAppKeyMultiUserDesktop` (or `setupWithTeamAppKeyMultiUser`/`setupWithTeamAppKeyMultiUserDesktop`)
* client manager instantiates a `DropboxTransportClient` (if not supplied by the user)
* client manager instantiates a `DropboxClient` (or `DropboxTeamClient`) with the transport client as a field

The `DropboxClient` (or `DropboxTeamClient`) is then used to make all of the desired API calls.

* call `resetClients` to logout Dropbox user but not clear any access tokens
* if specific access tokens need to be removed, use the `clearStoredAccessToken` method in `DropboxOAuthManager`

---

## Examples

* [PhotoWatch](https://github.com/dropbox/PhotoWatch) - View photos from your Dropbox. Supports Apple Watch.

---

## Documentation

* [Dropbox API v2 Swift SDK](http://dropbox.github.io/SwiftyDropbox/api-docs/latest/)
* [Dropbox API v2](https://www.dropbox.com/developers/documentation/http/documentation)

---

## Stone

All of our routes and data types are auto-generated using a framework called [Stone](https://github.com/dropbox/stone).

The `stone` repo contains all of the Swift specific generation logic, and the `spec` repo contains the language-neutral API endpoint specifications which serve
as input to the language-specific generators.

---

## Modifications

If you're interested in modifying the SDK codebase, you should take the following steps:

* clone this GitHub repository to your local filesystem
* run `git submodule init` and then `git submodule update`
* navigate to `TestSwifty_[iOS|macOS]` and run `pod install`
* open `TestSwifty_[iOS|macOS]/TestSwifty_[iOS|macOS].xcworkspace` in Xcode
* implement your changes to the SDK source code.

To ensure your changes have not broken any existing functionality, you can run a series of integration tests by
following the instructions listed in the `ViewController.m` file.

---

## Bugs

Please post any bugs to the [issue tracker](https://github.com/dropbox/SwiftyDropbox/issues) found on the project's GitHub page.
  
Please include the following with your issue:
 - a description of what is not working right
 - sample code to help replicate the issue

Thank you!

