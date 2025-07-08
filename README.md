flutter_benlink
===============

A Dart port of the excellent Python library [benlink](https://github.com/khusmann/benlink) by Kyle Husmann, designed for controlling Benshi-protocol radios (like the BTech GMRS-PRO) from a Flutter application.

This library provides the low-level protocol implementation and a high-level `RadioController` to make it easy to interact with your radio over Bluetooth. The primary goal is to empower developers to create custom, open-source mobile applications for their hardware.

This project is a work in progress, with the ultimate goal of supporting the full feature set of the original `benlink` library.

* * * * *

### ⚠️ Disclaimer

This project is an independent effort and is **not** affiliated with or endorsed by Benshi, BTech, Vero, or any other radio manufacturer. The protocol has been reverse-engineered. Use this library at your own risk. The authors are **not** responsible for any damage caused to your radio or other equipment.

* * * * *

🎯 Current Focus: Android + Bluetooth Classic
---------------------------------------------

The initial version of this library is focused on providing core functionality for **Android apps** using **Bluetooth Classic (RFCOMM)**.

* * * * *

✅ Supported Features
--------------------

This library already supports the core, non-audio features of the radio protocol:

-   **Connection Management**: Connects to the radio via `flutter_bluetooth_serial`.

-   **High-Level Controller**: An easy-to-use `RadioController` that manages state using `ChangeNotifier`, making it simple to integrate with your Flutter UI.

-   **Device Info**: Read the radio's model, vendor, version, and supported features.

-   **Radio Settings**: Read and write the main settings block of the radio.

-   **Channel Management**:

    -   Read and write individual channel configurations (frequency, name, tones, etc.).

    -   Fetch a list of all channels from the radio.

-   **Real-time Status**:

    -   Get live status updates (Power, TX/RX state, Scan state, GPS lock, etc.).

    -   Subscribe to events for changes in status, settings, and the current channel.

-   **Power & GPS**:

    -   Read the current battery voltage and percentage.

    -   Fetch the last known GPS position from the device.

-   **VFO Control**: Read the VFO channel and set its frequency.

* * * * *

🗺️ Roadmap to Full `benlink` Compatibility
-------------------------------------------

The goal is to implement the entire feature set of the original Python library. Pull requests are welcome!

-   [ ] **Audio Streaming**: Implement real-time audio capture (TX) and playback (RX) over the audio RFCOMM channel. This is the highest priority feature.

-   [ ] **TNC / Packet Support**: Add support for sending and receiving APRS/BSS data packets and managing `BeaconSettings`.

-   [ ] **BLE Support**: Add a `BleCommandLink` to allow connections over Bluetooth Low Energy, in addition to the current RFCOMM link.

-   [ ] **Expanded Command Set**: Implement the remaining commands from the protocol (e.g., programmable function buttons, advanced settings, etc.).

-   [ ] **Firmware Flashing**: As a long-term goal, investigate and implement the firmware update process.

* * * * *

📦 Installation
---------------

This package is not yet on `pub.dev`. To use it in your project, add it to your `pubspec.yaml` as a git dependency:

    dependencies:
      flutter_benlink:
        git:
          url: https://github.com/SarahRoseLives/flutter_benlink.git
          ref: main 

Then, run flutter pub get in your terminal.

# 🚀 Quick Start

Ensure you have flutter_bluetooth_serial set up in your Android project.

Use ChangeNotifierProvider (from the provider package) to make the RadioController available to your widget tree.


    import 'package:flutter/material.dart';
    import 'package:flutter_benlink/flutter_benlink.dart';
    import 'package:flutter_bluetooth_serial/flutter_bluetooth_serial.dart';
    import 'package:provider/provider.dart';
    
    class RadioInfoScreen extends StatefulWidget {
      final BluetoothDevice server; // The paired radio device
    
      const RadioInfoScreen({Key? key, required this.server}) : super(key: key);
    
      @override
      _RadioInfoScreenState createState() => _RadioInfoScreenState();
    }
    
    class _RadioInfoScreenState extends State<RadioInfoScreen> {
      RadioController? _radioController;
    
      @override
      void initState() {
        super.initState();
        _connectToDevice();
      }
    
      void _connectToDevice() async {
        try {
          final connection = await BluetoothConnection.toAddress(widget.server.address);
          setState(() {
            // Initialize the controller, which will start fetching radio state
            _radioController = RadioController(connection: connection);
          });
        } catch (e) {
          print('Cannot connect, exception occurred: $e');
        }
      }
    
      @override
      void dispose() {
        _radioController?.dispose();
        super.dispose();
      }
    
      @override
      Widget build(BuildContext context) {
        // If the controller isn't initialized, show a loading indicator
        if (_radioController == null) {
          return Scaffold(
            appBar: AppBar(title: const Text('Connecting...')),
            body: const Center(child: CircularProgressIndicator()),
          );
        }
    
        // Use a ChangeNotifierProvider to provide the controller to the widget tree
        return ChangeNotifierProvider.value(
          value: _radioController,
          child: Scaffold(
            appBar: AppBar(
              title: const Text('Radio Info'),
            ),
            // Use a Consumer to listen for changes and rebuild the UI
            body: Consumer<RadioController>(
              builder: (context, radio, child) {
                // Wait until the radio is ready before showing data
                if (!radio.isReady) {
                  return const Center(child: CircularProgressIndicator());
                }
    
                return ListView(
                  padding: const EdgeInsets.all(16.0),
                  children: [
                    Text('Product: ${radio.deviceInfo?.productName ?? "N/A"}'),
                    Text('Firmware Version: ${radio.deviceInfo?.firmwareVersion ?? "N/A"}'),
                    Text('Current Channel: ${radio.currentChannelName}'),
                    Text('Battery: ${radio.batteryLevelAsPercentage}% (${radio.batteryVoltage?.toStringAsFixed(2)}V)'),
                    Text('GPS Locked: ${radio.isGpsLocked ? "Yes" : "No"}'),
                  ],
                );
              },
            ),
          ),
        );
      }
    }

# 🙏 Acknowledgements

Kyle Husmann (khusmann) for creating and open-sourcing the original benlink library and for doing the heavy lifting of reverse-engineering the protocol.

@spohtl for help with the audio protocol in the original project.

@na7q for early testing and feedback on the original project.