# Criação Automática de Projeto Android com Gradle

Nesta aula, vamos explorar um código que automatiza a criação de um projeto Android básico com Gradle. Este boilerplate configura o projeto Android de forma automática, criando a estrutura de diretórios e arquivos necessários, além de configurar dependências, permissões e a integração com Firebase e WebView.

## Objetivo

O objetivo deste código é inicializar um projeto Android de forma rápida, já com as configurações essenciais, como:

- Configuração do Gradle para compilar o aplicativo.
- Inclusão de dependências como o Firebase e o WebView.
- Configuração de permissões necessárias, como acesso à internet, localização e notificações push.
- Estrutura básica de arquivos do Android, como o `AndroidManifest.xml` e a `MainActivity`.

## Explicação do Código

Vamos ver, passo a passo, o que cada parte do código faz.

### 1. Criação do diretório e configuração inicial

O código começa criando o diretório do projeto e configurando o Gradle com os arquivos essenciais:

```bash
mkdir desafioliga-app && cd desafioliga-app
Em seguida, cria a estrutura de diretórios:

bash
Copiar
Editar
mkdir -p android/app/src/main/java/com/desafioliga
Essa linha organiza o projeto para que possamos adicionar nosso código Java na pasta correta.

2. Arquivo settings.gradle
O settings.gradle define o nome do projeto e inclui o módulo do aplicativo:

bash
Copiar
Editar
cat > settings.gradle <<EOF
rootProject.name = 'desafioliga-app'
include ':app'
EOF
3. Arquivo build.gradle (Projeto)
Aqui, configuramos as dependências do projeto, como o repositório Maven Central e Google, além da dependência do Gradle para compilar o aplicativo:

bash
Copiar
Editar
cat > build.gradle <<EOF
buildscript {
  repositories { google(); mavenCentral() }
  dependencies { classpath 'com.android.tools.build:gradle:8.2.0' }
}
allprojects { repositories { google(); mavenCentral() } }
EOF
4. Arquivo build.gradle (Aplicativo Android)
Este arquivo contém as configurações específicas para o aplicativo Android, como o compileSdk, minSdk, targetSdk, e as dependências necessárias, como o Firebase e o WebView:

bash
Copiar
Editar
cat > android/app/build.gradle <<EOF
apply plugin: 'com.android.application'
android {
  namespace 'com.desafioliga'
  compileSdk 34
  defaultConfig {
    applicationId "com.desafioliga"
    minSdk 21
    targetSdk 34
    versionCode 1
    versionName "1.0"
  }
  buildTypes {
    release {
      minifyEnabled false
    }
  }
}
dependencies {
  implementation 'androidx.appcompat:appcompat:1.6.1'
  implementation 'android.webkit:webkit:1.7.0'
  implementation 'com.google.firebase:firebase-messaging:23.1.0'
}
EOF
Aqui, destacamos algumas dependências importantes:

WebView (android.webkit:webkit): Para carregar páginas web dentro do aplicativo.

Firebase Messaging (com.google.firebase:firebase-messaging): Para enviar notificações push.

5. Arquivo AndroidManifest.xml
O AndroidManifest.xml define as permissões necessárias para o funcionamento do aplicativo, como o acesso à internet e à localização, além de declarar a MainActivity como ponto de entrada:

bash
Copiar
Editar
cat > android/app/src/main/AndroidManifest.xml <<EOF
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.desafioliga">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    <application android:label="DesafioLigaJovem">
        <activity android:name=".MainActivity" android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
EOF
6. Arquivo MainActivity.java
A MainActivity.java é a atividade principal do aplicativo. Ela configura a WebView e solicita permissões ao usuário para acessar a localização e enviar notificações. Além disso, ela carrega a URL do aplicativo na WebView:

bash
Copiar
Editar
cat > android/app/src/main/java/com/desafioliga/MainActivity.java <<EOF
package com.desafioliga;

import android.Manifest;
import android.app.DownloadManager;
import android.content.pm.PackageManager;
import android.net.Uri;
import android.os.Bundle;
import android.webkit.*;
import androidx.activity.result.*;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;
import com.google.firebase.messaging.FirebaseMessaging;

public class MainActivity extends AppCompatActivity {
    private WebView webView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestPermissions();
        webView = new WebView(this);
        WebSettings ws = webView.getSettings();
        ws.setJavaScriptEnabled(true);
        webView.setWebViewClient(new WebViewClient());
        webView.setWebChromeClient(new WebChromeClient() {
            // localização
            @Override
            public void onPermissionRequest(PermissionRequest request) {
                request.grant(request.getResources());
            }
        });
        webView.setDownloadListener((url, ua, contentDisposition, mimeType, contentLength) -> {
            DownloadManager.Request req = new DownloadManager.Request(Uri.parse(url));
            req.allowScanningByMediaScanner();
            req.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);
            DownloadManager dm = (DownloadManager) getSystemService(DOWNLOAD_SERVICE);
            dm.enqueue(req);
        });
        webView.loadUrl("https://inscricao.desafioligajovem.com.br/login/email");
        setContentView(webView);

        // Firebase push token (base)
        FirebaseMessaging.getInstance().getToken()
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    String token = task.getResult();
                    System.out.println("FCM Token: " + token);
                }
            });
    }

    private void requestPermissions() {
        String[] perms = {Manifest.permission.POST_NOTIFICATIONS,
                          Manifest.permission.ACCESS_FINE_LOCATION};
        ActivityCompat.requestPermissions(this, perms, 1);
    }
}
EOF
7. Arquivo .gitignore
O arquivo .gitignore é usado para excluir arquivos desnecessários do repositório, como arquivos gerados pelo Android Studio e pelo Gradle:

bash
Copiar
Editar
cat > .gitignore <<EOF
*.iml
*.keystore
*/build/
EOF
Como Rodar o Projeto
Clone o repositório para o seu computador:

bash
Copiar
Editar
git clone <URL_DO_REPOSITORIO>
cd desafioliga-app
Abra o projeto no Android Studio ou no editor de sua preferência:

bash
Copiar
Editar
code .
Certifique-se de ter o Android SDK instalado.

Compile e instale o aplicativo:

bash
Copiar
Editar
./gradlew build
./gradlew installDebug
Conclusão
Este boilerplate automatiza a criação de um projeto Android básico com as configurações necessárias para carregar uma página web com WebView, integrar Firebase para notificações push e solicitar permissões essenciais. Ele serve como uma base para você começar a desenvolver um aplicativo Android com essas funcionalidades.

bash
Copiar
Editar

Esse README serve como uma aula explicando cada parte do código que automatiza a criação do projeto, de forma que você possa entender e aplicar em seus próprios projetos Android.








Perguntar ao ChatGPT
