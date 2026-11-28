## Dockerfile for ar5ivist, on a released latexml-oxide (see ../ar5ivist-base)
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

# A `<package>.sty.rhai` beside a source is a runtime binding latexml-oxide loads; for
# arXiv sources, which are other people's, it may not run programs (README "Trust").
ENV LATEXML_DISABLE_SHELL_ESCAPE=1

# The ar5iv profile, as arXiv's ar5iv conversions run it: ar5iv.sty preloaded (it
# loads latexml.sty and the ar5iv configuration), HTML5 with presentation MathML and
# the TeX of each formula, no invisible-times operators, and the stylesheet bundled
# in the release binary (ar5iv.css, written beside the page). A generous timeout for
# local use (45 minutes). The second stylesheet is ar5iv's webfonts, linked from the
# CDN as the Perl-based ar5ivist did; keep its version the ar5iv-css release the
# latexml-oxide release bundles (0.9.6 for 0.8.0).
# continue as instructed in https://www.howtogeek.com/devops/how-to-use-docker-to-package-cli-applications/
# tini as PID 1: see ar5ivist-base (figure conversion fails when latexml_oxide is PID 1).
ENTRYPOINT ["/usr/bin/tini", "-s", "--", "latexml_oxide", \
  "--preload=ar5iv.sty", \
  "--format=html5", "--pmml", "--mathtex", \
  "--timeout=2700", \
  "--noinvisibletimes", \
  "--css=ar5iv.css", "--nodefaultresources", \
  "--css=https://cdn.jsdelivr.net/gh/dginev/ar5iv-css@0.9.6/css/ar5iv-fonts.min.css"]

CMD ["--source=main.tex", "--dest=main.html"]
