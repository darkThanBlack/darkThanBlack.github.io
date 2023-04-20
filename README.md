# darkThanBlack.github.io

## My practice is?

* branchs
  * main: ``newest README.md`` +  ``all .md sources in ./docs/`` + ``mkdocs.yml``
  * gh-pages: all webside sources, generate by MKDocs CI
  * pictures: image hosts, generate by PicGo CI
* ignore
  * ``./site/``



## What tool did I use?



#### [MKDocs](https://www.mkdocs.org/)

```shell
# create
$ mkdocs new my-project

# Local host: http://127.0.0.1:8000/
$ mkdocs serve

# File path: ./site/
$ mkdocs build

# Github CI branch: gl-pages
$ mkdocs gh-deploy
```

* [Doc: Theme-Material](https://squidfunk.github.io/mkdocs-material/reference/)

  ```shell
  # install
  $ pip install mkdocs-material
  
  # third part plugins
  $ pip install mkdocs_pymdownx_material_extras
  ```

* [Doc: Keywords](https://www.mkdocs.org/user-guide/configuration/#introduction)



#### Image hosts

* Current: Github
  * CI branch: ``pictures``
  * CI Tool: [PicGo](https://github.com/Molunerfinn/PicGo)
    * [issue 781](https://github.com/Molunerfinn/PicGo/issues/781#issuecomment-1008603421)
  * image path: ``./docs/assets/pictures/``



> Sina weibo

 ``https://tva1.sinaimg.cn/large/008i3skNly1gy8f3g2v9fj301s01sdfm.jpg``

![sina weibo](https://tva1.sinaimg.cn/large/008i3skNly1gy8f3g2v9fj301s01sdfm.jpg)
<img src="https://tva1.sinaimg.cn/large/008i3skNly1gy8f3g2v9fj301s01sdfm.jpg" alt="Sina weibo" style="zoom:20%;" />



> Github

``https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/logo_64.png``

![Github](https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/logo_64.png)
<img src="https://raw.githubusercontent.com/darkThanBlack/darkThanBlack.github.io/pictures/docs/assets/pictures/logo_64.png" alt="Github" style="zoom:20%;" />



