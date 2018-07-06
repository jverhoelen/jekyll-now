---
layout: post
title: "How to fix DevCenter freezing on new Java versions"
category: "Java"
---

Shortly after updating my system Java version to `8.0.172` I realised that Datastax DevCenter was not working anymore. It just freezed immediately after loading and didn't react to input anymore.

Fortunately there was not a long time between the update and my next need to run Cassandra queries. So I tried changing the Java version for DevCenter to an older one to make it compatible again.

## SDKMAN! to the rescue

SDKMan is a general purpose SDK manager that has a bunch of prominent SDKs in its portfolio. There are Java, Scala, Spark, Spring Boot Groovy and many more available. First [install SDKMan from their instructions](https://sdkman.io/install). Once installed, it tells you to open another shell window or to load the sources in the current one. After you have done that, you can install all SDKs you need.

To have an older Java version available, let's get an available one from Java 7 that SDKMan suggests you with `sdk list java`. For me this was for example `7.0.181-zulu`, feel free to pick Oracle if you like. Install it with `sdk install java 7.0.181-zulu`.

To have a new Java version (I currently need Java 8), pick a newer one executing `sdk install java 8.0.172-zulu` and pick it as default as SDKMan prompts for that.

These two Java versions should be located in `~/.sdkman/candidates/java/<version>` now. So your Java binary for the older one is now for example `.sdkman/candidates/java/7.0.181-zulu/bin/java`.

## Tell DevCenter to use that version

DevCenter by default decides for himself what Java version it uses and probably just takes your linked `java` binary. Do the following:

* Right-click on `DevCenter.app` in your properties directory
* Click "Show package contents"
* Open `Contents/info.plist` in an editor of your choice

Then, at the bottom you will find configuration that looks like the following:

```
<dict>
    <!-- a lot of stuff above -->
    <key>Eclipse</key>
	<array>
        <!-- comments on how to change the Java version -->
        <string>-keyring</string><string>~/.eclipse_keyring</string>
		<string>-showlocation</string>
        <!-- some more comments -->
    </array>
</dict>
```

Into the `array` element, insert the following children and make sure to adapt the old Java version you want DevCenter to use and your users name.

```
<string>-vm</string><string>/Users/YOUR-USER/.sdkman/candidates/java/7.0.181-zulu/bin/java</string>
```

Now exit and start again your DevCenter. Voilà, it should be working again!
The credits for solving the root problem go to [this Stackoverflow answer](https://stackoverflow.com/a/49617513/5500928).
