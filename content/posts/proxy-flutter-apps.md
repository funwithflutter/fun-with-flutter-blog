---
title: "Proxy Flutter Apps"
slug: proxy-flutter-apps
description: null
date: 2019-11-28T09:37:10+07:00
type: blog
draft: false
categories:
- General
tags:
- tools
- proxy
- networking
thumbnail: "proxy_app_traffic.png"
description: "How to proxy Flutter network traffic"
# series:
# -
---
{{< youtube mnkNvcEYxdg >}}
From hacking, debugging, developing, or to tickle some curiosity - there are a variety of reasons that you might want to inspect the underlying web traffic of an application. This article discusses a couple of possible solutions for Flutter applications, with a focus on proxying.

The tools and techniques are from a developers perspective, not a security researcher, as most of the techniques listed below will require modification to the source code. If you are looking at an app from a black box perspective I would suggest the following article:
https://blog.nviso.be/2019/08/13/intercepting-traffic-from-android-flutter-applications/

Before we can proxy our application we need to understand some basics regarding HTTPS and SSL certificates. If you are modifying source code to intercept traffic it is important you understand what you are doing from a security perspective.

## Note ##

This blog post was written as a supplement to the video provided above. It serves to provide additional information along with summaries and links to relevant resources.

## Other Networking Tools ##

Below are a couple of tools that you can use "out of the box" to inspect your network traffic. I'm listing them here to give you options, but in my personal opinion, they're only useful up until a certain point. If they provide you with what you want, then great.

