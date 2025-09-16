# ar5ivist

A turnkey command for converting a LaTeX source to ar5iv-style HTML, via LaTeXML.


## Recommended Build

Use the published dockerhub image (under a Unix OS) as:

```bash
$ docker run -v "$(pwd)":/docdir -w /docdir \
             --user "$(id -u):$(id -g)" \
             latexml/ar5ivist:2509.16 --source=main.tex --destination=html/main.html
```

Grab a tea or coffee: the average conversion of an arXiv document today takes ~2 minutes, but ar5iv uses a max timeout of upto ~45 minutes.


## Local Build

build with:
```bash
$ cd ar5ivist-base
$ docker build --tag ar5ivist-base:latest .
$ cd ../ar5ivist
$ docker build --tag ar5ivist:latest .
```

run (under a Unix OS) with:
```
$ docker run -v "$(pwd)":/docdir -w /docdir \
             --user "$(id -u):$(id -g)" \
             ar5ivist:latest --source=main.tex --destination=html/main.html
```

where `main.tex` is the name of your main document source, and `html/main.html` names the HTML5 destination file, with  an (optional) destination directory.

Note that Docker will not be able to escape from the current directory from which you are running the command, so paths using a leading `../` will not work.

## Container Customization

The `ar5ivist-base` Dockerfile was extracted for easy customization in downstream Dockerfiles. The new `ar5ivist` Dockerfile is a thin wrapper over the base container.
You can follow that example to create your own Dockerfile, with additional dependencies, or completely reconfigured LaTeXML setup.

## Troubleshooting

If the installation has succeeded, the ar5ivist run of LaTeXML will produce a log file under the `.latexml.log` extension. For `main.tex`, that would be `main.latexml.log`.

While latexml Warnings generally do not harm the fidelity of the HTML5 document, `Error` and `Fatal` reports do, and should ideally be avoided.
In cases where you find troubleshooting a conversion run too obscure, please let us know by [opening a new issue](https://github.com/dginev/ar5ivist/issues).

We should be able to provide some support, and eventually make the reporting interface more convenient.
