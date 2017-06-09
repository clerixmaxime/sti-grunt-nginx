# Build de Askme-Backbone via imageSource

L'idée de l'image source est de construire une première fois l'application en utilisant l'imageBuilder sti-grunt-nginx et de réutiliser le résultat de cette image pour réinjecter les node_modules dans les builds qui vont suivre. Comme il est impératif de garder les node_modules dans le premier build, j'ai du recréer un image_builder qui ne les supprime pas comme dans l'image finale qui nous permet de gagner 200Mo.

## 1. Construire l'image sti-grunt-nginx "racine". Pour cela j'ai créé une nouvelle branche sur Github où je ne supprime pas les node_modules et src.

``` oc new-build https://github.com/clerixmaxime/sti-grunt-nginx.git#image-source --name=sti-grunt-nginx-root --strategy=docker --context-dir=/1.0 ```

## 2. Construire l'image sti-grunt-nginx "classique".

``` oc new-build https://github.com/clerixmaxime/sti-grunt-nginx.git --name=sti-grunt-nginx --strategy=docker --context-dir=/1.0 ```

## 3. Builder l'application une première fois avec l'image "racine"

``` oc new-build sti-grunt-nginx-root~https://github.com/clerixmaxime/askme-backbone.git --name=askme-backbone-root ```

## 4. Builder l'application à nouveau avec l'image "classique"

``` oc new-build sti-grunt-nginx~https://github.com/clerixmaxime/askme-backbone.git --name=askme-backbone ```

## 5. Avant de lancer le build, modifier le buildConfig avec le code suivant:

```
source:
    type: Git
    git:
      uri: 'https://github.com/clerixmaxime/askme-backbone.git'
    images:
      - from:
          kind: ImageStreamTag
          namespace: backbone-optimization
          name: 'askme-backbone-root:latest'
        paths:
          - sourcePath: /opt/app-root/src/node_modules
            destinationDir: .
```

Cela permet de copier une partie des fichiers contenus dans l'image "racine" pour les injecter dans l'image "classique". Ici, on copie les node_modules.
