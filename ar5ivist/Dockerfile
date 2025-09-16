## Dockerfile for ar5ivist, using a fixed LaTeXML commit
##
## build with
## $ cd ar5ivist-base
## $ docker build --tag ar5ivist-base:latest .
## $ cd ../ar5ivist
## $ docker build --tag ar5ivist:latest .
##
## run with
##
## $ docker run -v "$(pwd)":/docdir -w /docdir \
##              --user "$(id -u):$(id -g)" \
##              ar5ivist:latest --source=main.tex --destination=html/main.html

FROM ar5ivist-base:latest

# continue as instructed in https://www.howtogeek.com/devops/how-to-use-docker-to-package-cli-applications/
ENTRYPOINT ["latexmlc", \
  "--preload=[nobibtex,rawstyles,nobreakuntex,magnify=1.3,zoomout=1.3,tokenlimit=249999999,iflimit=3599999,absorblimit=1299999,pushbacklimit=599999]latexml.sty", \
  "--preload=ar5iv.sty", \
  "--path=/opt/ar5iv-bindings/bindings", \
  "--path=/opt/ar5iv-bindings/supported_originals", \
  "--format=html5","--pmml","--mathtex", \
  "--timeout=2700", \
  "--noinvisibletimes", "--nodefaultresources", \
  "--css=https://cdn.jsdelivr.net/gh/dginev/ar5iv-css@0.8.4/css/ar5iv.min.css",\
  "--css=https://cdn.jsdelivr.net/gh/dginev/ar5iv-css@0.8.4/css/ar5iv-fonts.min.css"]

CMD ["--source=main.tex", "--dest=main.html"]
