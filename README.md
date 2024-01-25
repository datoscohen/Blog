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

Una explicación más detallada del proceso de trabajar con módulos se puede encontrar [aquí](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

## Extras

### Eliminar submodulo

Para eliminar un submodulo debemos de ejecutar el siguiente comando:

```bash
git rm <submodule path> && git commit
```

Se puede encontrar mayor información en este [link](https://git-scm.com/docs/gitsubmodules), dentro del apartado **FORMS**.