[Flutter Stetho](https://pub.dev/packages/flutter_stetho)
> A plugin that connects Flutter to the Chrome Dev Tools on Android devices via the [Stetho Android Library]

[Alice](https://pub.dev/packages/alice)
> Alice is an HTTP Inspector tool for Flutter which helps debugging http requests. It catches and stores HTTP requests and responses, which can be viewed via simple UI. It is inspired from [Chuck](https://github.com/jgilfelt/chuck).

[Flipper](https://pub.dev/packages/flutter_flipperkit)
> [Flipper](https://fbflipper.com/) is a desktop app from Facebook that allows you to debug iOS and Android apps.

These tools instrument your application and use a variety of different techniques to log all your application traffic. It then forwards that information to some GUI client where you can inspect the traffic. Alice's GUI is baked directly into your Flutter application. Stetho forwards it to the Chrome Dev Tools (most convenient in my opinion). While Flipper requires a desktop client to connect to.

Then we have the [Dart DevTools](https://flutter.dev/docs/development/tools/devtools/overview), which can do a lot, however at the time of writing it does not have a network profiler. See this [feature request](https://github.com/flutter/devtools/issues/368) for more info. If the Dart DevTools has a network profiler by the time you are reading this then I would ignore the above options and use the DevTools.

## Proxying and HTTPS ##

The rest of this article will be all about proxying and what you need to know before you proxy an application.

### What is a proxy ###

A proxy, or a Proxy Server, is:
> a server (a computer system or an application) that acts as an intermediary for requests from clients seeking resources from other servers.

So imagine we have our Flutter application (A) and our web server (B). To communicate with our server the traffic flows like this:

A -> B.

Now let's put a proxy tool in the middle of those two (C):

A -> C -> B

> It's easy as one, three, two.

This is a very basic description. For more info see [Wikipedia](https://en.wikipedia.org/wiki/Proxy_server).

But it serves to illustrate what we want. We want a way to inspect the traffic as it is in transition from the application (client) to the server. To achieve this we will need to alter the flow of traffic by first sending the application traffic to an intercepting proxy, and that proxy will then forward it to the server.

### Proxy Tools ###

Lucky for us there are a number of proxy tools available. Some popular options are:
* [Fiddler](https://www.telerik.com/fiddler)
* [Charles](https://www.charlesproxy.com/)
* [BurpSuite](https://portswigger.net/burp)
* Literally 100s of others. Just do a Google search.

The tool is dependent on what you want to do. Depending on the tool you use it could allow you to monitor the traffic, or intercept and modify the traffic as it is being transmitted.  Most of these tools can also be used to perform various levels of security testing.

The companion video for this article demonstrates the basic usage of BurpSuite(https://portswigger.net/burp) as that is what the author is familiar with.

It doesn't really matter what tool you choose. What we are interested in is the Flutter side of things.

### How to Proxy HTTP Traffic ###

Standard HTTP traffic is easy to proxy. The data is sent unencrypted over the network, and all we need to do is configure our client to first send the data to the proxy, and the proxy will intercept and forward the HTTP traffic. Easy to do.

Please see the video for details how to configure the proxy for BurpSuite, as well as configure an Android Emulator to use a proxy. Whether you are proxying an Android or iOS device, an Emulator, or a browser, the basics are the same - you'll need to configure the proxy to point to the proxy server's ip address and listening port.

### Encrypted HTTP(S) ###

An easily digestable video regarding HTTPS - [How does HTTPS work](https://www.youtube.com/watch?v=T4Df5_cojAs&list=LLU8Mj6LLoNBXqqeoOD64tFg&index=2&t=0s)

This is where things get interesting. Aside from the fact that HTTPS encrypts our data over the wire, it also acts as a policewoman to ensure that we can trust the server that we are communicating with.

When our Flutter app sends an HTTPS request to our server an SSL Handshake is initiated.
> In a TLS/SSL handshake, clients and servers exchange SSL certificates, cipher suite requirements, and randomly generated data for creating session keys.

> An SSL certificate is a data file hosted on a website's origin server. SSL certificates make SSL/TLS encryption possible, and they contain the website's public key and the website's identity, along with related information. Devices attempting to communicate with the origin server will reference this file to obtain the public key and verify the server's identity. The private key is kept secret and secure.

The SSL certificate sent by the server is signed by a certificate authority (CA)
> In cryptography, a certificate authority or certification authority is an entity that issues digital certificates. A digital certificate certifies the ownership of a public key by the named subject of the certificate.

Our browsers and mobile devices have a list of known and trusted CAs. That means our browsers can verify that the server we are communicating with (who sent the SSL certificate) is who they say they are. If you want more information on Digital Certificates and Certificate Authorities I suggest the following videos:

* [Intro to Digital Certificates](https://www.youtube.com/watch?v=qXLD2UHq2vk&t=255s)
* [Digital Certificates: Chain of Trust](https://www.youtube.com/watch?v=heacxYUnFHA&t=25s)

Once we have established trust with the server we are communicating with, then we can initiate the encryption process. For that we will need an encryption key that is shared between the client and the server. This key will be used to encrypt and decrypt traffic between them. This is known as [Symmetric Key Encryption](https://www.cryptomathic.com/news-events/blog/symmetric-key-encryption-why-where-and-how-its-used-in-banking#:~:targetText=Symmetric%20encryption%20is%20a%20type,used%20in%20the%20decryption%20process).

First we need to generate a Symmetric Key, and here our client (Android/iOS device or browser) steps up and does just that. Now we need to send that key to our server. But hold on a minute. The point of this whole process is to be secure. That means we can't just send the Symmetric key over non-secure HTTP. We need a way to encrypt the encryption key, and a way for the server to decrypt that to, finally, get the key to do further encryption.

To do this we will use Asymmetric Encryption.
> Asymmetric cryptography, also known as public key cryptography, uses public and private keys to encrypt and decrypt data

That SSL Certificate we mentioned above, the one that the client receives from the server, contains a public key that we can use to encrypt the secret. To decrypt Asymmetric Encryption requires the matching private key of the public key used to encrypt the data. Under the correct circumstances, only our server will have access to that private key. And there we go, we have a secure way to send the Symmetric Key (or secret) to the server. And now all future traffic can be encrypted with that secret using Symmetric Cryptography.

You might ask, why do we need Symmetric Cryptography. Why not just use Asymmetric Encryption for everything. The short answer is because Symmetric Cryptography is faster, which lends itself well for internet traffic.

### Self-signed Certificates ###

> In cryptography and computer security, a self-signed certificate is a certificate that is not signed by a certificate authority (CA). These certificates are easy to make and do not cost money. However, they do not provide all of the security properties that certificates signed by a CA aim to provide.

Anyone can create their own Self-signed Certificate, meaning you do not have to make use of a Certificate Authority to sign your SSL certificate. But if anyone can create a self-signed certificate then why do we need a Certificate Authority? Well now it goes back to that chain of trust. Because a certificate is signed by a known, and trusted, Certificate Authority it means that our browsers and devices can trust those certificates. If a certificate is self-signed or signed by an unknown Certificate Authority you will see a screen similar to the one below:

![Proxy Error](/pictures/proxy_error.png)

If you are testing an application in a development environment the picture above might be very familiar. You will either tell your browser to ignore the warning and continue (bad idea if you do not trust the end point), or you will load your self-signed certificate into the browser's certificate manager as a trusted certificate. If you do the later your browser will no longer give a certificate warning and will treat the certificates signed with your self-signed certificate as trustworthy.

In the companion video I go into great detail explaining and demonstrating this.

On a mobile device this works in a similar fashion. Just like you can install a self-signed certificate in a browser, you can also install your own self-signed certificates on an Android and iOS device. Between Android and iOS the process is a little bit different and there are some limitations on Android (the reason I made this post). See the following links for more details:

* [iOS Certificate Profiles](https://support.apple.com/en-us/HT204477)
* [Android CA Certificate](https://stackoverflow.com/questions/4461360/how-to-install-trusted-ca-certificate-on-android-device)

The main thing that you need to know is that from Android N onwards it gets harder to intercept an application's SSL traffic. Android applications will not respect self-signed certificates added to the device's trust store. Certificates that you add on Android will be trusted by the Android browser, but not by other applications.

The reason that it won't work is for security purposes. A browser does not know which endpoint it will communicate with - you as the user might try to access https://google.com or you might go to https://flutter.dev, or any other possible endpoint. For a mobile application, or thick client, that is different. Under most circumstances your application should know the endpoint that it is communicating with. For example, if your company name is MathIsFun and you make an app that is called MathIsFun and your API server is hosted at https://api.mathisfun.com, then your app only needs to care about that particular API's SSL certificate.

If that SSL certificate is signed by a known and trusted Certificate Authority then great, all you need to do is make an HTTPS request to your endpoint and the rest will be handled for you. If it is not signed by a Certificate Authority then you will need to modify your source code to tell your application that it should trust your self-signed certificate.

This is quite easy to do, but should be done with caution. Self-signed certificates are perfect for development environments. However, under most circumstances you would prefer to go the Certificate Authority route. Most cloud services automatically configure an SSL certificate for you anyway, so there is no reason not to. Firebase hosting is an example. If you host an application on Firebase Hosting they will automatically provision SSL for you.

Note that a lot of these providers use shared SSL certificates - meaning other domains/sites might be using the same certificate as your site. This might not be an issue for you, but there are reasons that you might want your own dedicated SSL certificate issued to your domain/company. For more info see below:

[Firebase Shared SSL Certificates](https://stackoverflow.com/questions/53473695/google-firebase-ssl-certificate-my-certificate-has-a-large-number-of-other-web)

### How to proxy HTTPS ###

What we wanted from the start. I suggest you jump to the video as it will be easier to see exactly what you need to do, especially if this is the first time you are trying to intercept traffic.

The basics are as follows:
 - Start up the intercepting proxy tool (BurpSuite for example)
 - Configure the tool to listen on a dedicated port
 - Setup your device/browser to send traffic to the proxy tool
 - Get a certificate warning
 - Realise that your proxy tool's self-signed certificate is not recognised by your browser or mobile device
 - Add the proxy tool's self signed certificate to your device or browser's trust store
 - See that your proxy tool is intercepting the traffic (Unless you're using Android N or above)
 - Realise that your proxy tool is sending that data on to the server, and the server is responding to the proxy tool, and the proxy tool is forwarding the response on to you (win)

### How to proxy HTTPS traffic on Android N and above ###

As mentioned above, from Android N onwards you need to configure your application manually to trust self-signed certificates.

In Dart/Flutter this is quite easy to do.

Import some libraries:

{{< highlight dart >}}
import 'dart:io';
import 'package:flutter/services.dart';
{{< /highlight >}}

Add your proxy tool's certificate as an asset in your `pubsec.yaml` file. For example:

{{< highlight yaml >}}
assets:
   - assets/raw/certificate.pem
{{< /highlight >}}

Then add the following code somewhere in your application before making network requests. For example, in the `main` function.

{{< highlight dart >}}
void main() {
  ByteData data = await rootBundle.load('assets/raw/certificate.pem');
  SecurityContext context = SecurityContext.defaultContext;
  context.setTrustedCertificatesBytes(data.buffer.asUint8List());
  runApp(MyApp());
}
{{< /highlight >}}

What this does is it sets the set of trusted X509 certificates used by `SecureSocket` client connections, when connecting to a secure server.

And x509 is:
> a standard defining the format of public key certificates. X.509 certificates are used in many Internet protocols, including TLS/SSL, which is the basis for HTTPS, the secure protocol for browsing the web.

So basically you are adding your certificate to your application's list of trusted certificates. This is done at runtime and should only happen once. If you add it multiple times you will get an exception. You also need to add the certificate before attempting any SSL connections with your proxy, otherwise you will get an SSL exception.

Once you have done this you no longer need to add your self-signed certificate to your device/browser. As you are instructing your application to trust this certificate.

### SSL Encodings ###

The method `setTrustedCertificatesBytes` requires a PEM or PKCS12 file containing x509 certificates.

Depending on the Proxy Tool you use you might be required to convert your SSL certificates to a different encoding or file format. For example in the video I convert a DER file to a PEM file. For more information regarding this please see the following links:

* [Common OpenSSL commands](https://www.sslshopper.com/article-most-common-openssl-commands.html)
* [Difference between .pem, .cer and .der](https://stackoverflow.com/questions/22743415/what-are-the-differences-between-pem-cer-and-der)

### Considering Security ###

You will see a lot of questions on the internet asking how to get rid of the certificate errors in their application. A common solution is to disable all certificate verification and allow your app to accept any certificate.

As an example, see the Stack Overflow thread here:

[Solve Flutter Certificate Error](https://stackoverflow.com/questions/54285172/how-to-solve-flutter-certificate-verify-failed-error-while-performing-a-post-req)

This is obviously a terrible idea, and should never be considered - not even for a development environment. As you saw above it is quite easy to load a custom certificate into our application's list of trusted certificates, so don't be lazy when it comes to the security of your app.

You can limit the exposure more by only allowing these certificates to be loaded for debug builds of your application. Ensuring that the release build of your application will not trust any SSL certificates that were used for development and debugging purposes.

You can easily modify the example above into something similar to the following:

{{< highlight dart >}}
Future<bool> addSelfSignedCertificate() async {
  ByteData data = await rootBundle.load('assets/raw/certificate.pem');
  SecurityContext context = SecurityContext.defaultContext;
  context.setTrustedCertificatesBytes(data.buffer.asUint8List());
  return true;
}

void main() async {
  assert(await addSelfSignedCertificate());
  runApp(MyApp());
}
{{< /highlight >}}

Now our `main` function will only add our proxy certificate for debug builds of our application, as we wrap the function call in an `assert`. When performing a release build of our application, Dart strips all `assert` logic from our application.

## Conclusion ##

It is important to understand the basics of SSL Certificates and how HTTPS works before altering logic related to the security of your application's network traffic. Adding a self-signed certificate to your application's trust store is quite simple to do, but should still be done with caution. Do not take the easy route when it comes to the security of your application. 

Done correctly, you will have an easy way to intercept and inspect the underlying network traffic that your application generates.


