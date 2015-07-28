# Push Plugin for NativeScript

The code for the Push Plugin for NativeScript.

## Setup

- Create a new NativeScript application

		tns create MyApp

	or use an existing one.

- Add the Push Plugin. This will install the push plugin in node_module, in the root of the project. When adding a new platform (or using an existing one) the plugin will be added there as well.

		tns plugin add C:\nativescript-push\push-plugin

### Android

- Go to the application folder and add the Android platform to the application

		tns platform add android

- Add google play services, as GCM is part of it. It's present in the android-sdk. Add it like this:

		tns library add android C:\Users\your_user_name\AppData\Local\Android\android-sdk\extras\google\google_play_services\libproject\google-play-services_lib\libs


- Add sample code in main-view-model.js like this one to subscribe and receive messages (Enter your google project id in the options of the register method):

```javascript
	var pushPlugin = require("push-plugin");
	var self = this;
	pushPlugin.register({ senderID: 'your-google-project-id' }, function (data){
		self.set("message", "" + JSON.stringify(data));
	}, function() { });
	
	pushPlugin.onMessageReceived(function callback(data) {	
		self.set("message", "" + JSON.stringify(data));
	});
```

- Attach your phone to the PC, ensure "adb devices" command lists it and run the app on the phone:

		tns run android

### iOS

- Go to the application folder and add the iOS platform to the application

        tns platform add ios

- Add sample code in main-view-model.js like this one to subscribe and receive messages (Enter your google project id in the options of the register method):

```javascript
	var pushPlugin = require("push-plugin");
	var self = this;
	var iosSettings = {
		iOS: {
	    	badge: true,
	        sound: true,
	        alert: true,
	        interactiveSettings: {
	        	actions: [{
	            	identifier: 'READ_IDENTIFIER',
	                title: 'Read',
	                activationMode: "foreground",
	                destructive: false,
	                authenticationRequired: true
	           	}, {
	            	identifier: 'CANCEL_IDENTIFIER',
	                title: 'Cancel',
	                activationMode: "foreground",
	                destructive: true,
	                authenticationRequired: true
	            }],
	            categories: [{
					identifier: 'READ_CATEGORY',
	                actionsForDefaultContext: ['READ_IDENTIFIER', 'CANCEL_IDENTIFIER'],
	                actionsForMinimalContext: ['READ_IDENTIFIER', 'CANCEL_IDENTIFIER']
				}]
			}
		},
	    notificationCallbackIOS: function (data) {
	    	self.set("message", "" + JSON.stringify(data));
	    }
	};
	
	pushPlugin.register(iosSettings, function (data) {
		self.set("message", "" + JSON.stringify(data));
	}, function() { });
```
 
- Open the yourApp.xcodeproj file which has been generated for you by NativeScript in platforms/ios, set the correct bundle identifier and configure your push certificates (i.e. select a Team)
- Edit the package.json file in the root of application, by changing the bundle identifier to match the one set in the xcodeproject. For example:
        "id": "com.telerik.PushNotificationApp"

NOTE: If you encounter error like this: "Error registering: no valid 'aps-environment' entitlement string found for application" - this means that the certificates are not correctly set in the xcodeproject. Fix them and try again. The bundle identifier in xcode should be the same as the "id" in the package.json file in the root of the project.


## Troubleshooting

In case the application doesn't work as expected. Here are some things you can verify

### Android

- Ensure that the AndroindManifest.xml located at platforms\android contains the following snippets for registering the GCM listener:

```xml
	<activity android:name="com.telerik.pushplugin.PushHandlerActivity"/>
	<receiver
		android:name="com.google.android.gms.gcm.GcmReceiver"
	    android:exported="true"
	    android:permission="com.google.android.c2dm.permission.SEND" >
	    <intent-filter>
	    	<action android:name="com.google.android.c2dm.intent.RECEIVE" />
	        <category android:name="com.pushApp.gcm" />
	    </intent-filter>
	</receiver>
	<service
		android:name="com.telerik.pushplugin.PushPlugin"
	    android:exported="false" >
	    <intent-filter>
	    	<action android:name="com.google.android.c2dm.intent.RECEIVE" />
	    </intent-filter>
	</service>
```

- Ensure that the AndroindManifest.xml located at platforms\android contains the following permissions for push notifications:

```xml
	<uses-permission android:name="android.permission.GET_ACCOUNTS" />
	<uses-permission android:name="android.permission.WAKE_LOCK" />
	<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
``` 
