# egarim
Lenovo Mirage VR180 Camera API client - remote control and custom live streaming
Lenovo Mirage VR180 Camera APIクライアント - リモートコントロールとカスタムライブストリーミング


## Introduction

The Lenovo Mirage Camera with Daydream is a twin-sensor VR180 camera which does 3D photos, videos and livestreaming at 4k. It uses the VR180 format popularised by Google for use with VR headsets. Unfortunately, the format and the camera seem to have been abandoned, but it's still available for about $100 and is great value for what it does. The camera lacks a viewfinder or preview screen and needs a smartphone companion app (Google's VR180 app available on Android and iOS) to function as a viewfinder, setup the parameters for livestreaming, etc. Only Youtube livestreaming is enabled by the VR180 app.

Lenovo Mirage Camera with Daydreamは、4kで3D写真、ビデオ、ライブストリーミングを行うツインセンサーVR180カメラです。これは、VRヘッドセットで使用するためにGoogleが普及したVR180フォーマットを使用しています。残念ながら、このフォーマットとカメラは放棄されてしまったようですが、まだ約100ドルで販売されており、それが何をするにしても素晴らしい価値があります。このカメラにはビューファインダーやプレビュー画面が欠けており、ビューファインダーとして機能したり、ライブストリーミング用のパラメータを設定したりするためには、スマートフォンのコンパニオンアプリ（AndroidとiOSで利用可能なGoogleのVR180アプリ）が必要です。VR180アプリで有効なのはYoutubeのライブストリーミングのみです。

