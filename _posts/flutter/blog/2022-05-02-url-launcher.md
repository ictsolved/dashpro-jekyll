---
layout: post
title: Make Call, Send SMS or Email, or Open URL in Flutter
description: Want to make a call, send SMS or an email, or open a website from Flutter application? Here is the solution.
category:
  - Flutter
  - Blog
img: "assets/images/blog/Flutter URL Launcher.png"
---

Want to make a call, send SMS or an email, or open a website from Flutter application? This recipe will demonstrate how to use those functionalities in Flutter.

There is an official plugin from the Flutter team called [`url_launcher`](https://pub.dev/packages/url_launcher) to perform these actions. It supports all the platforms.

| Platform  | Android | iOS  | Linux | macOS  | Web | Windows     |
| --------- | ------- | ---- | ----- | ------ | --- | ----------- |
| Supported | SDK 16+ | 9.0+ | Any   | 10.11+ | Any | Windows 10+ |

Add the `url_launcher` plugin to `pubspec.yaml` file with following command in terminal:

```bash
flutter pub add url_launcher
```

This command will add the `url_launcher` plugin to your `pubspec.yaml` file and run the `flutter pub get` command to download the package. Now the `pubspec.yaml` file should look like following:

```yaml
dependencies:
  flutter:
    sdk: flutter
  url_launcher: ^6.1.0
```

## Platform Specific Configurations

### iOS

To launch the URL schemes, we need to add them in the `Info.plist` file as shown below:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>tel</string>
  <string>sms</string>
  <string>mailto</string>
  <string>http</string>
  <string>https</string>
</array>
```

> The above example includes multiple URL schemes. Make sure to add only the scheme required by your application.

### Android

Android requires package visibility configuration in the `AndroidManifest.xml` file starting from API 30. A `<queries>` element must be added to the manifest file as a child of the root element to function as intended. Our manifest file should look like following:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
<!-- Other Elements -->
<queries>
<!-- Open https URL -->
  <intent>
    <action android:name="android.intent.action.VIEW" />
    <data android:scheme="https" />
  </intent>
<!-- Make Call -->
  <intent>
    <action android:name="android.intent.action.DIAL" />
    <data android:scheme="tel" />
  </intent>
<!-- Send SMS -->
  <intent>
    <action android:name="android.intent.action.SENDTO" />
    <data android:scheme="smsto" />
  </intent>
<!-- Send Email -->
  <intent>
    <action android:name="android.intent.action.SEND" />
    <data android:mimeType="*/*" />
  </intent>
</queries>
<!-- Other Elements -->
</manifest>
```

Similar to the iOS configuration, the above example includes multiple URL schemes, but we need to add only the scheme required by our application. Let's see an example below to open an URL.

#### Import the plugin

```dart
import 'package:url_launcher/url_launcher.dart';
```

#### Create a button

```dart
ElevatedButton(
    onPressed: _launchUrl,
    child: const Text('Open URL'),
)
```

#### Launch URL inside `_launchUrl` function

```dart
void _launchUrl() async {
    try {
      final Uri? uri = Uri.tryParse('https://megadash.dev');
      if (uri != null) {
        bool canHandle = await canLaunchUrl(uri);
        if (canHandle) {
          bool launched = await launchUrl(uri);
          if (!launched) {
            await _showMessage('Failed to open URL');
          }
        } else {
          await _showMessage('Device cannot handle URL');
        }
      } else {
        await _showMessage('Invalid URL');
      }
    } on Exception {
      await _showMessage('Something Wrong!');
    }
}

Future<void> _showMessage(String message) => showDialog(
    context: context,
    builder: (context) => AlertDialog(
        title: const Text('Failed'),
        content: Text(message),
    ),
);
```

A `mode` property with value `LaunchMode.externalApplication` can be passed to the `launchUrl` function to open the web URL in an external browser instead of WebView on iOS and Android.

Here is a complete example that implements functionality to make phone call, send sms and email and open URL by taking an input from the user.

```dart
import 'package:flutter/material.dart';
import 'package:url_launcher/url_launcher.dart';

void main() {
  runApp(const MegaDash());
}

class MegaDash extends StatelessWidget {
  const MegaDash({Key? key}) : super(key: key);

  String get title => 'Mega Launcher';

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: title,
        debugShowCheckedModeBanner: false,
        home: MegaLauncher(title: title),
      );
}

enum LaunchType { sms, phone, url, email }

class MegaLauncher extends StatefulWidget {
  const MegaLauncher({required this.title, Key? key}) : super(key: key);

  final String title;

  @override
  State<MegaLauncher> createState() => _MegaLauncherState();
}

class _MegaLauncherState extends State<MegaLauncher> {
  final TextEditingController _emailController = TextEditingController();
  final GlobalKey<FormState> _emailKey = GlobalKey<FormState>();
  final TextEditingController _telephoneController = TextEditingController();
  final GlobalKey<FormState> _telephoneKey = GlobalKey<FormState>();
  final TextEditingController _urlController = TextEditingController();
  final GlobalKey<FormState> _urlKey = GlobalKey<FormState>();

  String? encodeQueryParameters(Map<String, String> params) => params.entries
      .map(
        (e) => '${Uri.encodeComponent(e.key)}=${Uri.encodeComponent(e.value)}',
      )
      .join('&');

  Future<void> showFailedDialog(String message) => showDialog(
        context: context,
        builder: (context) => AlertDialog(
          title: const Text('Failed'),
          content: Text(message),
        ),
      );

  void openUrl(Uri uri) async {
    try {
      bool canHandle = await canLaunchUrl(uri);
      if (canHandle) {
        bool launched = await launchUrl(uri);
        if (!launched) {
          await showFailedDialog('Failed to launch URL');
        }
      } else {
        await showFailedDialog('Cannot open URL');
      }
    } on Exception {
      await showFailedDialog('Something Wrong!');
    }
  }

  void parseUrl({
    required String url,
    required LaunchType type,
    required GlobalKey<FormState> formKey,
  }) {
    bool isFormValid = formKey.currentState?.validate() ?? false;
    if (isFormValid) {
      switch (type) {
        case LaunchType.sms:
          {
            final Uri smsUri = Uri(
              scheme: 'sms',
              path: url,
              query:
                  encodeQueryParameters({'body': 'This is a test sms message'}),
            );
            openUrl(smsUri);
          }
          break;
        case LaunchType.phone:
          {
            final Uri phoneUri = Uri(
              scheme: 'tel',
              path: url,
            );
            openUrl(phoneUri);
          }
          break;
        case LaunchType.url:
          {
            final Uri? urlUri = Uri.tryParse(url);
            if (urlUri != null) {
              openUrl(urlUri);
            } else {
              showFailedDialog('Invalid URL');
            }
          }
          break;
        case LaunchType.email:
          {
            final Uri emailUri = Uri(
              scheme: 'mailto',
              path: url,
              query: encodeQueryParameters(
                  {'subject': 'Test Email', 'body': 'This is a test email'}),
            );
            openUrl(emailUri);
          }
          break;
      }
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: Text(widget.title),
        ),
        body: SingleChildScrollView(
          child: Column(
            children: [
              Column(
                children: [
                  MegaTextField(
                    controller: _telephoneController,
                    label: 'Telephone Number',
                    formKey: _telephoneKey,
                  ),
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      ElevatedButton(
                        child: const Text('Make Call'),
                        onPressed: () => parseUrl(
                          url: _telephoneController.text,
                          type: LaunchType.phone,
                          formKey: _telephoneKey,
                        ),
                      ),
                      Padding(
                        padding: const EdgeInsets.only(left: 16),
                        child: ElevatedButton(
                          child: const Text('Send SMS'),
                          onPressed: () => parseUrl(
                            url: _telephoneController.text,
                            type: LaunchType.sms,
                            formKey: _telephoneKey,
                          ),
                        ),
                      ),
                    ],
                  ),
                  MegaTextField(
                    controller: _emailController,
                    label: 'Email Address',
                    formKey: _emailKey,
                  ),
                  ElevatedButton(
                    child: const Text('Send Email'),
                    onPressed: () => parseUrl(
                      url: _emailController.text,
                      type: LaunchType.email,
                      formKey: _emailKey,
                    ),
                  ),
                  MegaTextField(
                    controller: _urlController,
                    label: 'URL',
                    formKey: _urlKey,
                  ),
                  ElevatedButton(
                    child: const Text('Open URL'),
                    onPressed: () => parseUrl(
                      url: _urlController.text,
                      type: LaunchType.url,
                      formKey: _urlKey,
                    ),
                  ),
                ],
              ),
            ],
          ),
        ),
      );
}

class MegaTextField extends StatelessWidget {
  const MegaTextField({
    required this.controller,
    required this.formKey,
    required this.label,
    Key? key,
  }) : super(key: key);

  final TextEditingController controller;
  final GlobalKey<FormState> formKey;
  final String label;

  String? validator(value) {
    if (value == null || value.isEmpty) {
      return '$label Required';
    }

    return null;
  }

  @override
  Widget build(BuildContext context) => Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: formKey,
          child: TextFormField(
            controller: controller,
            decoration: InputDecoration(
              labelText: label,
              border: const OutlineInputBorder(),
            ),
            validator: validator,
            autovalidateMode: AutovalidateMode.onUserInteraction,
          ),
        ),
      );
}
```
