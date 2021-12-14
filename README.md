# CICD - FirebaseApp Distribution

Aplicativo utilizado para realizar a parte de CICD usando o Firebase App Distribution, e assim realizar o deploy automatico da aplicacao.

## Keystore Properties

Como recomendacao do Google e necessario criar um arquivo para manipulas os dados da keyStore, porem antes e necessario criar o jks usando o terminal:
- keytool -genkey -v -keystore hello.jks -alias hello -keyalg RSA -keysize 2048 -validity 10000
- OBS: Apos rodar o comando vai criar o arquivo .jks entao deve ser movido para dentro da pasta "app"

```keystore.properties
   storePassword=SENHA
   keyPassword=CONFORM_SENHA
   keyAlias=NAME_FILEL_JKS
   storeFile=NAME_FILEL_JKS.jks
```

## Arquivo service-accout-firebase.json
- Acessar No Console do [Google Cloud Platform](https://console.cloud.google.com/projectselector2/iam-admin/serviceaccounts).
- Selecionar o projeto
- Criar uma conta de servico
- Preencher o formulario "Detalhes da Conta de Servico", clicar em continuar
- Conceda a essa conta de serviço acesso ao projeto no campo selecionar papel escolha a opcao "administrador do Firebase App Distribution - Agente de servico do SDK Admin de distribuicao de aplicativo do Firebase - Acesso de leitura como SDK Admin".
- Documentacao [Usar Gradle](https://firebase.google.com/docs/app-distribution/android/distribute-gradle) na opcao Usar as credenciais da conta de servico do Firebase. 
- Adicionar o arquivo .json na pasta /app

## Configuracao do gradle

Realizar a configuracao do gradle, usando os imports da lib do Firebase, o tipo de build e a configuracao do arquivo JKS.


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
                releaseNotesFile="app/releasenotes.txt" //Deve criar este arquivo para adicionar as notas da versao
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
- [Documentacao](https://firebase.google.com/docs/app-distribution/android/distribute-gradle)






