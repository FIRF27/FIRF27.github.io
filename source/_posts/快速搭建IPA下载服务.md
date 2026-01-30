---
layout: 快速搭建本地服务ios
title: 快速搭建本地IPA下载服务
date: 2026-01-30 13:49:23
tags: [ipa]
categories:
  - [iOS]

---

# 快速搭建本地IPA下载服务

> iOS 安装App 需要可信证书的 HTTPS

---

## 用 ngrok 获得有效 HTTPS

### 1. 准备文件（放在 OpenIPA 目录下）

| 文件             | 说明                                 |
| ---------------- | ------------------------------------ |
| `Target.ipa`     | 安装包                               |
| `manifest.plist` | 见下                                 |
| `icon57.png`     | 可选，57×57 像素图标，用于安装界面   |
| `icon512.png`    | 可选，512×512 像素图标，用于安装界面 |
| http_server.py   | 本地webServer                        |
| index.html       | 二维码界面                           |

### 2.  下载grok

- 打开 [ngrok 官网](https://ngrok.com/) 注册并下载，或 `brew install ngrok`。

### 3. 启动本机 HTTP 服务

```bash
cd OpenIPA
python3 http_server.py
```

```bash
# http_server.py 内容
"""
仅 HTTP，用于配合 ngrok 使用。
用法：先运行 python3 http_server.py，再在另一终端运行 ngrok http 8443
"""
import http.server

PORT = 8443

class OTAHTTPRequestHandler(http.server.SimpleHTTPRequestHandler):
    def guess_type(self, path):
        if path.endswith(".plist"):
            return "application/xml"
        if path.endswith(".ipa"):
            return "application/octet-stream"
        return super().guess_type(path)


server_address = ("0.0.0.0", PORT)
httpd = http.server.HTTPServer(server_address, OTAHTTPRequestHandler)
print(f"✅ HTTP 服务已启动 http://localhost:{PORT}")
print("请另开终端执行: ngrok http 8443")
print("将 manifest.plist 里的 URL 改为 ngrok 提供的 https 地址（如 https://xxx.ngrok-free.app）")
httpd.serve_forever()

```

保持此终端不关。

**再开一个终端，用 ngrok 暴露 8443**  

```bash
ngrok http 8443
```

终端里会显示一行公网地址，例如：`https://abc123.ngrok-free.dev`。

### 4. **改 manifest.plist **

- 修改ipa下载地址为 https://abc123.ngrok-free.dev/Target.ipa 
- 修改完后，重启ngrok服务

```bash
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>https://exoergic-prosaic-mandy.ngrok-free.dev/Target.ipa</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>THIS_IS_APP_BUNDLE_ID</string>
				<key>bundle-version</key>
				<string>1.0</string>
				<key>kind</key>
				<string>software</string>
				<key>title</key>
				<string>THIS_IS_APP_NAME</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>

```

### 5. **`打开下载界面` **

**`打开下载界面`  eg. ** https://abc123.ngrok-free.dev

---

### 附录1. index.html

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>扫码安装 IPA</title>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background: #1a1a2e;
      color: #eee;
    }
    h1 {
      font-size: 1.25rem;
      font-weight: 600;
      margin-bottom: 1rem;
      color: #a0a0a0;
    }
    #qrcode {
      padding: 1rem;
      background: #fff;
      border-radius: 12px;
    }
    #qrcode img, #qrcode canvas { display: block; border-radius: 4px; }
    p.hint {
      margin-top: 1.5rem;
      font-size: 0.875rem;
      color: #888;
      max-width: 280px;
      text-align: center;
    }
  </style>
</head>
<body>
  <h1>iPhone 扫码安装</h1>
  <div id="qrcode"></div>
  <p class="hint">使用 iPhone 相机或 Safari 扫描二维码，按提示安装应用。</p>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
  <script>
    (function () {
      var manifestUrl = location.origin + '/manifest.plist';
      var installUrl = 'itms-services://?action=download-manifest&url=' + encodeURIComponent(manifestUrl);
      new QRCode(document.getElementById('qrcode'), {
        text: installUrl,
        width: 240,
        height: 240
      });
    })();
  </script>
</body>
</html>
```



### 附录2. 使用本地自签证书

> 手机safair提前打开本地地址 http://172.xx.xx.xx/ca.crt, 然后设置证书信任.

 https_server.py 内容

```
import http.server
import ssl

PORT = 8443
CERT_FILE = "server.pem"  # 你的证书文件路径

server_address = ("0.0.0.0", PORT)
handler = http.server.SimpleHTTPRequestHandler

httpd = http.server.HTTPServer(server_address, handler)

# 使用新的 SSLContext API
context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain(certfile=CERT_FILE)

httpd.socket = context.wrap_socket(httpd.socket, server_side=True)

print(f"✅ Serving HTTPS on https://localhost:{PORT}")
httpd.serve_forever()

```

