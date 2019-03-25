# 機密情報の暗号化(Android)
暗号化が必要なファイルは下記3個。それをopensslコマンドで暗号化する
- google-services.json // firebaseへの接続情報
- publisher-keys.json // PlayStoreへのキー
- release.jks // リリース用署名

それぞれ下記のコマンドで暗号化する
```
openssl aes-256-cbc -e -in release.jks -out encrypted-release.jks
openssl aes-256-cbc -e -in google-services.json -out encrypted-google-services.json
openssl aes-256-cbc -e -in publisher-keys.json  -out encrypted-publisher-keys.json
```

リリースCI内で下記のようにそれぞれ復号化して使用する
```
deploy:
  stage: deploy
  script:
    # それぞれ復号化
    - openssl aes-256-cbc -k $KEYSTORE_DECRYPT_PASSWORD -d -in encrypted-release.jks -out release.jks
    - openssl aes-256-cbc -k $JSON_DECRYPT_PASSWORD -d -in encrypted-google-services.json -out app/google-services.json
    - ./gradlew :app:uploadDeployGateRelease
    # 以降デプロイ処理～～～
    - echo "Deploy to Google Play"
    - openssl aes-256-cbc -k $PUBLISHER_KEYS_JSON_DECRYPT_PASSWORD -d -in encrypted-publisher-keys.json -out app/publisher-keys.json
    - ./gradlew publishApkRelease
  only:
    - master
    - release
```
