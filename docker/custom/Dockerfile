ARG JUPYTER_MINIMAL_VERSION=lab-3.3.2@sha256:a4bf48221bfa864759e0f248affec3df1af0a68ee3e43dfc7435d84926ec92e8
FROM jupyter/minimal-notebook:${JUPYTER_MINIMAL_VERSION}

LABEL maintainer="elisabettai" 

ENV JUPYTER_ENABLE_LAB="yes"
# autentication is disabled for now
ENV NOTEBOOK_TOKEN=""
ENV NOTEBOOK_BASE_DIR="$HOME/work"

USER root

RUN apt-get update && \
  apt-get install -y --no-install-recommends \
  ffmpeg \
  make \
  gnupg \
  dvipng \
  gosu \
  bc \
  gcc \
  g++ \
  ghostscript \
  texlive-xetex \
  texlive-fonts-recommended \
  texlive-latex-recommended \
  texlive-fonts-extra \
  zip \
  unzip \
  git-lfs \ 
  fonts-freefont-otf \
  libx11-dev \ 
  bison \
  flex \
  automake \
  pkg-config \
  libfreetype-dev \
  libpng-dev \
  libtool \ 
  libxext-dev \
  libncurses-dev \
  xfonts-100dpi \
  cython3 \
  libopenmpi-dev \
  make \
  zlib1g-dev \
  && \
  apt-get clean && rm -rf /var/lib/apt/lists/* 

# --------------------------------------------------------------------

# Install Python kernel in virtual-env
ENV HOME="/home/$NB_USER"

USER root

WORKDIR ${HOME}

RUN python3 -m venv .venv &&\
  .venv/bin/pip --no-cache --quiet install --upgrade pip wheel setuptools &&\
  .venv/bin/pip --no-cache --quiet install ipykernel &&\
  .venv/bin/python -m ipykernel install \
  --user \
  --name "PySONIC" \
  --display-name "PySONIC" \
  && \
  jupyter kernelspec list


# copy and resolve dependecies to be up to date
COPY --chown=$NB_UID:$NB_GID kernels/pysonic/requirements.txt ${NOTEBOOK_BASE_DIR}/requirements.txt
COPY --chown=$NB_UID:$NB_GID kernels/pysonic/logo-* /logos/

# Install requirements, remove default python logo and add custom logos
RUN .venv/bin/pip --no-cache install pip-tools && \
  .venv/bin/pip --no-cache install -r ${NOTEBOOK_BASE_DIR}/requirements.txt && \
  cp -f /logos/logo-* ${HOME}/.local/share/jupyter/kernels/pysonic && \
  rm ${HOME}/.local/share/jupyter/kernels/pysonic/logo-svg.svg

RUN jupyter serverextension enable voila && \
  jupyter server extension enable voila

# Import matplotlib the first time to build the font cache.
ENV XDG_CACHE_HOME /home/$NB_USER/.cache/
RUN MPLBACKEND=Agg .venv/bin/python -c "import matplotlib.pyplot" 

RUN fix-permissions /home/$NB_USER

# copy README and CHANGELOG
COPY --chown=$NB_UID:$NB_GID CHANGELOG.md ${NOTEBOOK_BASE_DIR}/CHANGELOG.md
COPY --chown=$NB_UID:$NB_GID README.ipynb ${NOTEBOOK_BASE_DIR}/README.ipynb
# remove write permissions from files which are not supposed to be edited
RUN chmod gu-w ${NOTEBOOK_BASE_DIR}/CHANGELOG.md && \
  chmod gu-w ${NOTEBOOK_BASE_DIR}/requirements.txt

# Compile the MOD files
RUN source .venv/bin/activate && \
    cd /home/jovyan/.venv/src/morphosonic/MorphoSONIC/nmodl && \
    nrnivmodl

RUN mkdir --parents "/home/${NB_USER}/.virtual_documents" && \
  chown --recursive "$NB_USER" "/home/${NB_USER}/.virtual_documents"
ENV JP_LSP_VIRTUAL_DIR="/home/${NB_USER}/.virtual_documents"

# TODO, move up
RUN cp -f /logos/logo-* ${HOME}/.local/share/jupyter/kernels/pysonic

# Copying boot scripts
COPY --chown=$NB_UID:$NB_GID docker /docker

RUN echo 'export PATH="/home/${NB_USER}/.venv/bin:$PATH"' >> "/home/${NB_USER}/.bashrc"

EXPOSE 8888

ENTRYPOINT [ "/bin/bash", "/docker/entrypoint.bash" ]