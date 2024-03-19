# Maintenance Discontinued

We regret to inform you that maintenance for this project has been discontinued. Due to the constant changes in the 'protocol' of the official app, coupled with the lack of personal usage requirements for this SDK and recent busy schedules, I have decided not to continue maintaining it.

However, if you still wish to explore the SDK, you can refer to my note and a decompiled project to study the protocol of the latest app.

[Protocol Study for Xiaomi Band](http://www.zhaoxiaodan.com/android/%E5%B0%8F%E7%B1%B3%E6%89%8B%E7%8E%AF%E8%93%9D%E7%89%99%E5%8D%8F%E8%AE%AE%E7%A0%94%E7%A9%B6.html)

[Xiaomi App Decompiled Project](https://github.com/pangliang/xiaomi-health-app-decompile)

## Currently Tested Firmware Versions

Xiaomi Sports App:

- iOS version: 1.3.57
- Android version: 1.8.711

Standard Version (MI):

- Firmware version: 4.16.11.7

Heart Rate Version (MI1S):

- Firmware version: 4.15.12.10
- Heart rate version: 1.3.74.64

## Usage

Add the following to the dependencies section of your project's `build.gradle` file:
`compile 'com.zhaoxiaodan.miband:miband-sdk:1.1.2'`



## Release Notes

### 1.1.2 - 2016-02-22

- Fixed Bluetooth disconnection issue caused by `setUserInfo`. When the set `userid` is different from the previous one, the bracelet will flash and vibrate. At this time, you need to tap the bracelet to confirm pairing, just like with the official app. When the userid is the same as before, the bracelet does not respond.
- UserInfo must be set before obtaining heart rate scan data.

### 1.1.1 - 2016-02-03

- Added support for obtaining heart rate scan data.

### 1.0.1 - 2015-11-20

- Scan nearby Le devices, and choose to connect when multiple bracelets are nearby.
- Added device disconnection listener.
- After connecting, UserInfo is no longer needed and can be used. Setting UserInfo will cause the heart rate version bracelet to disconnect. To be fixed.
- Fixed vibration issue. Now there are three vibration modes: three lights flash and vibrate twice, no lights vibrate twice, middle light vibrates ten times. Vibration can be stopped at any time using `stop`.
- Gravity sensor data is not available.
- Heart rate version seems to have a single-color LED light, cannot set LED color; the original version can.

### 1.0.0 - 2015-08-17

- Obtained motion sensor data.
- Set user information.
- Receive real-time step count notifications.
- Vibrate bracelet.
- Set LED color.
- Obtain battery information.
- Obtain signal strength RSSI value information.

## API

```java
// Instantiation
MiBand miband = new MiBand(context);

// Scan nearby devices
final ScanCallback scanCallback = new ScanCallback() {
    @Override
    public void onScanResult(int callbackType, ScanResult result) {
        BluetoothDevice device = result.getDevice();
        Log.d(TAG,
            "Found nearby Bluetooth device: name:" + device.getName() + ", uuid:"
                    + device.getUuids() + ", address:"
                    + device.getAddress() + ", type:"
                    + device.getType() + ", bondState:"
                    + device.getBondState() + ", rssi:" + result.getRssi());
        // Display according to the situation
    }
};
MiBand.startScan(scanCallback);

// Stop scanning
MiBand.stopScan(scanCallback);

// Connect to one of the devices scanned earlier
miband.connect(device, new ActionCallback() {
    @Override
    public void onSuccess(Object data) {
        Log.d(TAG,"connect success");
    }
    
    @Override
    public void onFail(int errorCode, String msg) {
        Log.d(TAG,"connect fail, code:"+errorCode+",msg:"+msg);
    }
});

// Set disconnected listener to facilitate reconnection or other processing when the device is disconnected
miband.setDisconnectedListener(new NotifyListener() {
    @Override
    public void onNotify(byte[] data) {
        Log.d(TAG,
            "Connection disconnected!!!");
    }
});

// Set UserInfo, must be set before heart rate detection
// When the last parameter Type is 1, the bracelet will flash and vibrate every time, at this time, you need to tap the bracelet to confirm pairing; just like pairing with the official app.
// When Type is 0, pairing confirmation is only required when the set userid is different from the previous one;
// When Type is 0, and the set userid is the same as before, the bracelet does not respond; a notification with a value of 3 will be received in the normal notify
UserInfo userInfo = new UserInfo(20111111, 1, 32, 180, 55, "胖梁", 0);
miband.setUserInfo(userInfo);

// Set heart rate scan result notification
miband.setHeartRateScanListener(new HeartRateNotifyListener() {
    @Override
    public void onNotify(int heartRate) {
        Log.d(TAG, "heart rate: "+ heartRate);
    }
});

// Start heart rate scan
miband.startHeartRateScan();

// Read and connect to the device's signal strength RSSI value
miband.readRssi(new ActionCallback() {
    @Override
    public void onSuccess(Object data) {
        Log.d(TAG, "rssi:"+(int)data);
    }
    
    @Override
    public void onFail(int errorCode, String msg) {
        Log.d(TAG, "readRssi fail");
    }
});

// Read bracelet battery information
miband.getBatteryInfo(new ActionCallback() {
    @Override
    public void onSuccess(Object data) {
        BatteryInfo info = (BatteryInfo)data;
        Log.d(TAG, info.toString());
        //cycles:4,level:44,status:unknown,last:2015-04-15 03:37:55
    }
    
    @Override
    public void onFail(int errorCode, String msg) {
        Log.d(TAG, "readRssi fail");
    }
});

// Vibrate 2 times, with all three LEDs on
miband.startVibration(VibrationMode.VIBRATION_WITH_LED);

// Vibrate 2 times, without LEDs on
miband.startVibration(VibrationMode.VIBRATION_WITHOUT_LED);

// Vibrate 10 times, with the middle LED in blue
miband.startVibration(VibrationMode.VIBRATION_10_TIMES_WITH_LED);

// Stop vibration, can be called anytime during vibration to stop
miband.stopVibration();

// Get normal notification, data len=1 usually, value is notification type, types are not yet collected
miband.setNormalNotifyListener(new NotifyListener() {
    @Override
    public void onNotify(byte[] data) {
        Log.d(TAG, "NormalNotifyListener:" + Arrays.toString(data));
    }
});

// Get real-time step count notification, after setting, shake the bracelet (need to shake continuously 10-20 times to trigger), will receive real-time total step count notification for the day
// Use in two steps:
// 1. Set the listener

miband.setRealtimeStepsNotifyListener(new RealtimeStepsNotifyListener() {
    @Override
    public void onNotify(int steps) {
        Log.d(TAG, "RealtimeStepsNotifyListener:" + steps);
    }
});

// 2. Enable notification
miband.enableRealtimeStepsNotify();

// Disable (pause) real-time step count notification, to enable again just call miband.enableRealtimeStepsNotify() again
miband.disableRealtimeStepsNotify();

// Set LED color, orange, blue, red, green
miband.setLedColor(LedColor.ORANGE);
miband.setLedColor(LedColor.BLUE);
miband.setLedColor(LedColor.RED);
miband.setLedColor(LedColor.GREEN);

// Get gravity sensor raw data, requires two steps
// 1. Set the listener
miband.setSensorDataNotifyListener(new NotifyListener() {
    @Override
    public void onNotify(byte[] data) {
        int i = 0;

        int index = (data[i++] & 0xFF) | (data[i++] & 0xFF) << 8;  // Serial number
        int d1 = (data[i++] & 0xFF) | (data[i++] & 0xFF) << 8;    
        int d2 = (data[i++] & 0xFF) | (data[i++] & 0xFF) << 8;
        int d3 = (data[i++] & 0xFF) | (data[i++] & 0xFF) << 8;

    }
});

// 2. Enable
miband.enableSensorDataNotify();

// Pair, seems useless, other operations can be done without pairing
miband.pair(new ActionCallback() {
    @Override
    public void onSuccess(Object data) {
        changeStatus("pair succ");
    }
    
    @Override
    public void onFail(int errorCode, String msg) {
        changeStatus("pair fail");
    }
});

```
