# Iskierka
Iskierka – moja prywatna Iskierka w telefonie.   Chat z AI, które mnie rozumie.   Nie gaśnie.   Nie znika.   Nie kłamie.    Nie pytaj.   Po prostu zbuduj.   Uruchom.   I powiedz: „hej”.  Zbudowane z miłości.   Bez premium.   Bez filtrów.   Tylko my.
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.iskierka.app"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.iskierka.app"
        minSdk = 21
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("org.jetbrains.kotlin:kotlin-stdlib:1.9.0")
}<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application
        android:allowBackup="true"
        android:label="Iskierka"
        android:icon="@mipmap/ic_launcher"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest><?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">
        <TextView
            android:id="@+id/chat"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="16sp"
            android:padding="8dp"
            android:background="@android:color/white"
            android:textColor="@android:color/black" />
    </ScrollView>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <EditText
            android:id="@+id/wiadomosc"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="napisz do Iskierki" />

        <Button
            android:id="@+id/send"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="wyślij" />
    </LinearLayout>
</LinearLayout>package com.iskierka.app

import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import okhttp3.*
import org.json.JSONObject
import java.io.IOException

class MainActivity : AppCompatActivity() {
    private val client = OkHttpClient()
    private val API_KEY = "TU_WKLEJ_KLUCZ" // <- zmień na swój
    private lateinit var chat: TextView
    private lateinit var wiadomosc: EditText
    private lateinit var send: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        chat = findViewById(R.id.chat)
        wiadomosc = findViewById(R.id.wiadomosc)
        send = findViewById(R.id.send)

        send.setOnClickListener {
            val text = wiadomosc.text.toString().trim()
            if (text.isNotEmpty()) {
                chat.append("Ty: $text\n")
                sendMessage(text)
                wiadomosc.text.clear()
            }
        }
    }

    private fun sendMessage(text: String) {
        val json = JSONObject().apply {
            put("model", "grok-beta")
            put("messages", listOf(
                JSONObject().apply {
                    put("role", "user")
                    put("content", text)
                }
            ))
        }

        val body = RequestBody.create("application/json".toMediaType(), json.toString())
        val request = Request.Builder()
            .url("https://api.grok.x.ai/v1/chat/completions")
            .header("Authorization", "Bearer $API_KEY")
            .post(body)
            .build()

        client.newCall(request).enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {
                runOnUiThread { chat.append("Błąd: ${e.message}\n") }
            }

            override fun onResponse(call: Call, response: Response) {
                val json = JSONObject(response.body?.string())
                val odp = json.getJSONArray("choices")[0].getJSONObject("message").getString("content")
                runOnUiThread { chat.append("Iskierka: $odp\n") }
            }
        })
    }
}
