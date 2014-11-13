# Introduction

This page details our obligations as a result of the software and services we use.

# Client

## Adobe RMSDK 11

_NOTE: This is taken from the RMSDK 11 User manual._

**7 Device Certification Process**

RMSDK 11 adds a new step to the certification process. There are two kinds of certificates, Development certificate and Distribution certificate.

New process is as follows:

1. Adobe provides a default development certificate as part of RMSDK 11 for the sample ‘book2png’, available in the SDK.
Note: ReaderClientCert.sig (available at the path: RMSDK/resources/) is the default development certificate provided which allows you to build and run sample book2png.
2. A development certificate is valid for 30 days and post that most of the RMSDK functionalities will stop working by throwing an error ‘E_ADEPT_CLIENT_CERT_FAILED’.
3. A developer can renew his development certificate for additional 30 days by procuring a new certificate from Adobe’s website or through a licensing partner.
4. Once the development period is completed and before submitting the app to Adobe or your licensing partner/reseller for ‘device certification’, the user has to get this distribution certificate by visiting the website again. User needs to repackage the reader with the distribution certificate and submit to Adobe or reseller. In case user wants to do some code change and rebuild his application, he has to go through certification process again by resubmitting his final app to get the final distribution certificate.

Ensure you use the distribution certificate in your final app before submitting to app market place otherwise the users will get the client_cert_failed message.

Note: Refer the sample book2png to see how the certificate is bundled and packaged with readers.
If there is any change in source code for the reader application, customer needs to re-apply for distribution
certification to Adobe.

For any certification related queries, customers can send mail to rmsuppor@adobe.com

**7.1 Difference between development certificate and distribution certificate**

**7.1.1 Development certificate**

Development certificate allows you to use RMSDK functionalities for 30 days and it will expire post that. RMSDK will throw ‘E_ADEPT_CLIENT_CERT_FAILED’ error message after it expires. Refer code level documentation in ‘RMSDK /drm/adept/public/adept_public.h’ for details on the error codes.

**7.1.2 Distribution certificate**

Distribution certificate must be used when your reader app is ready to ship and go to market place. In case the developer wants to do some code change and rebuild his application, he has to go through certification process again by resubmitting his final app to get the final distribution certificate. You should fill correct details of your app to get a distribution certificate.

* For an iOS/Mac app, make sure the AppID matches the CFBundleIdentifier value of your .app package while requesting for distribution certificate in Adobe’s website.
* For an Android app, make sure the AppID matches the packageName specified in your Android app manifest file.
* For remaining platforms please make sure you enter a proper AppID which can uniquely identify your reader instance.

The distribution certificate helps Adobe/reseller to uniquely identify a specific reader application built using RMSDK.