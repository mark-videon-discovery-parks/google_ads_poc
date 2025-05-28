# Google Mobile Ads

## Flutter Doctor 

```
DP077873:~ mark.videon$ flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.29.2, on macOS 15.5 24F74 darwin-arm64, locale en-AU)
[✓] Android toolchain - develop for Android devices (Android SDK version 35.0.0)
[✓] Xcode - develop for iOS and macOS (Xcode 16.0)
[✓] Chrome - develop for the web
[✓] Android Studio (version 2024.1)
[✓] VS Code (version 1.100.0)
[✓] Connected device (4 available)
    ! Error: Browsing on the local area network for iPad-RW64MQQ0LC. Ensure the device is unlocked and attached with a cable or associated with the same local area network as this Mac.
      The device must be opted into Developer Mode to connect wirelessly. (code -27)
    ! Error: Browsing on the local area network for Mark’s iPhone. Ensure the device is unlocked and attached with a cable or associated with the same local area network as this Mac.
      The device must be opted into Developer Mode to connect wirelessly. (code -27)
    ! Error: Browsing on the local area network for iPad-WXLNFPL7QR. Ensure the device is unlocked and attached with a cable or associated with the same local area network as this Mac.
      The device must be opted into Developer Mode to connect wirelessly. (code -27)
    ! Error: Browsing on the local area network for MV Pro Max. Ensure the device is unlocked and attached with a cable or associated with the same local area network as this Mac.
      The device must be opted into Developer Mode to connect wirelessly. (code -27)
[✓] Network resources

• No issues found!
```

## Creating the reproducible example

Steps:

1. Created clean Flutter project with android as the only platform
2. Added google_mobile_ads as project dependency
3. Updated ndkVersion in android/app/build.gradle.kts to "27.0.12077973" as suggested by compile-time error
4. Updated minSdk in android/app/build.gradle.kts to 23 as suggested by compile-time error
5. Updated version of "org.jetbrains.kotlin.android" in android/settings.gradle.kts to 2.1.0 as suggested by compile-time error
6. Complete Android Manifest integration step as suggested by compile-time error
7. Replace main.dart with the code below:

```
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await MobileAds.instance.initialize();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Hello World Ad Demo',
      home: const HelloWorldAdPage(),
      debugShowCheckedModeBanner: false,
    );
  }
}

class HelloWorldAdPage extends StatefulWidget {
  const HelloWorldAdPage({super.key});

  @override
  State<HelloWorldAdPage> createState() => _HelloWorldAdPageState();
}

class _HelloWorldAdPageState extends State<HelloWorldAdPage> {
  BannerAd? _bannerAd;

  @override
  void initState() {
    super.initState();
    _bannerAd = BannerAd(
      adUnitId:
          "ca-app-pub-3940256099942544/6300978111", // Official Google test ad unit ID for Android
      size: AdSize.banner,
      request: const AdRequest(),
      listener: BannerAdListener(
        onAdLoaded: (_) => setState(() {}),
        onAdFailedToLoad: (ad, error) {
          ad.dispose();
        },
      ),
    )..load();
  }

  @override
  void dispose() {
    _bannerAd?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          const Center(
            child: Text(
              'Hello World',
              style: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
            ),
          ),
          if (_bannerAd != null)
            Align(
              alignment: Alignment.bottomCenter,
              child: SafeArea(
                child: SizedBox(
                  width: _bannerAd!.size.width.toDouble(),
                  height: _bannerAd!.size.height.toDouble(),
                  child: AdWidget(ad: _bannerAd!),
                ),
              ),
            ),
        ],
      ),
    );
  }
}
```
