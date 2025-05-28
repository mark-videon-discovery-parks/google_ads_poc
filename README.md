# Google Mobile Ads

## Summary

When integrating Google Mobile Ads on Android, a runtime error has been observed in debug and release builds. After tapping on an ad while using the app, the device will open a link in a web browser on device. After doing this, the app session will be in an unrecoverable state. The OS does not immediately terminate the application, it is still observable in the app switcher, but logs suggest that the Flutter engine terminates.

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

## Logs

Logs after tapping an ad:

```
D/CompatibilityChangeReporter(28425): Compat change id reported: 289878283; UID 10300; state: ENABLED
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
W/Ads     (28425): Unable to append parameter to URL: https://www.googleadservices.com/pagead/aclk?fbs_aeid=-4234101963759360000&sa=L&ai=CNf8kWZ02aLG6LobE4t4PrdKPkAIA04b_or8JwI23ARABIMLs3CBgpYCAgJwBiAEBoAH71NLXA6kCuJZ_QWSjpT6oAwGqBM8CT9Bfuh_FZj35OcPkT05Tneax8eQNv-NMZLierLqKgow2-FQbh2z33Ao5lxyfgzXMrnbTbSXDTltjqY9qcbnus8p3oe3FlvGyP2-o3VSnBDnpEdD8ErJGrZRb9YEI2LGgaGtgYwpAvrlPNPZzxC6C3iLyS6M6C9vIwre-C_jEie9jBT0bZyufjKHedC4Qw_t4_VyQzMKIw9wNUJqeC-WdCizuXrZTSELRMPUiqMHJTDXUDQ9Yly7N2YbryLRiemHK_Zvgskk0Cu8ieWA4qG0VE1VvGXuX6lShLKFRXDkukqOuWZtKjL834jmxhsK-Wq7gI6v_1uTSVR_4NmjFXMw3HpGZ_Clyznq01_qxYBR2vNWmA4w_Ou_x2d5ICiX1gQAcbxH16amqFvMtI3wqyKtqBU4DD8VIjdHXNYwmLzQNGEajgyg512B7ToPwZmBzsU_ABKfx3_XmAYgF0Y3N1wWQBgGgBjnABguAB-2qrSiIBwGQBwKYBwGoB_-esQKoB6a-G6gH1ckbqAehAagH_p6xAqgHtq2xAqgHoaqxAqgHmbWxAqgH89EbqAeW2BuoB6qbsQKoB9-fsQKoB47cG6gH9rixAqgH4L2xAqgHw9KxAtgHAaAIl9KpBLAIAtIILAgAEAIYygEyAQA6Ep_QgICAgASQwICAgKCogAKoAUi9_cE6WPGa69O1xY0DgAoCogwIKgYKBNrPsQLaDCEKCxDA5ojAufvWm9UBEgEFMgQKAjE3QAFSAz0-P1oCCAyqDQJBVeoNEwjYjpHUtcWNAxUGotgFHS3pAyLwDQGCFCkaJ21vYmlsZWFwcDo6Mi1jb20uZXhhbXBsZS5nb29nbGVfYWRzX3BvY7AVAdAVAZgWAfgWAYAXAbIXAhgBshgDIgEAmhkHCAEaA1VTRA&ae=1&ase=3&num=1&cid=CAQSjQEA2abss7E3_RluiEMPDjKgueCvTEKEWhXN8iz63SmAaheMZU0y6HoF5kFr0_IU7s0vSnPOMVpFMbuEMdDxX85vpT5BZ1J6V6ndNgchNoJS89mrWP4gdQh4byncyrF1fIteRLq2nz0QKqWaY651rZS95-q0tM_CaHZCaiAPprqEmBGd47Prqtd0htlq-DIYAQ&sig=AOD64_1TQ8jnRU85JSL6TNxzpcVppsFwVQ&nb=2&nx=228&ny=29&mb=1&nb=2&uach=WyJBbmRyb2lkIiwiMTQuMC4wIiwiIiwiQTE0MiIsIjEzNi4wLjcxMDMuMTI1IixudWxsLDEsbnVsbCwiIixbWyJDaHJvbWl1bSIsIjEzNi4wLjcxMDMuMTI1Il0sWyJBbmRyb2lkIFdlYlZpZXciLCIxMzYuMC43MTAzLjEyNSJdLFsiTm90LkEvQnJhbmQiLCI5OS4wLjAuMCJdXSwwXQ..&ms=CrAFmsA_ATEaPwHInBRcIlXhh-ISGwCqaDQel5Mu4gdNQCTYzs2MwVXTC3skGLN9klSi2btmlBKwn6MA7msjCeOyler-XCsLFhLnBF0xmJrRszawpeVG-Opkn8Qnk9-fFStVs3WTlwPnyY1Yn6qnhBbbiY7KFiYiMe5_o9VIlQdXQLGvpAxxUR9gaqsHE4VCCeKY67WsDwOCwDnFm80vHbi5qunqZEGOXFH4jaN3Otbl_7xnqzqKm30MGWsmF0m413uc6UZY5El3ZgDdJOCltgRoY8FtINTTD6wWDm3n0rimp-wNDSITK_zaUGZxecmaGwMmqVBFKG9jKXR6vz237m-Me2Q0O9PaWWgg0G7z1SYuSI2DiMUFUmi4RA2HdFKN9H9gRXbklcNDHFe9o3ET23CwaQUwqmAq3BicOvEF7uJ_yK2ap_6kFVC7dBi8qVGxTJc0QbX0oy4Cf04kBJy7P8zk3y01eJlStHFC_iXYb0y-tEb9gy6UM-kloRrkQqmt6VToSKIzxYOcUMKT-Hl2jGAkWRSijw9nm9r-65G3fQD8X2A7nFFfnot0bhldDPKhI2qA6NaFpXF2iOhd2t1Gooo0NFdJfLYDCCv8yQUbLi4F5DU2QJXLAIdOV9euKyeyT8Z5NDG4d0J3eM0n6WZV0uEBLewoOALjpYN9z0-3ZO3nrenWgAWeEE92vIUVZewkwCM0WvNHSbeF3SO_rQ5_wdty8_rukZZNtqIf4JA7Vrr41Md8DDnSvdV9nE0Jx4ElsVfskLlJOxSJUelXcIYSBlJq5QRTb5m09fpgoWM0RMRgWID6FAVCyc1UkcBdzmclEFxutyBwLyRV9ANJq1BZOJpZhWorhrE6lvpUStTbpwV_vhxnD7ntU5kDDRDQqpY_h3mwUgdPF51lNUpH8C358tClXSAE&act=1&ri=1&adurl=https://developers.google.com/admob/%3Fgclid%3DCj0KCQjwxdXBBhDEARIsAAUkP6hKD5nC3NL392cEz5NNKmaOy7bWCYyTe7PtTCcILJeKEkatXBbc-94aAhUhEALw_wcB
D/MeasurementManager(28425): AdServicesInfo.version=17
D/Surface (28425): lockHardwareCanvas
E/GPUAUX  (28425): [AUX]GuiExtAuxCheckAuxPath:682: Null anb
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/libMEOW (28425): meow new tls: 0xb400007c68d7c0a0
D/libMEOW (28425): applied 1 plugins for [com.example.google_ads_poc]:
D/libMEOW (28425):   plugin 1: [libMEOW_gift.so]:
D/libMEOW (28425): meow delete tls: 0xb400007c68d7c0a0
D/VRI[AdActivity](28425): hardware acceleration = true, forceHwAccelerated = false
D/InputTransport(28425): Create ARC handle: 0xb400007d28dbc8f0
D/InputEventReceiver(28425): Input log is disabled in InputEventReceiver.
D/InputTransport(28425): Input log is disabled in InputChannel.
D/BufferQueueConsumer(28425): [](id:6f0900000004,api:0,p:-1,c:28425) connect: controlledByApp=false
D/NtViewRootImpl(28425): mPopUpViewOffsets: offset=(0.0, 0.0), scale=(1.0, 1.0)
E/OpenGLRenderer(28425): Unable to match the desired swap behavior.
D/BLASTBufferQueue(28425): [VRI[AdActivity]#2](f:0,a:1) acquireNextBufferLocked size=1084x2412 mFrameNumber=1 applyTransaction=true mTimestamp=15064952120897(auto) mPendingTransactions.size=0 graphicBufferId=122084445388820 transform=0
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/Surface (28425): lockHardwareCanvas
E/FrameEvents(28425): updateAcquireFence: Did not find frame.
D/BLASTBufferQueue(28425): [SurfaceView[com.example.google_ads_poc/com.example.google_ads_poc.MainActivity]#1](f:0,a:5) destructor()
D/BufferQueueConsumer(28425): [SurfaceView[com.example.google_ads_poc/com.example.google_ads_poc.MainActivity]#1(BLAST Consumer)1](id:6f0900000001,api:0,p:-1,c:28425) disconnect
D/BLASTBufferQueue(28425): [VRI[MainActivity]#0](f:0,a:5) destructor()
D/BufferQueueConsumer(28425): [VRI[MainActivity]#0(BLAST Consumer)0](id:6f0900000000,api:0,p:-1,c:28425) disconnect
D/BufferQueueConsumer(28425): [ImageReader-840x131f22m5-28425-1](id:6f0900000003,api:0,p:-1,c:28425) disconnect
D/BLASTBufferQueue(28425): [VRI[AdActivity]#2](f:0,a:1) destructor()
D/BufferQueueConsumer(28425): [VRI[AdActivity]#2(BLAST Consumer)2](id:6f0900000004,api:0,p:-1,c:28425) disconnect
Application finished.

Exited.
```