---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS
css: unocss
---

# FFmpegKit を使ってみよう

Android アプリで収録音声にフィルターを掛ける

<div class="abs-br m-6 flex gap-2">
<p>おぎ</p>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# Agenda

- **Android アプリのプロジェクト構成説明**
- **FFmpegKit の導入**
- **FFmpeg コマンドの利用**
- **Demo**
- **エミュレータでデバッグ時の注意点**
  <br>
  <br>

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Here is another comment.
-->

---

# **Android アプリのプロジェクト構成説明**

<div grid="~ cols-2 gap-4">
<div>

### 主に利用するのは以下の 3 箇所

- **Activity 及び Fragment ファイル**
  - 表示制御やメソッドの呼び出し
- **ViewModel**
  - ロジック等を記述
- **Layout ファイル**
  - 画面の見た目を記述
  - UI でも編集可能

</div>
<div>

<img border="rounded" width="300" src="/images/FolderDirectory.png">

</div>
</div>
---

# **Android アプリのプロジェクト構成説明**

<div grid="~ cols-2 gap-4">
<div>

### 主に利用するのは以下の 3 箇所

- **Activity 及び Fragment ファイル**
  - 表示制御やメソッドの呼び出し
- **ViewModel**
  - ロジック等を記述
- **Layout ファイル**
  - 画面の見た目を記述
  - UI でも編集可能

</div>
<div>

<img border="rounded" src="/images/LayoutFile.png">

</div>
</div>
---

# **Android アプリのプロジェクト構成説明**

<div grid="~ cols-2 gap-4">
<div>

### その他重要なファイル

- **Manifests ファイル**
  - アプリ起動時の設定やパーミッション等の設定
- **gradle ファイル**
  - アプリバージョンの設定や外部ライブラリの依存関係を記述

</div>
<div>

<img border="rounded" width="300" src="/images/FolderDirectory.png">

</div>
</div>
---

# **FFmpegKit の導入**

<div grid="~ cols-2 gap-4">
<div>
### FFmpegKit

FFmpeg を

- Android
- iOS
- Flutter
- macOS
  などで利用できるライブラリ

https://github.com/arthenica/ffmpeg-kit

</div>
<div>

<img border="rounded" width="300" src="/images/ffmpeg-kit-icon-v9.png">

</div>
</div>
---

# **FFmpegKit の導入**

### gradle ファイルへ利用する外部ライブラリを記述

```ts {8}
dependencies {
    implementation 'androidx.core:core-ktx:1.7.0'
    implementation 'androidx.appcompat:appcompat:1.4.2'
    implementation 'com.google.android.material:material:1.6.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.5.1'
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.5.1'
    implementation 'com.arthenica:ffmpeg-kit-full:5.1'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
}
```

---

# **FFmpegKit の導入**

### AndroidManifest.xml へマイク利用のパーミッション設定を記述

```xml {6}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.ffmpeg_practice">

    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
```

---

# **FFmpegKit の導入**

### AndroidManifest.xml へマイク利用のパーミッション設定を記述

```xml {6}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.ffmpeg_practice">

    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
```

導入はこれだけで完了！🎉

---

# **FFmpeg コマンドの利用**

FFmpegKit.execute("FFmpeg のコマンド") という形式で FFmpeg のコマンドを実行可能

```kotlin {1-5}
    fun echoFilter(filteredFileName: String) {
        val mediaInformation = FFprobeKit.getMediaInformation(fileName)
        Log.d("VOICE INFO BEFORE", mediaInformation.logsAsString)
        val removeSilence = FFmpegKit.execute("-y -i $fileName -af aecho=0.8:0.9:1000:0.3 $filteredFileName")
    }

    fun getDuration(): String {
        val mediaInformation = FFprobeKit.getMediaInformation(fileName)
        return mediaInformation.mediaInformation.duration
    }
```

- -y 出力ファイルを上書きする
- -i 入力ファイルのパス指定
- -af audio filter を指定

- 1000ms の delay を指定

---

# **FFmpeg コマンドの利用**

FFmpegKit.execute("FFmpeg のコマンド") という形式で FFmpeg のコマンドを実行可能

```kotlin {2,4}
    fun echoFilter(filteredFileName: String) {
        val mediaInformation = FFprobeKit.getMediaInformation(fileName)
        Log.d("VOICE INFO BEFORE", mediaInformation.logsAsString)
        val removeSilence = FFmpegKit.execute("-y -i $fileName -af aecho=0.8:0.9:1000:0.3 $filteredFileName")
    }

    fun getDuration(): String {
        val mediaInformation = FFprobeKit.getMediaInformation(fileName)
        return mediaInformation.mediaInformation.duration
    }
```

- -y 出力ファイルを上書きする
- -i 入力ファイルのパス指定
- -af audio filter を指定

- 1000ms の delay を指定

---

# **Demo**

参考:

- https://developer.android.com/guide/topics/media/mediarecorder?hl=ja#sample-code
- https://ffmpeg.org/ffmpeg-all.html#Examples-42

---

# **エミュレータでデバッグ時の注意点**

<div grid="~ cols-2 gap-4">
<div>

エミュレータの設定から  
ホストデバイスのマイクを使用する設定を ON にする必要がある

</div>
<div>

<img border="rounded" src="/images/emulatorTips.png">

</div>
</div>
---

# ご清聴ありがとうございました！
