# a2dp-sink-demo
動いて

# 動かない
bluetoothをコンテナ側から乗っ取るのはかなりきつそう

# ネイティブで構築
## 書き換え

```diff:/lib/systemd/system/bluetooth.service
+ ExecStart=/usr/libexec/bluetooth/bluetoothd --plugin=a2dp --compat --noplugin=sap
- ExecStart=/usr/libexec/bluetooth/bluetoothd
```


