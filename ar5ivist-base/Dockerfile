## Dockerfile for ar5ivist-base, using a fixed LaTeXML commit
##
## build with
##
## $ docker build --tag ar5ivist-base:latest .
##
## run with the main ar5ivist Dockerfile:
##
## $ docker run -v "$(pwd)":/docdir -w /docdir \
##              --user "$(id -u):$(id -g)" \
##              ar5ivist:latest --source=main.tex --destination=html/main.html

FROM ubuntu:24.04

# Install the texlive toolchain first (v2021 in Ubuntu 22.04)
# as it largely stays constant in the background.
ENV DEBIAN_FRONTEND=noninteractive
RUN set -ex && apt-get update -qq && apt-get install -qy --no-install-recommends tzdata
RUN set -ex && apt-get update -qq && apt-get install -qy \
  texlive \
  texlive-fonts-extra \
  texlive-lang-all \
  texlive-latex-extra \
  texlive-bibtex-extra \
  texlive-science \
  texlive-pictures \
  texlive-pstricks \
  texlive-publishers

# latexml dependencies
RUN set -ex && apt-get update -qq && apt-get install -qy \
  build-essential \
  git \
  imagemagick \
  libarchive-zip-perl \
  libdb-dev \
  libfile-which-perl \
  libimage-magick-perl \
  libimage-size-perl \
  libio-string-perl \
  libjson-xs-perl \
  libparse-recdescent-perl \
  libtext-unidecode-perl \
  liburi-perl \
  libuuid-tiny-perl \
  libwww-perl \
  libxml-libxml-perl \
  libxml-libxslt-perl \
  libxml2-dev \
  libxslt1-dev \
  liblocal-lib-perl \
  make \
  perl-doc \
  cpanminus

# make sure perl paths are OK
RUN eval $(perl -I$HOME/perl5/lib -Mlocal::lib)
RUN echo 'eval "$(perl -I$HOME/perl5/lib/perl5 -Mlocal::lib)"' >> ~/.bashrc

# Collect the extended ar5iv-bindings files
ENV AR5IV_BINDINGS_COMMIT=940ef38c88c813196972e9abd9acd265bf44d98e
RUN rm -rf /opt/ar5iv-bindings
RUN git clone https://github.com/dginev/ar5iv-bindings /opt/ar5iv-bindings
WORKDIR /opt/ar5iv-bindings
RUN git reset --hard $AR5IV_BINDINGS_COMMIT

# Install LaTeXML, at a fixed commit, via cpanminus
RUN mkdir -p /opt/latexml
WORKDIR /opt/latexml
ENV LATEXML_COMMIT=25ec2b0e9070cc05cbb5e5e22bebf5ba98a0d86c
RUN cpanm --notest --verbose --build-args formats https://github.com/brucemiller/LaTeXML/archive/${LATEXML_COMMIT}.zip

# Enable imagemagick policy permissions for work with arXiv PDF/EPS files
RUN perl -pi.bak -e 's/rights="none" pattern="([XE]?PS\d?|PDF)"/rights="read|write" pattern="$1"/g' /etc/ImageMagick-6/policy.xml
# Extend imagemagick resource allowance to be able to create with high-quality images
RUN perl -pi.bak -e 's/policy domain="resource" name="width" value="(\w+)"/policy domain="resource" name="width" value="126KP"/' /etc/ImageMagick-6/policy.xml
RUN perl -pi.bak -e 's/policy domain="resource" name="height" value="(\w+)"/policy domain="resource" name="height" value="126KP"/' /etc/ImageMagick-6/policy.xml
RUN perl -pi.bak -e 's/policy domain="resource" name="area" value="(\w+)"/policy domain="resource" name="area" value="2GiB"/' /etc/ImageMagick-6/policy.xml
RUN perl -pi.bak -e 's/policy domain="resource" name="disk" value="(\w+)"/policy domain="resource" name="disk" value="8GiB"/' /etc/ImageMagick-6/policy.xml
RUN perl -pi.bak -e 's/policy domain="resource" name="memory" value="(\w+)"/policy domain="resource" name="memory" value="2GiB"/' /etc/ImageMagick-6/policy.xml
RUN perl -pi.bak -e 's/policy domain="resource" name="map" value="(\w+)"/policy domain="resource" name="map" value="2GiB"/' /etc/ImageMagick-6/policy.xml

ENV MAGICK_DISK_LIMIT=2GiB
ENV MAGICK_MEMORY_LIMIT=512MiB
ENV MAGICK_MAP_LIMIT=1GiB
ENV MAGICK_TIME_LIMIT=900
ENV MAGICK_TMPDIR=/tmp
ENV TMPDIR=/tmp
