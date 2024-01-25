# Blog

Es muy importante ir a leer la documentación del theme que estamos usando: [Enlace al tema de GitHub](https://github.com/Lednerb/bilberry-hugo-theme/tree/master/v4)

## Levantar localmente

Para hacerlo debemos de ir al root del proyecto y correr los siguientes comandos que inicializaran el submodulo y lo descargaran:

```bash
git submodule init
git submodule update
```

Una vez que tenemos el submodulo descargado, debemos de ir a la carpeta `blog` y ejecutar el siguiente comando:

```bash
cd blog
hugo server --ignoreCache --poll 700ms --cleanDestinationDir --gc
```
