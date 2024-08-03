<img width="804" alt="image" src="https://github.com/user-attachments/assets/bae2c6bf-e83f-4213-bfa5-12e53cfabdb5"># Reference

1. [https://medium.com/@dcostalloyd90/automating-android-builds-with-github-actions-a-step-by-step-guide-2a02a54f59cd](https://medium.com/@dcostalloyd90/automating-android-builds-with-github-actions-a-step-by-step-guide-2a02a54f59cd)

2. [https://www.obytes.com/blog/react-native-ci-cd-github-action](https://www.obytes.com/blog/react-native-ci-cd-github-action)

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
            signingConfig signingConfigs.release // <-- Ad this line
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

## Step 6. In your github project Click Action tab and choose your pipeline and download .apk file

<img width="1358" alt="image" src="https://github.com/user-attachments/assets/e376d047-cf54-4658-bb27-3636f800974e">

<img width="1342" alt="image" src="https://github.com/user-attachments/assets/65860344-8cdb-404e-8645-95b02e246865">

<img width="1409" alt="image" src="https://github.com/user-attachments/assets/88fcef25-0d41-482f-bda1-539d1ea233e0">


<h1>We have completed CI/CD with  github action</h1>

# Advance
## add Firebase Distribute to github action

### 1. Add code bellow to android-ci-cd.yml file

```sh
# Distribute app to Firebase App Distribution for testing / use google play internal track if you have a google play account
- name: upload artifact to Firebase App Distribution
  uses: wzieba/Firebase-Distribution-Github-Action@v1
  with:
    appId: ${{secrets.ANDROID_FIREBASE_APP_ID}}
    serviceCredentialsFileContent: ${{ secrets.CREDENTIAL_FILE_CONTENT }}
    groups: testers
    file: android/app/build/outputs/apk/release/*.apk
```

<img width="1367" alt="image" src="https://github.com/user-attachments/assets/50434db1-2a70-4297-9ca9-fb32cb926170">


### Step 2: Add ANDROID_FIREBASE_APP_ID, CREDENTIAL_FILE_CONTENT

<img width="1183" alt="image" src="https://github.com/user-attachments/assets/de392d2d-7c62-443a-a209-ac349ac6e942">

# How to get ANDROID_FIREBASE_APP_ID

1. Go to [https://console.firebase.google.com/](https://console.firebase.google.com/)

2. Click Project Setting
   
<img width="439" alt="image" src="https://github.com/user-attachments/assets/48265335-b5f3-4156-87a4-a701053c6bd9">

3. Create an Android App and see **App ID** 

<img width="862" alt="image" src="https://github.com/user-attachments/assets/9798f3db-de1b-4094-9bc4-3141df56dd65">

4. Copy android **App ID** to **ANDROID_FIREBASE_APP_ID**


# How to get CREDENTIAL_FILE_CONTENT
1. Go to Service on Google cloud:

[https://console.cloud.google.com/iam-admin/serviceaccounts](https://console.cloud.google.com/iam-admin/serviceaccounts)

2. Click CREATE SERVICE ACCOUNT

   <img width="1080" alt="image" src="https://github.com/user-attachments/assets/fd053643-6cb0-4af5-b962-4693a935d841">

3. Then click your service account at:
   
   <img width="864" alt="image" src="https://github.com/user-attachments/assets/4c2d06eb-94ff-4c97-811d-4557786553d7">

4. Click **KEYS** -> **Add Key** -> **Create New Key** -> Choose **JSON** and Click **Create**

   Then **CREDENTIAL JSON** will automatically download
   
   <img width="1073" alt="image" src="https://github.com/user-attachments/assets/7630157d-a576-4ecc-b31f-404e742a8fbe">

   <img width="804" alt="image" src="https://github.com/user-attachments/assets/0419c318-a608-43c8-ac3b-5b9c9c7c32db">

6.  Copy the summary of **CREDENTIAL JSON** to **CREDENTIAL_FILE_CONTENT**
   
   <img width="376" alt="image" src="https://github.com/user-attachments/assets/fcc45081-7edf-421a-9ab3-99c30fee9d99">

  <img width="643" alt="image" src="https://github.com/user-attachments/assets/4d871c16-9382-4b0a-bd90-960c230f9d0c">




