# This image includes a minimal Emacs built from source, Cask, and make.  It is
# used to run Flycheck's unit tests.

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Build Emacs

# We need 16.04 as Emacs 24.4 and 24.5 won't build with 18.04.
# But they work fine with a 18.04 runtime.
FROM ubuntu:16.04 AS emacs-builder

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update >/dev/null && \
    apt-get install --no-install-recommends -y \
            autoconf \
            automake \
            build-essential \
            ca-certificates \
            curl \
            libncurses-dev \
            unzip \
            xz-utils \
            >/dev/null \
    && rm -rf /var/lib/apt/lists/*

ARG EMACS_VERSION
ENV EMACS_VERSION ${EMACS_VERSION:-25.3}
RUN case ${EMACS_VERSION} in \
      master) \
        curl -sSLo emacs.zip "https://github.com/emacs-mirror/emacs/archive/master.zip" \
        && unzip emacs.zip >/dev/null \
        && rm emacs.zip \
        ;; \
      *-rc*) \
        curl -sSLo emacs.tz "https://alpha.gnu.org/pub/gnu/emacs/pretest/emacs-${EMACS_VERSION}.tar.xz" \
        && mkdir emacs-master \
        && tar -C emacs-master --strip-components=1 -xf emacs.tz \
        && rm emacs.tz \
        ;; \
      *) \
        curl -sSLo emacs.tz "https://ftpmirror.gnu.org/emacs/emacs-${EMACS_VERSION}.tar.xz" \
        && mkdir emacs-master \
        && tar -C emacs-master --strip-components=1 -xf emacs.tz \
        && rm emacs.tz \
        ;; \
    esac \
    && cd emacs-master \
    && ./autogen.sh >/dev/null \
    && ./configure --prefix=/opt/emacs \
                   --with-x-toolkit=no --without-x --without-all --with-xml2 \
                   CFLAGS='-O2 -march=native' CXXFLAGS='-O2 -march=native' \
                   >/dev/null \
    && make -j8 install >/dev/null

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Get and bootstrap cask

FROM ubuntu:18.04 AS cask-builder

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update >/dev/null && \
    apt-get install --no-install-recommends -y \
            ca-certificates \
            curl \
            git \
            gnutls-bin \
            python \
            >/dev/null \
    && rm -rf /var/lib/apt/lists/*

ENV PATH /opt/emacs/bin:$PATH
COPY --from=emacs-builder /opt/emacs /opt/emacs

RUN curl -fsSL https://raw.githubusercontent.com/cask/cask/master/go | python

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Final images, without developement dependencies
FROM ubuntu:18.04

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update >/dev/null && \
    apt-get install --no-install-recommends -y \
            ca-certificates \
            gnutls-bin \
            make \
            python \
            >/dev/null \
    && rm -rf /var/lib/apt/lists/*

ENV PATH /opt/emacs/bin:$PATH
COPY --from=emacs-builder /opt/emacs /opt/emacs

ENV PATH /root/.cask/bin:$PATH
COPY --from=cask-builder /root/.cask /root/.cask
RUN cask init

# Check that they were correctly built
RUN emacs --version && cask --version

# To grab Emacs, use:
# ENV PATH /opt/emacs/bin:$PATH
# COPY --from=emacs25 /opt/emacs /opt/emacs

# To grab Cask:
# ENV PATH /root/.cask/bin:$PATH
# COPY --from=emacs25 /root/.cask /root/.cask

# To reuse the `cask init` call:
# COPY --from=emacs25 /root/.emacs.d /root/.emacs.d
