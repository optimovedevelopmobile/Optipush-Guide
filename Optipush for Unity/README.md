 - [Introduction](#Introduction)
 - [Setup](#Setup)
    - [Pre-Requisites ](#pre-reqs)
    - [Optipush Configuration](#configuration)
    - [Deep Linking](#deep%20linking)
    - [Enabling Test Mode](#test%20mode) 
 - [Post-Setup](#Post%20setup)
    - [Create & test notification templates](#notification%20template) 
    - [Set up an Optipush campaign](#Optipush%20campaign) 
 
<br>

# <a id="Introduction"></a>Introduction
**Optipush** is Optimove's mobile push notification delivery add-in module, powering all aspects of preparing, delivering and tracking mobile push notification communications to customers, seamlessly from withinOptimove.<br>
**Optimove SDK for Unity** includes built-in functionality for receiving push messages, presenting notifications in the app UI and tracking user responses.

# <a id="Setup"></a>Setup

## <a id="pre-reqs"></a>Pre-Requisites 

1. [Optimove Mobile SDK for Unity](https://github.com/optimove-tech/Unity-SDK-Integration-Guide) implemented 

## <a id="configuration"></a>Optipush Configuration

### iOS Build

### Notification Service Extension

To allow Optimove to present and track incoming push notifications, you'll need to add a **Notification Extension** to your project. <br>
Since OptimoveUnitySDK.unitypackage contains a `PostProcees` execution script that add this extension among with it's capabilities, you as a developer should not do anything actively, but just verify that the `AppGroup` of the extension doesn't have any errors, and that the `Deployment Target` (minimum iOS version) of the _Notification Extension target_ is the same as the _application target_. 

### User Notification Center Delegate
The _Optimove SDK for Unity_ automatically sets the `UNUserNotificationCenterDelegate` property of The `UNUserNotificationCenter`.<br>
Since there can have only one such property, if your app also sets a `UNUserNotificationCenterDelegate` you must propagate 2 `UNUserNotificationCenterDelegate` callbacks to the Optimove SDK for Unity from the generated XCode project as follow:

1. `OptimoveSdkBridge willPresentWithNotification:withCompletionHandler:`
2. `OptimoveSdkBridge didReceiveWithResponse:withCompletionHandler:`
 
### Notification Presentation Authorization

To allow the SDK to present OptiPush notifications to the end user, the user must grant the appropriate permission.
Whenever the flow of your app deems it appropriate, ask to grant the notification permission by calling the `NotificationServices.RegisterForNotifications()` function With the desired types of notifications you'd want the user to receive.
For example:

```c#
#if UNITY_IOS
using NotificationServices = UnityEngine.iOS.NotificationServices;
using NotificationType = UnityEngine.iOS.NotificationType;
#endif

public class MyGameObject : MonoBehaviour
{
    private void Start()
    {
#if UNITY_IOS
        NotificationServices.RegisterForNotifications(NotificationType.Alert | NotificationType.Badge | NotificationType.Sound);
#endif
    }
}
```

> For more information about the `UserNotifications` framework see [Apple's Documentation](https://developer.apple.com/documentation/usernotifications)

## <a id="deep linking"></a>Deep Linking

### Setting up Optipush Deep Linking

To route users back to a specific screen inside the application from by opening an Optipush notification, you must support _Deep Linking_.
Other than _UI attributes_, an **Optipush Notification** can contain metadata linking to a specific screen within your application, along with custom (screen specific) data.<br>

From any `Scene` within your app (the very first `Scene` is recommended), create an `OptimoveDeepLinkCallback`. That callback receives the name of the targeted screen. Then, pass the callback to the `OptimoveUnitySdk`, which in turn will activate the callback as soon as the deep link has been detected (during app open).

Example:
```c#
public class MyGameObject : MonoBehaviour
{
    // Pass the OptimoveDeepLinkCallback to the SDK as soon as the scene is loaded
    private void Awake()
    {
        OptimoveUnitySdk.RegisterDeepLinkCallback((screenName) =>
        {
            // Use the screenName to navigate to the correct screen within your app
        });
    }
}
```

### Android Only

#### Setting up Optipush Deep Linking

The Optimove SDK for Unity is delivered with a custom `OptimoveUnityPlayerActivity` that is set via a custom `Plugins/Android/AndroidManifest.xml`. To support new deep links add the following `intent-filter` to the `<activity android:name="com.optimove.sdk.unity_native_plugin.OptimoveUnityPlayerActivity">` (Replace with your app package and screen name):

```xml
 <intent-filter>
    <action android:name="android.intent.action.VIEW"/>

    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
        
    <!--The host must match the app's package-->
    <!--The pathPrefix must match the screen's name as defined on the Optimove site-->
    <data 
        android:host="REPLACE.WITH.THE.APP.PACKAGE"  
        android:pathPrefix="/REPLACE_WITH_A_CUSTOM_SCREEN_NAME" 
        android:scheme="http"/>
</intent-filter>
```

#### The Hosting Application Specifies Custom Manifest

The _Optimove SDK for Unity_ is delivered with a custom _Android Manifest_ file that sets `OptimoveUnityPlayerActivity` as the root (and only) `Activity`. If your app uses another _Activity_ as the root _Activity_ then you can either: 
1. Inherit from `OptimoveUnityPlayerActivity` 

```java
public class MyUnityPlayerActivity extends OptimoveUnityPlayerActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    // If you implement this method, don't forget to call the super class's callback impl
    super.onCreate(savedInstanceState);
  }

  @Override
  protected void onNewIntent(Intent intent) {
    // If you implement this method, don't forget to call the super class's callback impl
    super.onNewIntent(intent);
  }
}
```

OR

2. Notify Optimove's _UnityNativePlugin_ about the `onCreate` `onNewIntent` callbacks:

```java
public class MyUnityPlayerActivity extends UnityPlayerActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Propagate the callback to to the Optimove SDK
    new DeepLinkHandler(getIntent()).extractLinkData(new DeepLinkMarshallingHandler());
  }

  @Override
  protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);
    // Propagate the callback to to the Optimove SDK
    new DeepLinkHandler(intent).extractLinkData(new DeepLinkMarshallingHandler());
  }
}
```

<br>

## <a id="test mode"></a>Enabling Test Mode

You can test an **Optipush template** on your device *before* having to create an **Optipush campaign**.<br>
To enable _"test campaigns"_ on one or more devices, call the `OptimoveUnitySdk.StartOptipushTestMode()` method. To stop receiving "test campaigns" call `OptimoveUnitySdk.StopOptipushTestMode()`.

>- It is recommended to maintain 2 apps - one with test mode enabled (for internal purposes ONLY) and one without test mode enabled (for the general public).
>- The app that is published to the App Store must NOT have the test mode enabled.

<br>
 
# <a id="Post setup"></a>Post-Setup

## <a id="notification template"></a>Create & Test notification templates

Once Optimove has enabled Optipush as an execution channel for your Optimove instance, you can begin creating and testing your first Optipush template.
>Note: In order to be able to test your templates, the test mode must be [enabled](https://github.com/optimove-tech/A/tree/master/O/O%20for%20A#enabling-test-mode) within your mobile app.<br>

### Create  an Optipush Template
1. Go to the Manage Templates page and choose 'Optipush' from the Channel drop-down menu. 
2. Enter values for the following fields:
    - Template Name- Name te template 
    - Template Title - Title of push template
    - Message - Message of the template <br>
![](https://raw.githubusercontent.com/optimove-tech/Optipush-Guide/master/Opitpush%20for%20iOS/1.png)
3. Personalization - you can personalize the notification by adding dynamic tags and emojis to the notification. 
4. Preview - you can preview the push template notification before sending it out. 
5. **Deep links** (Optional) - choose the app (iOS) and select the target screen in your app that you want your customers to be directed to when opening the message. <br>

>Notes:
>- In order to add Deep Links to the drop-down menu, please send the list of screen names to your CSM that you have configured in your app as described [here(https://github.com/optimove-tech/A/tree/master/O/O%20for%20iOS#deep-linking).
>- *If a Deep Link is not chosen, the customer will be directed to the main screen when opening the message.* 
>- When creating templates for Optipush, if the template is targeted for a specific device (iOS/Android), it is recommended to add the device name to the template naming convention. This way it will be identifiablewhen choosing a template for a campaign targeting a specific device.<br>

### Test an Optipush Template
1. **Validate** - validates the push notification template to make sure all mandatory fields are completed and contain no errors. <br>
2. **Send Test**  - clicking this link will send the push notification template to all devices that have the app installed with the test mode enabled.<br>


## <a id="Optipush campaign"></a>Set up an Optipush campaign

### Run Campaign

Please follow these steps in order to run a pre-scheduled campaign via execution channel **Optipush**.
1. From the main menu go on *One-to-One Campaigns* --> click on *More* from the drop-down menu --> click on ***Run Campaign***.  
2. Go through Steps 1 & 2 of the Run Campaign wizard as you would for any campaign. 
3. In Step 3 (Execution Details) choose from the *Channel* drop-down menu *Optipush*. This action will open the **Optipush Options** window.
![](https://raw.githubusercontent.com/optimove-tech/Optipush-Guide/master/Opitpush%20for%20iOS/2_a.png)
5. Choose from the *App* drop-down menu if you would like to run the campaign for your iOS app, Android app, or both by selecting the relevant box(es).
6. Choose the relevant template for the *Template* drop down menu that you would like the targeted audience to receive.
7. Continue through the remaining steps of the Run Campaign wizard to schedule the campaign for your preferred dates and times.
