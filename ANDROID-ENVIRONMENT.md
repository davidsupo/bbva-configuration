# Configuración del entorno Android

## Tabla de contenido 

1. [Instalación de Android Studio](#instalación-de-android-studio)
2. [Configuración de gradle](#configuración-gradle)
3. [Variables de entorno](#agregar-variables-de-entorno)
4. [Agregar ficheros de firma y credenciales](#agregar-los-ficheros-de-firma-y-credenciales)
5. [Generación del empaquetado](#generación-del-empaquetado)
6. [Enlace simbolico](#generación-del-enlace-simbolico)

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

* Borrar los archivos de test y demo en componentes bower

```
rm -rf components/*/test && rm -rf components/*/demo && rm -rf components/*/coverage-reports && rm -rf components/*/test && rm -rf components/*/test_e2e/node_modules && rm -rf components/*/test_e2e@tmp && rm -rf components/*/gdtoollink && rm -rf components/*/screenshots
```

* Borrar los archivos de test y demo en node_modules

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
unlink "$ANDROID_PROJECT/app/src/peruDesa/assets/www"build/pe/android-dev/novulcanize/
ln -s "$WEB_PROJECT/build/pe/android-dev/novulcanize/" "$ANDROID_PROJECT/app/src/peruDesa/assets/www"
ls -la "$ANDROID_PROJECT/app/src/peruDesa/assets/"
```


