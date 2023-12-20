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
  ghostscript \
  texlive-xetex \
  texlive-fonts-recommended \
  texlive-latex-recommended \
  texlive-fonts-extra \
  zip \
  unzip \
  git-lfs \ 
  fonts-freefont-otf \
  && \
  apt-get clean && rm -rf /var/lib/apt/lists/* 

# Install the pysonic repo list and key
# https://pysonic.cs.ox.ac.uk/trac/wiki/InstallGuides/UbuntuPackage
RUN echo "deb http://www.cs.ox.ac.uk/pysonic/ubuntu focal/" >> /etc/apt/sources.list.d/pysonic.list && \
  apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 422C4D99

RUN pip --no-cache --quiet install --upgrade \
  pip \
  setuptools \
  wheel



# Fix CMake warnings: https://github.com/autowarefoundation/autoware/issues/795
RUN update-alternatives --install /usr/bin/vtk vtk /usr/bin/vtk7 7

# Install TextTest for regression testing (this requires pygtk)
RUN pip install texttest
ENV TEXTTEST_HOME /usr/local/bin/texttest


# Allow pysonic_DIR to be set at build time if desired
ARG pysonic_DIR="/home/jovyan/pysonic"
ENV pysonic_DIR=${pysonic_DIR}


# Add scripts
COPY --chown=$NB_UID:$NB_UID kernels/pysonic/scripts "${pysonic_DIR}/scripts"
# WORKDIR ${pysonic_DIR}
ENV PATH "${pysonic_DIR}/scripts:${PATH}"

# Set environment variables
# RUN source /home/pysonic/scripts/set_env_vars.sh
ENV pysonic_SOURCE_DIR="${pysonic_DIR}/src" \
    pysonic_BUILD_DIR="${pysonic_DIR}/lib" \
    pysonic_PROJECTS_DIR="${pysonic_DIR}/src/projects" \
    pysonic_TEST_OUTPUT="${pysonic_DIR}/testoutput"

# CMake environment variables
ARG CMAKE_BUILD_TYPE="Release"
ARG pysonic_ERROR_ON_WARNING="OFF"
ARG pysonic_UPDATE_PROVENANCE="OFF"
ENV CMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} \
    pysonic_ERROR_ON_WARNING=${pysonic_ERROR_ON_WARNING} \
    pysonic_UPDATE_PROVENANCE=${pysonic_UPDATE_PROVENANCE}

USER $NB_UID

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

RUN .venv/bin/pip --no-cache install pip-tools && \
  .venv/bin/pip --no-cache install -r ${NOTEBOOK_BASE_DIR}/requirements.txt && \
  && cp -f /logos/logo-* .local/share/jupyter/kernels/pysonic

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

RUN mkdir --parents "/home/${NB_USER}/.virtual_documents" && \
  chown --recursive "$NB_USER" "/home/${NB_USER}/.virtual_documents"
ENV JP_LSP_VIRTUAL_DIR="/home/${NB_USER}/.virtual_documents"

# Copying boot scripts
COPY --chown=$NB_UID:$NB_GID docker /docker

#RUN echo 'export PATH="/home/${NB_USER}/.venv/bin:$PATH"' >> "/home/${NB_USER}/.bashrc"

EXPOSE 8888

ENTRYPOINT [ "/bin/bash", "/docker/entrypoint.bash" ]