# Cordova and Capacitor plugin for Brother Label Printers

## Supports Android and iOS

Uses the Brother Print SDK for Android and iOS ...

More info can be found here, including a list of compatible printers: 
https://support.brother.com/g/s/es/dev/en/mobilesdk/download/index.html?c=us_ot&lang=en&comple=on&redirect=on

Already bundled is the following version:
 - Brother Print SDK for Android™ v4.13.0 (2025-08-09) which is in the `src/android/libs` dir.
 - Brother Print SDK for iPhone/iPad v4.0.2 (2020-09-08)

## Installing

In your Cordova project, run the following command to install the plugin.

```
cordova plugin add git+https://github.com/AbobosSoftware/cordova-plugin-brother-label-printer
```
or

```
cordova plugin add cordova-plugin-brother-label-printer
```

And then read [usage](#usage) below.

## Help and Support

Please do not email me for support or help with this plugin, use the issue tracker link above, so everyone will benefit from community questions and involvement, and I don't have to answer the same questions over and over for many individuals.

This is a Cordova plugin, firstly. You should be familiar with the Cordova plugin system before you try to use this plugin. Fortunately, it's pretty straight forward and easy to understand.

You can [read more about Android plugin development for Cordova here](https://cordova.apache.org/docs/en/latest/guide/platforms/android/plugin.html).  Knowledge of all of these internals is not necessary, but it doesn't hurt to be familiar either. 

Read here to [learn how to use Cordova Plugins](https://cordova.apache.org/docs/en/latest/guide/cli/index.html#add-plugins).


## Target mobile printers:
```
PocketJet PJ-722, PJ-723, PJ-762, PJ-763, PJ-763MFi, PJ-773
PocketJet PJ-622, PJ-623, PJ-662, PJ-663
PocketJet PJ-520, PJ-522, PJ-523, PJ-560, PJ-562, PJ-563
MPrint MW-145MFi, MW-260MFi
MPrint MW-140BT, MW-145BT, MW-260
RJ-4030Ai, RJ-4030, RJ-4040
TD-2020, TD-2120N, TD-2130N, TD-4000, TD-4100N
QL-710W, QL-720NW, QL-810W, QL-820NWB, QL-1110NWB, QL-1115NWB,
PT-E550W, PT-P750W
RJ-3050, RJ-3150
PT-E800W, PT-D800W, PT-E850TKW
PT-P900W, PT-P950NW
```

__Tested models:__ `QL-720NW`, `QL-810W`, `QL-820NWB`, `QL-1110NWB`, `TD-2120N`

(if you have tried this with other models, please update this list and send a pull request)


## Supported interfaces (by this plugin):

* Wi-Fi (Infrastructure mode)
* Bluetooth (Android and iOS)
* USB

### BlueTooth notes:
If your app is supporting BlueTooth, you will need the following changes in you rproject:
#### Android permissions
```
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
```
#### iOS

* For iOS, you need to apply for an MFi PPID.
Please follow the steps at 
https://secure6.brother.co.jp/mfi/Top.aspx

* Code change in your cordova config

```
    <config-file parent="NSBluetoothPeripheralUsageDescription" platform="ios" target="*-Info.plist">
        <string>Bluetooth access for Brother label printer</string>
    </config-file>
    <config-file parent="NSBluetoothAlwaysUsageDescription" platform="ios" target="*-Info.plist">
        <string>Bluetooth access for Brother label printer</string>
    </config-file>
    <config-file parent="UISupportedExternalAccessoryProtocols" platform="ios" target="*-Info.plist">
        <array>
            <string>com.brother.ptcbp</string>
        </array>
    </config-file>
    <config-file parent="NSLocalNetworkUsageDescription" platform="ios" target="*-Info.plist">
        <string>Used to discover printers on your network</string>
    </config-file>
    <config-file parent="NSBonjourServices" platform="ios" target="*-Info.plist">
        <array>
            <string>_printer._tcp</string>
            <string>_pdl-datastream._tcp</string>
            <string>_ipp._tcp</string>
        </array>
    </config-file>
```


## Usage

See here for JS interfaces to the plugin: `www/printer.js`

There are six available methods... 

* [findNetworkPrinters(success, failure)](#findnetworkprinters)
* [findBluetoothPrinters(success, failure)](#findbluetoothprinters)
* [findPrinters(success, failure)](#findprinters)
* [setPrinter(printer, success, failure)](#setprinter)
* [printViaSDK(data, success)](#printviasdk)
* [sendUSBConfig(data, success)](#sendusbconfig)

### findNetworkPrinters

Upon success, [`findNetworkPrinters`](#findNetworkPrinters) will provide a list of printers that were discovered on the network (likely using WiFi). It is not considered an error for no printers to be found, and in this case the list will just be empty.

```typescript
function findNetworkPrinters(success: (printers: Printer[]) => void, failure: (reason: string) => void): void
```

### findBluetoothPrinters

Upon success, [`findBluetoothPrinters`](#findBluetoothPrinters) will provide a list of printers that were discovered that have already been paired via Bluetooth. It is not considered an error for no printers to be found, and in this case the list will just be empty.

```typescript
function findBluetoothPrinters(success: (printers: Printer[]) => void, failure: (reason: string) => void): void
```

### findPrinters

[`findPrinters`](#findPrinters) is a convenience function that will perform the actions of both [`findNetworkPrinters`](#findNetworkPrinters) and [`findBluetoothPrinters`](#findBluetoothPrinters), and combine the the results into a single continuous list.

```typescript
function findPrinters(success: (printers: Printer[]) => void, failure: (reason: string) => void): void
```

### setPrinter

must be called before [`printViaSDK`](#printViaSDK). It takes a single object that should be one of the objects returned from [`findNetworkPrinters`](#findNetworkPrinters), [`findBluetoothPrinters`](#findBluetoothPrinters), or [`findPrinters`](#findPrinters). Upon successfully setting the printer, the success callback
will be invoked.  Otherwise, the error callback will be invoked with a string for an error message.

```typescript
function setPrinter(printer: Printer, success: () => void, failure: (reason: string) => void): void
```

### printViaSDK

takes one parameter, which is a base64 encoded bitmap image. The result should be a status code that is passed directly from the SDK. The status codes are documnted in the Brother SDK Appendix in section 4.2.2.5.Error Code. If everything works, the response should be `"ERROR_NONE"`.

__Clarification__:
> A bitmap image in this case can be any image with an encoding that is supported by the platform.


```typescript
function printViaSDK(data: string, success: () => void): void
```

### sendUSBConfig

calls the Brother SDK's `printFile` method. The expected input is a string containing raw print commands, which is written to a temporary file in the app cache directory, and is then sent to the `printFile` method and deleted afterwards. You will need a device that supports USB-OTG and a USB-OTG cable. On first run the app will request USB permissions, and it should be saved after that for subsequent prints. As-is, this method is used to send raw commands in PCL (Printer Control Language) to the printer... For example, to configure the network settings of the printer, etc... You will need to reach out to Brother for documentation of the PCL commands. You can probably find them by searching for "[Brother Printer Command Reference](https://duckduckgo.com/?q=Brother+Printer+Command+Reference)" and appending your model number. This method could be extended easily to accept other types of file input, so you could, for example, print JPG images, etc... See here for a simple way to generate a PJL file to reconfigure the network: https://github.com/gordol/PJL-Generator


```typescript
function sendUSBConfig(data: string, success: () => void): void
```
### Interface Reference

```typescript
interface Printer {
    model: string // Usually of the form 'QL_720NW' on Android
    port: 'NET' | 'BLUETOOTH'
    modelName: string // Usually of the form 'Brother QL-720NW'
    ipAddress?: string
    macAddress?: string
    serialNumber?: string
    nodeName?: string
    location?: string
    paperLabelName?: string // 'W17H54'|'W17H87'|'W23H23'|'W29H42'|'W29H90'|'W38H90'|'W39H48'|'W52H29'|'W62H29'|'W62H100'|'W12'|'W29'|'W38'|'W50'|'W54'|'W62'|'W60H86'|'W54H29'|'W62RB'
    numberOfCopies?: number // Default value 1
}
```

### Sample Code
```

declare var cordova: any //needed if using capacitor

 private setAndPrint(thePrinter: Printer, dataUrl: string) {
        // the dataUrl param is coming from  an HTML canvas HTMLCanvasElement.toDataURL()
        console.debug(`===== in setAndPrint===`)

        thePrinter.orientation = 'LANDSCAPE'
        thePrinter.paperLabelName = 'W62'
        thePrinter.ipAddress = 'YOUR_IP_ADDRESS'

        cordova.plugins.brotherPrinter.setPrinter(thePrinter, (success) => {
            console.debug(`===== set printer ok`)

            let separatorIdx: number = dataUrl.indexOf(',');

            if (separatorIdx != -1) {
                dataUrl = dataUrl.substring(separatorIdx + 1);
            }

            cordova.plugins.brotherPrinter.printViaSDK(dataUrl, (success) => {
                console.debug(`===== in printViaSDK ok`)

                if (success && success['result'] && success['result'] != "ERROR_NONE" && success['result'].indexOf('ERROR') != -1) {
                    console.error(`Printing on ${this.toString(thePrinter)} returned message : ${JSON.stringify(success)}`)
                } else {
                    console.debug(`=====print via sdk ok!!!`)
                    //this.toastService.showInfo(`Badge printed successfully`)
                }


            }, (err) => {
                console.error(`Failed to print via sdk`)
                console.error(`Printing failed on ${this.toString(thePrinter)} with message : ${JSON.stringify(err)}`)

            })
        }, (err) => {
            console.error(`====error failed to set printer: "${JSON.stringify(err)}"`)
            console.error(`Failed to set printer to ${this.toString(thePrinter)} with message : ${JSON.stringify(err)}`)

        })


        console.debug(`===== in setAndPrint END ===`)

    }

    private toString(thePrinter: Printer): string {
        if (!thePrinter) {
            return ''
        }

        return `ip:${thePrinter.ipAddress}, model:${thePrinter.model}, port: ${thePrinter.port}`
    }
  
}
```

## iOS notes:
* You may get errors while compiling: you need to make sure the native lib is "embed and sign" .
 see https://github.com/AbobosSoftware/cordova-plugin-brother-label-printer/issues/39

* If you are getting errors about unsupported architectures, please look at
 http://ikennd.ac/blog/2015/02/stripping-unwanted-architectures-from-dynamic-libraries-in-xcode/
