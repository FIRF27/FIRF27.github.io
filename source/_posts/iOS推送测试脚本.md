---
title: iOS推送测试脚本
tags: [iOS]
categories:
  - [iOS]
date: 2026-03-03 17:51:53

---

# APNs推送测试脚本(p8)

```
#!/bin/bash
#
# APNs推送测试脚本 

# .p8密钥文件路径
AUTH_KEY="/Users/apple/Desktop/PushShell/AuthKey.p8"  

# key id        
KEY_ID="XXXXXXXXX"    
# tema id       
TEAM_ID="XXXXXXXXX"    
# app bundle id 
TOPIC="com.xxx.app"  

# device token
DEVICE_TOKEN="6484x5xx2xxx0e9ab1d89c22304f66cf8db5e1a5f52b682357586e2e34f3d597"  

# 开发环境：api.sandbox.push.apple.com；生产环境：api.push.apple.com
APNS_HOST="api.push.apple.com"      

# 推送内容
PAYLOAD='{"aps":{"alert":"Hello from APNs!","sound":"default","badge":1}}'

# 生成JWT
generate_jwt() {
    local header='{"alg":"ES256","kid":"'"$KEY_ID"'"}'
    local header_b64=$(printf '%s' "$header" | openssl base64 -e -A | tr -- '+/' '-_' | tr -d '=')
    
    local iat=$(date +%s)
    local payload='{"iss":"'"$TEAM_ID"'","iat":'"$iat"'}'
    local payload_b64=$(printf '%s' "$payload" | openssl base64 -e -A | tr -- '+/' '-_' | tr -d '=')
    
    local header_payload="${header_b64}.${payload_b64}"
    local signature=$(printf '%s' "$header_payload" | openssl dgst -binary -sha256 -sign "$AUTH_KEY" | openssl base64 -e -A | tr -- '+/' '-_' | tr -d '=')
    
    echo "${header_payload}.${signature}"
}

# 主函数
main() {
    echo "🔑 生成JWT Token..."
    TOKEN=$(generate_jwt)
    
    echo "📱 发送推送通知..."
    echo "目标设备: $DEVICE_TOKEN"
    echo "推送内容: $PAYLOAD"
    echo "------------------------"
    
    curl -v --http2 \
        --header "authorization: bearer $TOKEN" \
        --header "apns-topic: $TOPIC" \
        --header "apns-push-type: alert" \
        --data "$PAYLOAD" \
        "https://${APNS_HOST}/3/device/${DEVICE_TOKEN}"
    
    echo "------------------------"
}

main "$@"
```

