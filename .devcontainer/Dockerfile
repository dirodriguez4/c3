ARG VARIANT="3.9"
# It is necessary for libraries that are not supported for M1/ARM Machines
#FROM arm64v8/python:${VARIANT}-slim
#FROM python:${VARIANT}-slim
FROM mcr.microsoft.com/vscode/devcontainers/python:0-${VARIANT}
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# Most of the below is taken from https://github.com/microsoft/vscode-dev-containers/tree/master/containers/python-3-miniconda/.devcontainer
# [Option] Install zsh
ARG INSTALL_ZSH="true"
# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="true"

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

COPY ./.devcontainer/library-scripts/common-debian.sh /tmp/library-scripts/

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    # Remove imagemagick due to https://security-tracker.debian.org/tracker/CVE-2019-10131
    && apt-get purge -y imagemagick imagemagick-6-common 
    # Install common packages, non-root user
RUN bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts

#common_debian.sh does a great job installing zsh and oh-my-zsh with a custom theme but it does not have the plugins to enhance its usability
# Add zsh plugins 
ARG USER_RC_PATH="/home/${USERNAME}"
ARG ZSH_CUSTOM="${USER_RC_PATH}/.oh-my-zsh/custom"
RUN git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM}/plugins/zsh-autosuggestions
RUN git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM}/plugins/zsh-syntax-highlighting
RUN git clone https://github.com/supercrabtree/k ${ZSH_CUSTOM}/plugins/k
RUN git clone --depth 1 https://github.com/junegunn/fzf.git ${USER_RC_PATH}/.fzf \
    && ${USER_RC_PATH}/.fzf/install
# Add to profile .zshrc
COPY ./.devcontainer/library-scripts/zsh.profile ${USER_RC_PATH}/.zshrc

# ENV PIP_TARGET=/usr/local/pip-global
# ENV PYTHONPATH=${PIP_TARGET}:${PYTHONPATH}
# ENV PATH=${PIP_TARGET}/bin:${PATH}
# RUN if ! cat /etc/group | grep -e "^pip-global:" > /dev/null 2>&1; then groupadd -r pip-global; fi \
#     && usermod -a -G pip-global vscode \
#     && umask 0002 && mkdir -p ${PIP_TARGET} \
#     && chown :pip-global ${PIP_TARGET} \
#     && ( [ ! -f "/etc/profile.d/00-restore-env.sh" ] || sed -i -e "s/export PATH=/export PATH=\/usr\/local\/pip-global:/" /etc/profile.d/00-restore-env.sh )

# Setup default python tools in a venv via pipx to avoid conflicts
ARG DEFAULT_UTILS="\
    pylint \
    flake8 \
    autopep8 \
    black \
    yapf \
    mypy \
    pydocstyle \
    pycodestyle \
    bandit \
    isort \
    virtualenv \
    poetry \
    pydeps "
ENV PIPX_HOME=/usr/local/py-utils
ENV PIPX_BIN_DIR=${PIPX_HOME}/bin
ENV PATH=${PATH}:${PIPX_BIN_DIR}
RUN mkdir -p ${PIPX_BIN_DIR} \
    && export PYTHONUSERBASE=/tmp/pip-tmp \
    && pip3 install --disable-pip-version-check --no-warn-script-location --no-cache-dir --user pipx \
    && /tmp/pip-tmp/bin/pipx install --pip-args=--no-cache-dir pipx \
    && echo "${DEFAULT_UTILS}" | xargs -n 1 /tmp/pip-tmp/bin/pipx install --system-site-packages --pip-args=--no-cache-dir --pip-args=--force-reinstall \
    && chown -R ${USER_UID}:${USER_GID} ${PIPX_HOME} \
    && rm -rf /tmp/pip-tmp


# Installing IDE Dev python packages
RUN pip3 --disable-pip-version-check --no-cache-dir install tqdm ipython ipykernel jupyter notebook pytest mkdocs-material "mkdocstrings[python]" "mkdocs-exclude" "mkdocs-ezlinks-plugin"


## [Optional] Uncomment this section to install additional OS packages.
RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    build-essential 

# [Optional] If your pip requirements rarely change, uncomment this section to add them to the image.
ARG PYPACKAGE="requirements.txt"
COPY ./.devcontainer/${PYPACKAGE} /tmp/pip-tmp/
RUN pip3 --disable-pip-version-check --no-cache-dir install -r /tmp/pip-tmp/${PYPACKAGE} \
   && rm -rf /tmp/pip-tmp







