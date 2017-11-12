
## 获取WiFi IP 的方法如下：
```
private String getLocalIpAddress() {
        //获取wifi服务
        WifiManager wifiManager = (WifiManager) getApplicationContext().getSystemService(WIFI_SERVICE);
        //判断wifi是否开启
        if (!wifiManager.isWifiEnabled()) {
            wifiManager.setWifiEnabled(true);
        }
        WifiInfo wifiInfo = wifiManager.getConnectionInfo();
        int ipAddress = wifiInfo.getIpAddress();
        return intToIp(ipAddress);
    }

    private String intToIp(int i) {

        return (i & 0xFF) + "." +
                ((i >> 8) & 0xFF) + "." +
                ((i >> 16) & 0xFF) + "." +
                (i >> 24 & 0xFF);
    }
```
需要注意的是

```
WifiManager wifiManager = (WifiManager) getApplicationContext().getSystemService(WIFI_SERVICE);
```
==WiFiManager 需要用Application context来获取，否则会报如下错误==

```
Error:(199) Error: The WIFI_SERVICE must be looked up on the Application context or memory will leak on devices < Android N. Try changing  to .getApplicationContext()  [WifiManagerLeak]
```