The reference implementation of the camera firmware has been open-sourced by Google (https://github.com/google/vr180), but the companion app is not . This project provides a Linux/Python utilities which can function as a "companion app" to pair with the camera, issue API commands, setup live streaming with custom end-points.

カメラのファームウェアのリファレンス実装はGoogle (https://github.com/google/vr180)によってオープンソース化されていますが、コンパニオンアプリはオープンソース化されていません。このプロジェクトでは、カメラとのペアリング、APIコマンドの発行、カスタムエンドポイントでのライブストリーミングの設定などを行うための「コンパニオンアプリ」として機能するLinux/Pythonユーティリティを提供します。

## Installation

Instructions below apply to Ubuntu 18.04, YMMV for other distributions.

First, packages to be installed:
```
sudo apt-get install default-jdk libdbus-1-dev python3-dev libglib2.0-dev libcairo2-dev libgirepository1.0-dev
```

Run `make` to compile the Java files. These are taken from the Google VR180 camera reference implementation, so we get to use the exact same crypto code the camera is using. We can write python equivalents, but why bother? Life is short.

`make`を実行してJavaファイルをコンパイルします。これらはGoogle VR180カメラのリファレンス実装から取得したものなので、カメラが使用しているものと全く同じ暗号コードを使用することができます。pythonと同等のものを書くこともできますが、なぜ悩むのでしょうか？人生は短いのです。

Then setup the python environment.
```
virtualenv -p python3 /path/to/egarim-ve
. /path/to/egarim-ve/bin/activate
pip install dbus-python protobuf PyGObject
```

## Basic Usage

The camera uses an application level pairing protocol over Bluetooth which uses ECDH key agreement to establish a shared key. This key is then used to encrypt further API calls over Bluetooth, and/or to sign HTTP API calls over Wi-Fi.

カメラはBluetooth上でアプリケーションレベルのペアリングプロトコルを使用し、ECDHキー合意を使用して共有キーを確立します。このキーは、Bluetoothを介したAPIコールの暗号化や、Wi-Fiを介したHTTP APIコールの署名に使用されます。

Before using the camera we must pair it using `bluestrap.py` (works only on Linux). Once paired, the shared key may be used with `egarim.py` to issue API requests over Wi-Fi on any machine (Linux/MacOS)

カメラを使う前に、`bluestrap.py`を使ってペアリングする必要があります(Linuxでのみ動作します)。ペアリングが完了したら、共有鍵を `egarim.py` で使用して、どのマシン(Linux/MacOS)でもWi-Fi経由でAPIリクエストを発行することができます。

The LED surrounding the shutter button is in one of 4 states:
  1. Off - the camera is sleeping/powere off.
  2. Blinking green - the camera is booting.
  3. Solid blue - the camera is booted/awake and ready for action.
  4. Blinking green/blue - the camera is in pairing mode.
  
シャッターボタン周辺のLEDは4つの状態のいずれかです。
  1. オフ - カメラはスリープ/電源オフです。
  2. 緑の点滅 - カメラが起動しています。
  3. 青色の点灯 - カメラは起動中/起床中で、動作の準備ができています。
  4. 緑/青の点滅 - カメラはペアリングモードになっています。

### Pairing

This 'pairing' is not the same as Bluetooth pairing with a number code - don't bother going to your OS Bluetooth menu and looking for the camera to pair.

この「ペアリング」は、番号コードとBluetoothペアリングと同じではありません - あなたのOSのBluetoothメニューに行くとペアリングするカメラを探してわざわざ行く必要はありません。

Power on the camera. Once the shutter LED is in the solid blue state, hold the shutter button down for 5 seconds, until the LED enters the blinking blue/green state. Run

`python bluestrap.py pair`

カメラの電源を入れます。シャッターLEDが青一色の状態になったら、LEDが青/緑の点滅状態になるまでシャッターボタンを5秒間押し続けます。

`python bluestrap.py pair`

を実行します。

If successful, you will see a series of messages like

成功すると、次のような一連のメッセージが表示されます。

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

尋ねられたらシャッターを1回押すと，ペアリングが継続され，共有キーファイル `me_cam.skey` が生成されます．このファイルはカメラとの通信に必要になります。python bluestrap.py status`を実行して、ペアリングと認証情報が有効であることを確認します。

### Bluetooth API calls

Once pairing is done and we have the shared key, we can issue commands over Bluetooth or HTTP. Bluetooth is useful to pair, set the wifi creds, and check status to get the camera's IP address.

ペアリングが完了し、共有キーが手に入れば、BluetoothやHTTPでコマンドを発行することができます。Bluetoothは、ペアリング、wifi認証の設定、カメラのIPアドレスを取得するためのステータスチェックに便利です。

```
./bluestrap.py config_wifi --ssid <SSID> --password <password>
```

The camera's Wi-Fi LED lights up when connected. Use `status` to find its IP address. This command also caches the results to `~/.egarim-status`, so the IP address can be found for HTTP API calls.

接続するとカメラのWi-Fi LEDが点灯します。IPアドレスを調べるには `status` を使います。このコマンドは結果を `~/.egarim-status` にキャッシュするので、HTTP APIの呼び出しでIPアドレスを見つけることができます。

```
./bluestrap.py status

...
http_server_status {
    camera_hostname: "192.168.1.44"
    camera_port: 8443
    camera_certificate_signature: "0A\002.."
  }
...
```

To factory reset the camera,

カメラを工場出荷時にリセットするには

```
./bluestrap.py factory_reset
```

### HTTP API calls

To issue HTTP calls, we need the shared key as established by a previous bluetooth pairing process (by default, stored as `me_cam.skey`) and the IP address of the camera (found using `python bluestrap.py status`).

HTTPコールを発行するには、以前のbluetoothペアリングプロセスで確立された共有鍵（デフォルトでは `me_cam.skey` として保存されています）とカメラのIPアドレス（`python bluestrap.py status`で確認できます）が必要です。

To get the camera status using HTTP,

HTTPを使用してカメラの状態を取得するには

```
./egarim.py --host 192.168.1.44 status
```
In normal usage, `--host` is not required as the IP address is retrieved from the value from the last cached status received over Bluetooth. Run `bluestrap.py status` whenever you change Wi-Fi APs to refresh the status cache.

通常の使用法では、Bluetoothで最後に受信したステータスキャッシュの値からIPアドレスを取得するので、`--host`は必要ありません。Wi-Fi APを変更する際には常に `bluestrap.py status` を実行し、ステータスキャッシュを更新する。

### Taking a photo

To take a photo, we first need to set the capture mode to `photo`, (optionally) configure the resolution, and then initiate a capture.

写真を撮るには、まずキャプチャモードを `photo` に設定し、(オプションで)解像度を設定してからキャプチャを開始する必要があります。

```
./egarim.py config_capture -h
```

shows the list of available capture configuration options to be set. For photos, only `width` and `height` are relevant. At a minimum, we set the capture mode:

↑は設定可能なキャプチャ設定オプションのリストを示します。写真の場合は `width` と `height` だけが関係しています。最低限、キャプチャモードを設定します。

```
./egarim.py config_capture --mode photo
./egarim.py status | grep active_capture_type
```

The last command should print `active_capture_type: PHOTO`. To click,

最後のコマンドは `active_capture_type. PHOTO`と表示されるはずです。クリックします。

```
./egarim.py start_capture
```

You should hear a shutter noise as a photo is captured. Taking a video is similar, except there is an additional `egarim.py stop_capture` to terminate the video.

写真が撮影されるとシャッター音がするはずです。動画を撮るのも似ていますが、`egarim.py stop_capture`を追加して動画を終了させます。

### Custom Live Streaming

The VR180 companion app only lets you live stream to Youtube. To stream to a custom end-point, e.g. one running at `rtmp://192.168.1.43/egarim_sample` with the stream id `foo`,

VR180 のコンパニオンアプリでは、Youtube へのライブストリーミングしかできません。例えば、`rtmp://192.168.1.43/egarim_sample` でストリーム ID `foo` を指定して実行しているエンドポイントなど、カスタムのエンドポイントにストリームするには、次のようにします。

```
./egarim.py config_capture --mode live --rtmp_endpoint rtmp://192.168.1.43/egarim_sample --stream_name_key foo
# verify with status
# start capture
./egarim.py start_capture
# stop capture using the shutter button or the command below
./egarim.py stop_capture

```

If the camera goes to sleep, the live streaming parameters are lost, so always set them with `config_capture` before `start_capture`. Wifi connections in the 5GHz band are preferred for live streaming at high resolution. Many routers have different SSIDs for 2.4 and 5GHz - given a choice, pick the latter.

カメラがスリープ状態になるとライブストリーミングのパラメータが失われるので、`start_capture`の前に必ず`config_capture`で設定してください。高解像度のライブストリーミングを行うには、5GHz帯のWifi接続が望ましいです。多くのルーターは2.4GHzと5GHzで異なるSSIDを持っています。

### Setting up a Custom Live Stream Server

There are many RTMP server solutions; here is a minimal setup using Ubuntu 18.04 and nginx.

多くのRTMPサーバーソリューションがありますが、ここではUbuntu 18.04とnginxを使った最小限のセットアップを紹介します。

```
sudo apt-get install nginx libnginx-mod-rtmp ffmpeg
```

Edit `/etc/nginx/nginx.conf` and append the following lines to the bottom:

```
rtmp {
    server {
        listen 1935;
        application egarim_sample {
            live on;
            record off;
        }
    }
}
```

and reload nginx using `systemctl reload nginx`. Firewall issues are beyond the scope of this README.

そして、 `systemctl reload nginx` を使って nginx をリロードしてください。ファイアウォールの問題はこのREADMEの範囲を超えています。

Start live streaming from the camera using the steps described in the previous section, replacing the IP address in the example with the IP address of the server on which nginx is running. To view the live stream from the same machine, 

前のセクションで説明した手順で、例のIPアドレスをnginxが動作しているサーバーのIPアドレスに置き換えて、カメラからのライブストリーミングを開始します。同じマシンからライブストリームを表示するには 

```
ffplay -fflags nobuffer rtmp://localhost/egarim_sample/foo
```

The `nobuffer` flag reduces latency to about 500ms.

`nobuffer` フラグはレイテンシを約500msに短縮する。

### Media Management

To list, download and delete images and videos from internal storage, use the `list_media`, `get_media` and `delete_media` commands. Examples below:

内部ストレージから画像や動画を一覧表示したりダウンロードしたり削除したりするには、`list_media`, `get_media`, `delete_media` コマンドを利用します。以下に例を示します。

List files in camera internal storage
```
./egarim.py list_media
# each line contains pathname, file size, video duration in ms, width and height
/storage/emulated/0/DCIM/VR180/20200218-051539536.vr.mp4 1238976 5868 2560 1440
/storage/emulated/0/DCIM/VR180/20200218-051539556.vr.jpg 3243859 0 3016 3016
```

Download a file to /tmp
```
./egarim.py get_media --dest /tmp /storage/emulated/0/DCIM/VR180/20200218-051539536.vr.mp4
```

Delete a file in camera internal storage
```
./egarim.py delete_media /storage/emulated/0/DCIM/VR180/20200218-051539536.vr.mp4
```

Download all files to /tmp
```
for i in `./egarim.py list_media | awk '{print $1}'`; do
  ./egarim.py get_media --dest /tmp $i
done
```

Delete all files in camera internal storage
```
for i in `./egarim.py list_media | awk '{print $1}'`; do
  ./egarim.py delete_media $i
done
```

## Advanced Usage

Read `camera_api.proto` and implement new commands `¯\_(ツ)_/¯`

`camera_api.proto` を読み込んで、新しいコマンドを実装します。

## Technical details

## Credits

  1. Google's VR180 reference camera implementation: https://github.com/google/vr180
  2. Dash Zhou's ECDH key agreement project https://github.com/zhoupeng6d/openssl-key-exchange, which uses the same key generation mechanism, but very well-explained.
  3. Adafruit's BLE project for Bluetooth/Linux details: https://github.com/adafruit/Adafruit_Python_BluefruitLE
