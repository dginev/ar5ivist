# ar5ivist

A turnkey command for converting a LaTeX source to ar5iv-style HTML, via [latexml-oxide](https://github.com/dginev/latexml-oxide),
the Rust port of LaTeXML that arXiv's ar5iv conversions run on.


## Recommended Build

Use the published dockerhub image (under a Unix OS) as (the `2512.17` image predates the move to latexml-oxide and runs
Perl LaTeXML; the local build below uses the latexml-oxide 0.8.0 release):

```bash
$ docker run -v "$(pwd)":/docdir -w /docdir \
             --user "$(id -u):$(id -g)" \
             latexml/ar5ivist:2512.17 --source=main.tex --destination=html/main.html
```

Grab a tea or coffee: the average conversion of an arXiv document today takes ~2 minutes, but ar5iv uses a max timeout of upto ~45 minutes.


## Local Build

The base image installs TeX Live from Ubuntu 24.04 and the latexml-oxide `.deb` from its GitHub release, checked
against the release's SHA-256 (`--build-arg LATEXML_OXIDE_VERSION=X.Y.Z` picks another release).

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
You can follow that example to create your own Dockerfile, with additional dependencies, or a reconfigured latexml-oxide setup
(`latexml_oxide --help` lists the options; the `ar5ivist` image passes the ar5iv profile: `--preload=ar5iv.sty`, HTML5
with MathML, and the release's bundled `ar5iv.css`).

The base image carries the TeX kernel formats prebuilt for its own TeX Live tree. A derived image that changes that
tree (another TeX Live, an updated `latex.ltx`) should rebuild them where `$LATEXML_DUMP_DIR` points, since `--init`
writes into `resources/dumps` under the current directory:
`RUN cd /usr/local/share/latexml-oxide && latexml_oxide --init=plain.tex && latexml_oxide --init=latex.ltx`.
Otherwise each run builds the format again (about 10 seconds). A derived image that sets its own `ENTRYPOINT` should
keep `/usr/bin/tini -s --` in front of `latexml_oxide`: run as the container's first process, latexml-oxide 0.8.0
cannot convert figures.

## Trust

A source tree can carry its own bindings: a `<package>.sty.rhai` beside the source is loaded by latexml-oxide like a
package binding. The `ar5ivist` image sets `LATEXML_DISABLE_SHELL_ESCAPE=1`, so such a binding cannot run programs;
it can still change the conversion. For sources you did not write, such as arXiv submissions, also run the container
without network access (`docker run --network=none ...`); note that the HTML links ar5iv's webfonts from a CDN.

## Troubleshooting

If the installation has succeeded, the ar5ivist run of latexml-oxide will produce a log file under the `.latexml.log` extension. For `main.tex`, that would be `main.latexml.log`.

While latexml Warnings generally do not harm the fidelity of the HTML5 document, `Error` and `Fatal` reports do, and should ideally be avoided.
In cases where you find troubleshooting a conversion run too obscure, please let us know by [opening a new issue](https://github.com/dginev/ar5ivist/issues).

We should be able to provide some support, and eventually make the reporting interface more convenient.
