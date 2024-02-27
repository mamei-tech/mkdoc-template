# mkdoc project


## [Getting Started with MkDocs](https://www.mkdocs.org/getting-started/)

## Requirements 

Necessary extensions to build documentation:
```bash
pip3 install pymdown-extensions
```

To install the theme used by us:
```bash
pip install mkdocs-material
```

Localizing the theme:
```bash
pip install mkdocs[i18n]
```

## Commands

To start the server, run:
 ```bash
mkdocs build -f config/en/mkdocs.yml
```


To build the site:
 ```bash
mkdocs serve -f config/en/mkdocs.yml
```