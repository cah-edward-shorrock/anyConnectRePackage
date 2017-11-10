# Cisco AnyConnect Selective Install
### Introduction
It is often necessary to selectively install only some of the packages contained in the Cisco AnyConnect package. Below is a quick gist to get you going.

### Setup
You will need a copy of the cisco package and a mac running 10.10 or newer. For these examples lets assume you have a package called:

```
Anyconnect.pkg
```

And that this package is located on your desktop.

```
~/Desktop/AnyConnect.pkg
```

### Break the PKG Open

```shell
pkgutil --expand ~/Desktop/Anyconnect.pkg ~/Desktop/Anyconnect
```

You will now have a folder on your desktop called Anyconnect

Open this folder up and locate the file called Distribution. This distribution file defines how the installer behaves with regard to what screens are shown and what choices the user is offered.

### Edit the Distribution file
You will want to open the Distribution in a plain text editor like Atom, Sublime or Vi. I used Vi since its on every mac:

```shell
vi ~/Desktop/Anyconnect/Distribution
```

Press 'i' to enter insert mode if you used vi.

You are now greeted with an XML file. The first section defines the license and background image. After that we get to list of the actual choices. You can remove some if you want but I recomend leaving it along in case you need to manually install some later.

```xml
<choices-outline>
    <line choice="choice_vpn"/>
    <line choice="choice_websecurity"/>
    <line choice="choice_fireamp"/>
    <line choice="choice_dart"/>
    <line choice="choice_posture"/>
    <line choice="choice_iseposture"/>
    <line choice="choice_nvm"/>
    <line choice="choice_umbrella"/>
</choices-outline>
```

After this we come to the individual choice blocks. The first block for VPN is a litte different from the others so we will edit it first.

```xml
<choice id="choice_vpn" start_enabled="enable_module()" enabled="choice_vpn_enabled()" start_selected="check_upgrade()" selected="choice_vpn_selected()" title="VPN" description="Installs the module that enables VPN capabilities.">
    <pkg-ref id="com.cisco.pkg.anyconnect.vpn"/>
</choice>
```
In my case I need VPN enabled so no editing is needed as it is enabled by default. However if you wanted to disable remove the start_selected block as its optional and change selected from choice_vpn_selected to false. In general I would leave the start_enabled and enabled set to the default values.

Now lets look at the rest of the blocks as they are all identical except for the fact that they each represent one of the other packages contained in the package.

```xml
<choice id="choice_websecurity" start_selected="check_upgrade()" start_enabled="enable_module()" title="Web Security" description="Installs the WebSecurity module that enables cloud scanning of web content to protect against malware and enforce acceptable use policies via the ScanSafe cloud proxies.">
    <pkg-ref id="com.cisco.pkg.anyconnect.websecurity_v2"/>
</choice>
```

All we need to do to ensure this is not checked by default is change the start_selected block to selected and set it to false, like so:

```xml
<choice id="choice_websecurity" selected="false" start_enabled="enable_module()" title="Web Security" description="Installs the WebSecurity module that enables cloud scanning of web content to protect against malware and enforce acceptable use policies via the ScanSafe cloud proxies.">
    <pkg-ref id="com.cisco.pkg.anyconnect.websecurity_v2"/>
</choice>
```

Repeat this for all of the blocks that you do not want to be selected by default.

###### Note: You can still manually select the packages. This is why we did not change their enabled flags. But they will not install as part of an automated package pushed through munki or Jamf Pro etc.

Alright now save your file. It should not have any file extension. If you used vi you would:

```
Press Esc
Press Shift + ;
Type wq
Press Return
```

### Re-Package Anyconnect

So now that our installer is configured the way we want we need to flatten the package.

```shell
pkgutil --flatten ~/Desktop/Anyconnect ~/Desktop/companyAnyConnect.pkg
```

This will create an installer package that will install just the options you enabled or disabled!

### Re-Sign the Package

One thing is that since we edited the package it will not be signed anymore. If you have a Developer ID Installer certificate from Apple you can re-sign the package:

```shell
productsign --sign "Developer ID Installer:" ~/Desktop/companyAnyConnect.pkg ~/Desktop/companyAnyConnect_signed.pkg
```

### Conclusion

That just about wraps it up. If you are curious about the other options in the Distribution file you can find more about them in Apple's developer documentation by clicking [here](https://developer.apple.com/library/content/documentation/DeveloperTools/Reference/DistributionDefinitionRef/Chapters/Distribution_XML_Ref.html).
