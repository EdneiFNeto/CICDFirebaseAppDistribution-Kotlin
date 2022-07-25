# CICD - FirebaseApp Distribution

Aplicativo utilizado para realizar a parte de CICD usando o Firebase App Distribution, e assim realizar o deploy automático da aplicação.

## Keystore Properties

Como recomendação do Google e necessário criar um arquivo para manípulas os dados da keyStore, porem antes é necessário criar o jks usando o terminal:
- keytool -genkey -v -keystore hello.jks -alias hello -keyalg RSA -keysize 2048 -validity 10000
- Criar o arquivo keystore.properties
- OBS: Apos rodar o comando vai criar o arquivo .jks entao deve ser movido para dentro da pasta "app"
- [Referencia-Publish-App](https://developer.android.com/studio/publish/app-signing)

```keystore.properties
   storePassword=SENHA
   keyPassword=CONFORM_SENHA
   keyAlias=NAME_FILEL_JKS
   storeFile=NAME_FILEL_JKS.jks
```

## Arquivo service-accout-firebase.json
- Acessar no Console do [Google Cloud Platform](https://console.cloud.google.com/projectselector2/iam-admin/serviceaccounts).
- Selecionar o projeto
- Criar uma conta de serviço
- Preencher o formulário "Detalhes da Conta de Serviço", clicar em continuar
- Conceda a essa conta de serviço acesso ao projeto no campo selecionar papel escolha a opção "administrador do Firebase App Distribution - Agente de serviço do SDK Admin de distribuição de aplicativo do Firebase - Acesso de leitura como SDK Admin".
- Documentação [Usar Gradle](https://firebase.google.com/docs/app-distribution/android/distribute-gradle) na opção Usar as credenciais da conta de serviço do Firebase. 
- Adicionar o arquivo .json na pasta /app

## Configuracao do gradle

Realizar a configuração do gradle, usando os imports da lib do Firebase, o tipo de build e a configuração do arquivo JKS.
- Criar o arquivo releasenotes.txt com as notas das versões. 

```build.gradle
//Project
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.google.gms:google-services:4.3.10'
        classpath 'com.google.firebase:firebase-appdistribution-gradle:2.1.1'
    }
}


//Module
apply plugin: 'com.google.gms.google-services'
apply plugin: 'com.google.firebase.appdistribution'

def keystorePropertiesFile = rootProject.file("app/keystore.properties")

def keystoreProperties = new Properties()

keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

android {
  ...
  signingConfigs {
     release {
         keyAlias keystoreProperties['keyAlias']
         keyPassword keystoreProperties['keyPassword']
         storeFile file(keystoreProperties['storeFile'])
         storePassword keystoreProperties['storePassword']
     }
 }
 
 buildTypes {
        release {
            ...
            signingConfig signingConfigs.release

            firebaseAppDistribution {
                appId="ID_DO_APLICATIVO"
                releaseNotesFile="app/releasenotes.txt" 
                groups="NOME_DO_GRUPO_DE_TESTE_CRIADO_NA_OPCAO_APP_DISTRIBUTION"
                serviceCredentialsFile="app/service-accout-firebase.json"
            }
        }
    }
}

dependencies {
    ...
    implementation platform('com.google.firebase:firebase-bom:29.0.2')
    implementation 'com.google.firebase:firebase-analytics'
}
```

## Rodar o comando no Terminal

Comando assinatura e deploy no Firebase App Distribuition.

- ./gradlew assembleRelease appDistributionUploadRelease
- [Documentação](https://firebase.google.com/docs/app-distribution/android/distribute-gradle)



## Criando Env
Para criar variáveis de ambiente para facilitar a segurança do projeto, basta seguir os passos a baixio:

- Abrir terminal
- Execute o comando: nano ~/.zshrc

Apos abrir o arquivo .zshrc vai até a parte de baixo

- export PROJECT_ID=PROJECT_ID (Localiozado das configs. do Firebase)
- export KEY_API=KEY_API (Key usada realizar a notificacao)
- export FIREBASE_TOKEN=ID_DO_APLICATIVO
- export JKS_FILE=/Users/USER_PROFILE/PROJECT_PATHr/app/file.jks

```build.gradle
   buildTypes {
        def KEY_API = System.getenv("KEY_API")
        def PROJECT_ID = System.getenv("PROJECT_ID")
        def FIREBASE_TOKEN = System.getenv("FIREBASE_TOKEN")

        debug {
            buildConfigField('String', 'KEY_API', '"' + KEY_API + '"')
            buildConfigField('String', 'PROJECT_ID', '"' + PROJECT_ID + '"')
        }

        release {
            buildConfigField('String', 'KEY_API', '"' + KEY_API + '"')
            buildConfigField('String', 'PROJECT_ID', '"' + PROJECT_ID + '"')

            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'

            signingConfig signingConfigs.release

            firebaseAppDistribution {
                appId = buildConfigField('String', 'FIREBASE_TOKEN', '"' + FIREBASE_TOKEN + '"')
                releaseNotesFile = "app/releasenotes.txt"
                groups = "Teste"
                serviceCredentialsFile = "app/service-accout-firebase.json"
            }
        }
    }
 ```
 
 ## Key SHA256
 - ./gradlew signingReport


