# Reference

[https://medium.com/@dcostalloyd90/automating-android-builds-with-github-actions-a-step-by-step-guide-2a02a54f59cd](https://medium.com/@dcostalloyd90/automating-android-builds-with-github-actions-a-step-by-step-guide-2a02a54f59cd)

# Getting Started

## For Android

### Step 1: Create **.github/workflows/android-ci-cd.yml** and add code bellow

```sh
name: Build and Upload Android APK

on:
  push:
    branches: [main]  # Replace with your desired branches

jobs:
  build-and-upload:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      # Install Java
      - name: Install Java
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: adopt
          cache: gradle
      

      # Set up Node.js environment (adjust versions as needed)
      - name: Set up Node.js environment
        uses: actions/setup-node@v3
        with:
          node-version: 18

      # Install dependencies
      - name: Install dependencies
        run: npm install

      # configure cash for gradle : will help to reduce build time
      - name: Cache Gradle Wrapper
        uses: actions/cache@v2
        with:
          path: ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-wrapper-${{ hashFiles('gradle/wrapper/gradle-wrapper.properties') }}

      - name: Cache Gradle Dependencies
        uses: actions/cache@v2
        with:
          path: ~/.gradle/caches
          key: ${{ runner.os }}-gradle-caches-${{ hashFiles('gradle/wrapper/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-caches-

      - name: Make Gradlew Executable
        run: cd android && chmod +x ./gradlew

      - name: Decode Keystore
        env:
          ENCODED_STRING: ${{ secrets.KEY_STORE_SIGN }}
          MYAPP_UPLOAD_STORE_PASSWORD: ${{ secrets.MYAPP_UPLOAD_STORE_PASSWORD }}
          MYAPP_UPLOAD_KEY_ALIAS: ${{ secrets.MYAPP_UPLOAD_KEY_ALIAS }}
          MYAPP_UPLOAD_KEY_PASSWORD: ${{ secrets.MYAPP_UPLOAD_KEY_PASSWORD }} 

        run: |
          cd android/app && echo $ENCODED_STRING > keystore-b64.txt
          base64 -d keystore-b64.txt > my-upload-key.keystore

      # Build Android APK
      - name: Generate App APK
        env:
          MYAPP_UPLOAD_STORE_PASSWORD: ${{ secrets.MYAPP_UPLOAD_STORE_PASSWORD }}
          MYAPP_UPLOAD_KEY_ALIAS: ${{ secrets.MYAPP_UPLOAD_KEY_ALIAS }}
          MYAPP_UPLOAD_KEY_PASSWORD: ${{ secrets.MYAPP_UPLOAD_KEY_PASSWORD }}
        run: cd android && ./gradlew assembleRelease --stacktrace

      # Upload APK as an artifact
      - name: Upload application
        uses: actions/upload-artifact@v3
        with:
          name: android-apk
          path: android/app/build/outputs/apk/release/*.apk
          # retention-days: 3



```

### Step 2. Create my-upload-key.keystore
   **Go to android/app folder**
   ```sh
      cd android/app
   ```
   **Create my-upload-key.keystore**
   ```sh
      sudo keytool -genkey -v -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
   ```
   with
   
   MYAPP_UPLOAD_STORE_FILE=my-upload-key.keystore
   MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
   MYAPP_UPLOAD_STORE_PASSWORD=*****
   MYAPP_UPLOAD_KEY_PASSWORD=*****

   At android/app folder run code bellow to convert my-upload-key.keystore to base64
   ```sh
      openssl base64 < my-upload-key.keystore | tr -d '\n' | tee my-upload-key.base64.txt
   ```

<img width="634" alt="image" src="https://github.com/user-attachments/assets/66d8df62-bc83-42bb-afa0-609030e6b4f0">



### Step 3. Config Release app

Go to **android/app/build.gradle** and add config

```sh
signingConfigs {
        ...
        release{
            storeFile file('my-upload-key.keystore')
            storePassword System.getenv('MYAPP_UPLOAD_STORE_PASSWORD')
            keyAlias System.getenv('MYAPP_UPLOAD_KEY_ALIAS')
            keyPassword System.getenv('MYAPP_UPLOAD_KEY_PASSWORD')
        }
}
```
```sh
buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }
        release {
            ...
            signingConfig signingConfigs.release
            ...
        }
    }
```

## Step 4. Config Github variables environment

1. Go to **github Project** and choose **Setting** 

  <img width="1416" alt="image" src="https://github.com/user-attachments/assets/8ad7d40f-600d-485f-b891-99cb58d13df3">

2. Then scroll down and choose **Secrets and Variables** in left panel and click **Actions** like bellow
   
   <img width="1216" alt="image" src="https://github.com/user-attachments/assets/a715a513-ab5a-4b4b-bf09-a6bfc01ec5fb">


2. At **Add environment serects** click **New responsitory secrets**

   ```
   KEY_STORE_SIGN: get in my-upload-key.base64.txt (get at Step 2)
   MYAPP_UPLOAD_KEY_ALIAS: is MYAPP_UPLOAD_KEY_ALIAS (get at Step 2)
   MYAPP_UPLOAD_KEY_PASSWORD: is MYAPP_UPLOAD_KEY_PASSWORD (get at Step 2)
   MYAPP_UPLOAD_STORE_PASSWORD: is MYAPP_UPLOAD_STORE_PASSWORD (get at Step 2)
   ```

  <img width="1164" alt="image" src="https://github.com/user-attachments/assets/0687e5e9-739f-4b63-867b-d2cca8f7dcdd">

  <img width="946" alt="image" src="https://github.com/user-attachments/assets/3c617337-2f3d-4e59-84aa-683e15822246">


## Step 5. Commit and push code to main

## Step 6. Go to github action and choose your pipeline and download .apk file

<img width="1342" alt="image" src="https://github.com/user-attachments/assets/65860344-8cdb-404e-8645-95b02e246865">

<img width="1409" alt="image" src="https://github.com/user-attachments/assets/88fcef25-0d41-482f-bda1-539d1ea233e0">





