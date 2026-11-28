## Dockerfile for ar5ivist-base: TeX Live + a released latexml-oxide
##
## build with
##
## $ docker build --tag ar5ivist-base:latest .
##
## (another release: --build-arg LATEXML_OXIDE_VERSION=X.Y.Z, with its .deb's SHA-256
##  checked against the release's own .sha256 file)
##
## run with the main ar5ivist Dockerfile:
##
## $ docker run -v "$(pwd)":/docdir -w /docdir \
##              --user "$(id -u):$(id -g)" \
##              ar5ivist:latest --source=main.tex --destination=html/main.html

FROM ubuntu:24.04

# The TeX Live toolchain first (TeX Live 2023 in Ubuntu 24.04), as it largely stays
# constant in the background. latexml-oxide reads the texmf tree through kpathsea at
# convert time, so the full arXiv-capable set matters for the long tail of packages
# and fonts. This is the same set and the same graphics tools as latexml-oxide's own
# images (its Dockerfile's `texbase` stage).
ENV TZ=Etc/UTC DEBIAN_FRONTEND=noninteractive
RUN set -ex && apt-get update -qq && apt-get install -qy --no-install-recommends \
  tzdata \
  ca-certificates \
  curl \
  texlive \
  texlive-fonts-extra \
  texlive-lang-all \
  texlive-latex-extra \
  texlive-bibtex-extra \
  texlive-science \
  texlive-pictures \
  texlive-pstricks \
  texlive-publishers \
  ghostscript \
  imagemagick \
  poppler-utils \
  mupdf-tools \
  dvipng \
  dvisvgm \
  && rm -rf /var/lib/apt/lists/*
# ghostscript is explicit (only a Recommends of imagemagick): `gs` rasterizes EPS/PS
# and is ImageMagick's PS/EPS/PDF delegate. poppler-utils gives pdftocairo and
# mupdf-tools `mutool draw`, latexml-oxide's PDF rasterizers.

# The image redistributes these (A)GPL tools unmodified: record, for each, the exact
# Ubuntu source package and where to get it (as latexml-oxide's own images do).
RUN set -e; d=/usr/local/share/doc/ar5ivist; mkdir -p "$d"; \
    dpkg-query -W -f='${Package} ${Version} ${source:Package} ${source:Version}\n' \
      ghostscript mupdf-tools poppler-utils > /tmp/srcpkgs; \
    { echo "Source of the GPL/AGPL programs redistributed unmodified in this image,"; \
      echo "as Ubuntu built them (binary package, version; source package, version, URL):"; \
      echo; \
      while read -r pkg ver src srcver; do \
        echo "$pkg $ver; source $src $srcver: https://launchpad.net/ubuntu/+source/$src/$srcver"; \
      done < /tmp/srcpkgs; } > "$d/THIRD-PARTY-SOURCES"; \
    [ "$(wc -l < /tmp/srcpkgs)" -eq 3 ]; rm -f /tmp/srcpkgs; cat "$d/THIRD-PARTY-SOURCES"

# latexml-oxide, from its GitHub release: the .deb for this architecture, checked
# against the release's SHA-256 file. The binary is self-contained (kernel dumps for
# TeX Live 2022-2026, the XSLT, the RelaxNG schema and ar5iv.css are embedded).
ARG LATEXML_OXIDE_VERSION=0.8.0
ARG TARGETARCH
RUN set -ex \
 && arch="${TARGETARCH:-$(dpkg --print-architecture)}" \
 && deb="latexml-oxide_${LATEXML_OXIDE_VERSION}-1_${arch}.deb" \
 && base="https://github.com/dginev/latexml-oxide/releases/download/${LATEXML_OXIDE_VERSION}" \
 && cd /tmp \
 && curl -fsSL -o "$deb" "$base/$deb" \
 && curl -fsSL -o "$deb.sha256" "$base/$deb.sha256" \
 && sha256sum -c "$deb.sha256" \
 && apt-get update -qq \
 && apt-get install -qy --no-install-recommends "./$deb" \
 && rm -f "$deb" "$deb.sha256" \
 && rm -rf /var/lib/apt/lists/* \
 && latexml_oxide --version

# Let ImageMagick read/write PDF/EPS and raise its resource ceilings: arXiv figures
# need it (Ubuntu's default policy.xml blocks PDF/EPS). The same patch as
# latexml-oxide's images.
RUN set -e; P=/etc/ImageMagick-6/policy.xml; if [ -f "$P" ]; then \
      sed -i -E 's/rights="none" pattern="([XE]?PS[0-9]?|PDF)"/rights="read|write" pattern="\1"/g' "$P"; \
      sed -i -E 's/(name="width" value=)"[^"]*"/\1"256KP"/'  "$P"; \
      sed -i -E 's/(name="height" value=)"[^"]*"/\1"256KP"/' "$P"; \
      sed -i -E 's/(name="area" value=)"[^"]*"/\1"4GiB"/'    "$P"; \
      sed -i -E 's/(name="disk" value=)"[^"]*"/\1"6GiB"/'    "$P"; \
      sed -i -E 's/(name="memory" value=)"[^"]*"/\1"4GiB"/'  "$P"; \
      sed -i -E 's/(name="map" value=)"[^"]*"/\1"4GiB"/'     "$P"; \
    fi

# Bake the TeX kernel formats for THIS image's TeX Live tree. The release binary embeds
# formats built from the TeX Live years it was released against, and uses one only
# for the tree it was built from; for any other tree it builds the format once into
# the user's ~/.cache, which a fresh container starts without (~10 s per run). Dumps
# in LATEXML_DUMP_DIR are found first, by every user, and only for the tree they were
# built from: an image derived from this one that changes the TeX tree must re-run
# the two --init lines below. LATEXML_FORMAT_CACHE is where a format for any other
# tree is built: the default, ~/.cache, cannot be created for a uid with no passwd
# entry (HOME=/), which would leave every run in the slow raw-load fallback.
ENV LATEXML_DUMP_DIR=/usr/local/share/latexml-oxide/resources/dumps \
    LATEXML_FORMAT_CACHE=/tmp/latexml-oxide-formats
RUN set -ex \
 && mkdir -p /usr/local/share/latexml-oxide/resources/dumps \
 && cd /usr/local/share/latexml-oxide \
 && latexml_oxide --init=plain.tex \
 && latexml_oxide --init=latex.ltx \
 && ls -la resources/dumps/*.dump.txt

# Build-time self-test: an ar5iv-style conversion must succeed, read the baked latex
# format, and write the bundled ar5iv.css beside the page, or the image is not built.
RUN set -ex \
 && mkdir -p /tmp/selftest && cd /tmp/selftest \
 && printf '\\documentclass{article}\n\\begin{document}\nHello from ar5ivist, $x^2$.\n\\end{document}\n' > main.tex \
 && latexml_oxide --preload=ar5iv.sty --format=html5 --pmml --mathtex --noinvisibletimes \
      --css=ar5iv.css --nodefaultresources --source=main.tex --dest=html/main.html \
 && grep -q 'Hello from ar5ivist' html/main.html \
 && grep -q '<math' html/main.html \
 && test -s html/ar5iv.css \
 && grep -q "from $LATEXML_DUMP_DIR/latex\." main.latexml.log \
 && cd / && rm -rf /tmp/selftest

ENV TMPDIR=/tmp
