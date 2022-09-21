# Configuración del entorno Android

## Tabla de contenido 

1. [Instalación de Android Studio](#instalación-de-android-studio)
2. [Configuración de gradle](#configuración-gradle)
3. [Variables de entorno](#agregar-variables-de-entorno)
4. [Agregar ficheros de firma y credenciales](#agregar-los-ficheros-de-firma-y-credenciales)
5. [Generación del empaquetado](#generación-del-empaquetado)
6. [Enlace simbolico](#generación-del-enlace-simbolico)
7. [Configurar gradle.java.home](#configurar-gradlejavahome)
8. [Script para construcción nativa](#script-para-ejecutar-la-construcción-nativa)

## Instalación de Android Studio 

Usar la [siguiente guía de instalación](https://medium.com/@gracenikole/c%C3%B3mo-instalar-el-ide-android-studio-en-linux-ubuntu-4f0eb5a80f18)

## Configuración gradle

* Crear el archivo **gradle.properties** en la carpeta **~/.gradle/**

```
cd ~
mkdir .gradle && cd .gradle
<your_editor> gradle.properties
```

* Agregar lo siguiente en el archivo **gradle.properties**

```
artifactory_user=<USER>
artifactory_password=<API_KEY>
artifactory_contextUrl=https://globaldevtools.bbva.com/artifactory
artifactory_url=https://globaldevtools.bbva.com/artifactory-api/cells-native
#globaldevtools.bbva.com
```

El `<USER>` y `<API_KEY>` se obtienen del [artifactory](https://globaldevtools.bbva.com/artifactory/webapp/#/profile) 

## Agregar variables de entorno

* Crear el fichero **~/.bash_profile**
* Crear el fichero **~/.zshrc**

Agregar el siguiente contenido a ambos ficheros

```
export ANDROID_HOME=$HOME/Library/Android/sdk # Ruta SDK Android Mac. En otra plataforma, buscar la ubicación
ANDROID_BUILD_TOOLS_VERSION=$(ls $ANDROID_HOME/build-tools | tail -1 | awk '{print $NF}')
export PATH="$PATH:./" # Prevent the need of write ./ before executable files.
export PATH="$PATH:$ANDROID_HOME/emulator"
export PATH="$PATH:$ANDROID_HOME/platform-tools" # 'adb' command
export PATH="$PATH:$ANDROID_HOME/tools/proguard/bin" # Proguard and Retrace
export PATH="$PATH:$ANDROID_HOME/build-tools/$ANDROID_BUILD_TOOLS_VERSION" # 'aapt' command
```

## Agregar los ficheros de Firma y credenciales

* En la raíz del proyecto [glomo-android](https://globaldevtools.bbva.com/bitbucket/projects/GLOMO/repos/glomo-android/browse) agregar el archivo [glomo-hybrid.keystore](https://drive.google.com/file/d/1VBirK6Yr17RQuHzHxQdutHMhLa6bRq1U/view) y el archivo [signing.properties](https://drive.google.com/file/d/1bwXpsvCedZT-AKHQSSWrfVD9v9ZcW2S6/view)



## Generación del empaquetado

* Depurar archivos en componentes/

```
rm -rf components/*/test && rm -rf components/*/demo && rm -rf components/*/coverage-reports && rm -rf components/*/test && rm -rf components/*/test_e2e/node_modules && rm -rf components/*/test_e2e@tmp && rm -rf components/*/gdtoollink && rm -rf components/*/screenshots
```

* Depurar archivos node_modules/

```
rm -rf node_modules/*/*/coverage-reports && rm -rf node_modules/*/*/demo && rm -rf node_modules/*/*/test && rm -rf node_modules/*/*/parse-json-* && rm -rf node_modules/*/*/test_e2e/node_modules && rm -rf node_modules/*/*/test_e2e@tmp && rm -rf node_modules/*/*/screenshots
```

* Ejecutar comando para generar el empaquetado 

```
cells app:build -c pe/android-dev.js -b novulcanize
```

## Generación del enlace simbolico

```
mkdir "$ANDROID_PROJECT/app/src/peruDesa/assets"
unlink "$ANDROID_PROJECT/app/src/peruDesa/assets/www"
ln -s "$WEB_PROJECT/build/pe/android-dev/novulcanize/" "$ANDROID_PROJECT/app/src/peruDesa/assets/www"
ls -la "$ANDROID_PROJECT/app/src/peruDesa/assets/"
```

## Configurar gradle.java.home

* Descargar [Java 11](https://www.openlogic.com/openjdk-downloads?field_java_parent_version_target_id=406&field_operating_system_target_id=All&field_architecture_target_id=All&field_java_package_target_id=All), escoger el archivo **tar.gz**
* Descomprimir y renombrar por `openjdk`
* En el archivo [gradle.properties](#configuración-gradle) agregar 
```
org.gradle.java.home=<PATH>/openjdk/
```
Donde `PATH` es la ruta donde se encuentra la carpeta

Con esta configuración se pueden utilizar los comandos de **./gradlew**

* `./gradlew assemblePeruDesaDebug` para generar el apk
* `./gradlew installPeruDesaDebug` para instalar en emulador o dispositivo fisico

## Script para ejecutar la construcción nativa

```bash
#!/bin/bash
listmodos="develop novulcanize vulcanize"
listnativo="ios android"
novulcanizepath=$PWD/build/pe/android-dev/novulcanize/
androidpathwww=~/Documents/Glomo/glomo-android/gl-app-config/src/peruDesa/assets/www #change for your android path
currentpath=$PWD/dist/
modo=$1
nativo=${2:-android}
if [[ -z "$modo" ]]; then
	echo "Falta el modo de construcción, ejecutar con {{modo}} {{native}} (native default android)"
	exit 1
fi
if [[ $listmodos != *$modo* ]]; then
	echo "$modo no es reconocido, utiliza uno de los siguientes: [$listmodos]"
	exit 1
fi
if [[ $listnativo != *$nativo* ]]; then
	echo "$nativo no es reconocido, utiliza uno de los siguientes: [$listnativo]"
	exit 1
fi
if [[ $modo == "novulcanize" ]]; then
	currentpath=$novulcanizepath
fi
if [[ $currentpath != */glomo-pe/* ]]; then
	echo "Debes ejecutar el script en la raiz del proyecto glomo-pe"
	exit 1
fi
rm -rf components/*/test && rm -rf components/*/demo && rm -rf components/*/coverage-reports && rm -rf components/*/test && rm -rf components/*/test_e2e/node_modules && rm -rf components/*/test_e2e@tmp && rm -rf components/*/gdtoollink && rm -rf components/*/screenshots
echo "Carpeta components/ depurada!"
rm -rf node_modules/*/*/coverage-reports && rm -rf node_modules/*/*/demo && rm -rf node_modules/*/*/test && rm -rf node_modules/*/*/parse-json-* && rm -rf node_modules/*/*/test_e2e/node_modules && rm -rf node_modules/*/*/test_e2e@tmp && rm -rf node_modules/*/*/screenshots
echo "Carpeta node_modules/ depurada!"
echo "Ejecutando cells app:build -c pe/$nativo-dev.js -b $modo"
cells app:build -c pe/$nativo-dev.js -b $modo
echo "Creando enlace simbolico con $currentpath"
unlink $androidpathwww
ln -s $currentpath $androidpathwww
```

