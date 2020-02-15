# egarim
Lenovo Mirage VR180 Camera API client - remote control and custom live streaming

## Introduction

The Lenovo Mirage Camera with Daydream is a twin-sensor VR180 camera which does 3D photos, videos and livestreaming at 4k. It uses the VR180 format popularised by Google for use with VR headsets. Unfortunately, the format and the camera seem to have been abandoned, but it's still available for about $100 and is great value for what it does. The camera lacks a viewfinder or preview screen and needs a smartphone companion app (Google's VR180 app available on Android and iOS) to function as a viewfinder, setup the parameters for livestreaming, etc. Only Youtube livestreaming is enabled by the VR180 app.

The reference implementation of the camera firmware has been open-sourced by Google (https://github.com/google/vr180), but the companion app is not . This project provides a Linux/Python utilities which can function as a "companion app" to pair with the camera, issue API commands, setup live streaming with custom end-points.

## Installation

## Basic Usage

The camera uses an application level pairing protocol over Bluetooth which uses ECDH key agreement to establish a shared key. This key is then used to encrypt further API calls over Bluetooth, and/or to sign HTTP API calls over Wi-Fi.

Before using the camera we must pair it using `bluestrap.py` (works only on Linux). Once paired, the shared key may be used with `egarim.py` to issue API requests over Wi-Fi on any machine (Linux/MacOS)

The LED surrounding the shutter button is in one of 4 states:
  1. Off - the camera is sleeping/powere off.
  2. Blinking green - the camera is booting.
  3. Solid blue - the camera is booted/awake and ready for action.
  4. Blinking green/blue - the camera is in pairing mode.
  
### Pairing

This 'pairing' is not the same as Bluetooth pairing with a number code - don't bother going to your OS Bluetooth menu and looking for the camera to pair.

Power on the camera. Once the shutter LED is in the solid blue state, hold the shutter button down for 5 seconds, until the LED enters the blinking blue/green state. Run

`python bluestrap.py pair`

If successful, you will see a series of messages like

```
waiting for device (attempt 1 of 30)
waiting for device (attempt 2 of 30)
found camera
waiting for service endpoint (attempt 1 of 30)
key init request type: KEY_EXCHANGE_INITIATE
key_exchange_request {
  public_key: "\003\033..."
  salt: "\360\316.."
}

key response response_status {
  status_code: OK
}
request_id: 0
key_exchange_response {
  public_key: "\004\063..."
  salt: "\160\216.."
}

Pairing response received; press shutter key once within 5 seconds to confirm
key finalize request type: KEY_EXCHANGE_FINALIZE
key_exchange_request {
  public_key: "\003\033..."
  salt: "\360\316.."
}

key finalize response response_status {
  status_code: OK
}
request_id: 0

Pairing finalized! Shared encryption key written to  me_cam.skey
cleaning up..

```

Press the shutter once when asked, and the pairing will continue and generate the shared key file, `me_cam.skey`. This file will be needed for all further communication with the camera. Run `python bluestrap.py status` to check that the pairing and credentials are valid.

### Bluetooth API calls

Once pairing is done and we have the shared key, we can issue commands over Bluetooth or HTTP. Bluetooth is useful to pair, set the wifi creds, and check status to get the camera's IP address.

```
python bluestrap.py config_wifi --ssid <SSID> --password <password>
```

The camera's Wi-Fi LED lights up when connected. Use `status` to find its IP address.

```
python bluestrap.py status

...
http_server_status {
    camera_hostname: "192.168.1.44"
    camera_port: 8443
    camera_certificate_signature: "0A\002.."
  }
...
```

To factory reset the camera,

```
python bluestrap.py factory_reset
```

### HTTP API calls

## Advanced Usage

Read `camera_api.proto` and implement new commands `¯\_(ツ)_/¯`

## Technical details

## Credits

  1. Google's VR180 reference camera implementation: https://github.com/google/vr180
  2. Dash Zhou's ECDH key agreement project https://github.com/zhoupeng6d/openssl-key-exchange, which uses the same key generation mechanism, but very well-explained.
  3. Adafruit's BLE project for Bluetooth/Linux details: https://github.com/adafruit/Adafruit_Python_BluefruitLE
