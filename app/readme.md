amv-access-sdk-sample-app-android
=================================

The amv Access Demo App is an example application using the amv Access SDK for Android.
The projects purpose is to provide a basic overview and demonstrate fundamental concepts of
the Access SDK. It contains examples for downloading access certificates from the backend server
and starting bluetooth broadcasting with a chosen certificate.

### api credentials
Copy the `application.properties.template` file to `application.properties` and provide your own
credentials. Please keep in mind that storing credentials in plaintext is a security risk and
should not be done in an application used in production. This application uses this approach for
demonstration purposes and for the sake of simplicity. You responsible for securely storing api
credentials.

### demo application
If your backend server runs in demonstration mode, it will create an "application entity" for
demonstration purposes at startup. When you use the credentials of this demo application, the
server will create at least one access certificate associated to a demo vehicle for every newly
registered device.

### integration tests
The tests are in InstrumentedTests file. The app should be deleted before running tests to assure
a new keypair is generated and a device certificate for that keypair is downloaded from the server.

Integration tests currently cover following scenarios:
* creating a keypair
* downloading a device certificate for that keypair
* downloading access certificates for that device certificate
* visibility of the correct serials/count for the certificates
* deleting an access certificate

## how to use the amv Access SDK in your own app
See [jitpack.io/#amvnetworks/amv-access-sdk-android](https://jitpack.io/#amvnetworks/amv-access-sdk-android)
for all available versions.

Add the following in your root `build.gradle` at the end of repositories:
```groovy
dependencies {
  	allprojects {
  		repositories {
  			...
  			maven { url 'https://jitpack.io' }
  		}
  	}
}
```

Include an implementation of the `amv-access-sdk-spi` in your project e.g. `amv-hm-access-sdk`.
Include it in the `dependencies` block of your `build.gradle`:
```groovy
dependencies {
    // ...
    implementation "com.github.amvnetworks.amv-access-sdk-android:amv-hm-access-sdk:${accessSdkImplVersion}"
}
```

The Access SDK handles most of the networking/bluetooth + certificate management.
You can use this example repository as template for your own application.
Most important concepts to keep in mind:
- The SDK must be initialized before usage. You can use its functionality only after
the initializing phase has ended successfully.
- On first startup a key pair is generated and registered with your backend server.
- If your backend runs in demonstration mode, a newly registered device will immediately get
an example access certificate for a demo vehicle.
- All information is lost if your application is uninstalled. Reinstalling will start from scratch.

Depending on the implementation you may need additional libraries in your classpath to
successfully compile the source or use all features. You can always contact devs if you need
further information.

## sample app usage
### initialization
`AmvSdkInitializer.create(Context context)` will create and initialize an `AccessSdk` instance.
It does so by reading api credentials from a the `application.properties` file (as mentioned above
this is done for demonstration purposes only and imposes a security risk).

### certificate management
To handle certificates your Activity should implement `ICertificatesView` interface for callbacks.
Main access point for is in `CertificateController.java`. You can initialize it in your
Activity with:
```java
this.controller = new CertificateController();
this.controller.initialize(this, this);
```

If initialize succeeds then ICertificatesView's `onInitializeFinished()` is called.
After this you can call the CertificateController's
```java
void downloadCertificates();
```
to download/refresh the access certificates. When finished, ICertificatesView's
`onCertificatesDownloaded()` will be called.

Also,
```java
Observable<AccessCertificatePair> getCertificates();
```
is available to get the already downloaded access certificates. These certificates will be used
to start bluetooth broadcasting and you can use the gaining serial in them to get the vehicle serial.

To revoke a certificate from the server
```java
void revokeCertificate(AccessCertificate certificate);
```
can be called. If the backend server supports revoking certificate and deletes it successfully the
locally stored copy will also be removed. Subsequently `onCertificateRevoked();` is called.

Once access certificates are present, they can be displayed to the user. When the user selects a
certificate, bluetooth broadcasting will be started and the app tries to connect to the vehicle.