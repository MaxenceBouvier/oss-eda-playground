
FROM nvidia/cuda:12.6.1-base-ubuntu24.04 AS base

# Non-interactive apt
ENV DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

RUN apt-get -y update && apt-get install -y --no-install-recommends \
    locales ca-certificates curl git git-lfs \
    build-essential pkg-config \
    libgl1 libglib2.0-0 \
    less \
    && rm -rf /var/lib/apt/lists/* \
 && locale-gen en_US.UTF-8

RUN apt-get -y update 
RUN apt-get -y install software-properties-common
RUN apt-get -y install --no-install-recommends apt-utils 
RUN add-apt-repository -y ppa:deadsnakes/ppa 
RUN apt-get -y install wget tk-dev libgdbm-dev zlib1g-dev libffi-dev libssl-dev libbz2-dev libreadline-dev libsqlite3-dev liblzma-dev libncurses5-dev libtk8.6 libgdm-dev libpcap-dev git help2man perl make autoconf g++ flex bison ccache libgoogle-perftools-dev numactl perl-doc libfl2   libfl-dev  zlib1g zlib1g-dev gperf meson gperf desktop-file-utils libgtk-3-dev libjudy-dev libgirepository1.0-dev bison flex libreadline-dev gawk tcl-dev libffi-dev git graphviz xdot pkg-config         libboost-system-dev libboost-python-dev libboost-filesystem-dev zlib1g-dev software-properties-common help2man perl make autoconf g++ flex bison ccache libgoogle-perftools-dev numactl perl-doc libfl2  libfl-dev zlib1g zlib1g-dev python3.11 qtbase5-dev qttools5-dev libqt5xmlpatterns5-dev qtmultimedia5-dev libqt5multimediawidgets5 libqt5svg5-dev ruby ruby-dev libz-dev libgit2-dev zip cmake swig
RUN apt-get -y install python3 python3-dev python3-venv python3-pip

ARG NPROC

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# FROM base AS tech_lib
# WORKDIR /app
# RUN git clone https://github.com/The-OpenROAD-Project/asap7sc7p5t_28.git /app/asap7sc7p5t_28
# ENV MODEL_SOURCES=/app/asap7sc7p5t_28/Verilog
# ENV LIB_DIR=/prog/OpenROAD-flow-scripts/flow/platforms/asap7/lib/NLDM/

FROM base AS user_setup

# Create a non-root user matching the devcontainer default
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
# Create group if needed; create user if needed; ensure /workspace ownership
RUN set -eux; \
    # Determine group name to use for the target GID
    EXISTING_GROUP_NAME="$(getent group "${USER_GID}" | cut -d: -f1 || true)"; \
    GROUP_NAME="${EXISTING_GROUP_NAME:-$USERNAME}"; \
    # Ensure group exists with the desired GID
    if [ -z "${EXISTING_GROUP_NAME}" ]; then \
        groupadd --gid "${USER_GID}" "${GROUP_NAME}"; \
    fi; \
    # Create the user with fixed name USERNAME.
    # If the UID is taken, either modify USER_UID or delete the existing user (choose policy).
    if id -u "${USERNAME}" >/dev/null 2>&1; then \
        echo "User ${USERNAME} already exists"; \
    else \
        if getent passwd "${USER_UID}" >/dev/null; then \
            echo "UID ${USER_UID} already exists; creating ${USERNAME} without specifying UID"; \
            useradd -s /bin/bash --gid "${USER_GID}" -m "${USERNAME}"; \
        else \
            useradd -s /bin/bash --uid "${USER_UID}" --gid "${USER_GID}" -m "${USERNAME}"; \
        fi; \
    fi; \
    # Ensure primary group of USERNAME is GROUP_NAME (if names differ)
    usermod -g "${GROUP_NAME}" "${USERNAME}"; \
    mkdir -p /workspace; \
    chown -R "$(id -u "${USERNAME}")":"${USER_GID}" /workspace

FROM user_setup AS verilator
RUN git clone https://github.com/verilator/verilator /prog/verilator
WORKDIR /prog/verilator
RUN git checkout -b build_version tags/v5.036 && autoconf
RUN ./configure --prefix /prog/verilator
RUN make -j
ENV PATH=$PATH:/prog/verilator/bin

FROM verilator AS yosys
RUN git clone https://github.com/YosysHQ/yosys.git /prog/yosys
WORKDIR /prog/yosys
RUN git checkout -b build_version tags/v0.54 && git submodule update --init
RUN make config-gcc && make -j `nproc` && make install
RUN git clone --recursive https://github.com/povik/yosys-slang 
WORKDIR /prog/yosys/yosys-slang
RUN make -j `nproc` && make install
ENV PATH=$PATH:/prog/yosys/bin

# FROM yosys AS orfs
# RUN git clone https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts /prog/OpenROAD-flow-scripts
# WORKDIR /prog/OpenROAD-flow-scripts
# RUN sed -i 's/sudo -u $SUDO_USER//g' setup.sh && ./setup.sh && ./build_openroad.sh --local
# ENV PATH=$PATH:/prog/OpenROAD-flow-scripts/tools/install/OpenROAD/bin/
# ENV PATH=$PATH:/prog/verilator/bin

# FROM orfs AS cudd
# RUN git clone https://github.com/davidkebo/cudd /prog/cudd
# WORKDIR /prog/cudd/cudd_versions
# RUN tar xvfz cudd-3.0.0.tar.gz
# WORKDIR /prog/cudd/cudd_versions/cudd-3.0.0
# RUN ./configure --prefix  /prog/cudd/cudd_versions/cudd-3.0.0
# RUN make -j $NPROC install
# ENV CUDD_INSTALL_DIR=/prog/cudd/cudd_versions/cudd-3.0.0

# FROM cudd AS opensta
# RUN git clone https://gitlab.com/libeigen/eigen.git /prog/eigen && mkdir /prog/eigen/build_dir
# WORKDIR /prog/eigen/build_dir
# RUN cmake .. && make install
# ENV PATH=$PATH:/prog/OpenSTA/app

# RUN git clone https://github.com/The-OpenROAD-Project/OpenSTA /prog/OpenSTA
# WORKDIR /prog/OpenSTA
# RUN mkdir build 
# WORKDIR /prog/OpenSTA/build
# RUN cmake -DCUDD_DIR=$CUDD_INSTALL_DIR ../
# RUN make -j $NPROC

# FROM orfs AS rust
# RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --no-modify-path
# ENV PATH="/root/.cargo/bin:${PATH}"
# RUN git clone https://github.com/antmicro/trace2power.git /prog/trace2power
# WORKDIR /prog/trace2power
# RUN git checkout 74949-glitch-power && cargo install --path .

FROM yosys AS env_setup


# Enable Git LFS for large model files
RUN git lfs install --system

# Cache directory env vars for HF
ENV HF_HOME=/home/$USERNAME/.cache/huggingface
ENV TRANSFORMERS_CACHE=$HF_HOME/transformers
ENV HF_HUB_ENABLE_HF_TRANSFER=1

# Make CUDA visible inside container; devcontainer.json should still pass --gpus=all
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility

USER $USERNAME

FROM env_setup AS python_313
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH=$PATH:/home/$USERNAME/.local/bin
WORKDIR /home/$USERNAME
RUN uv venv -n .pyenv/313_env --python=3.13
ENV PATH=/home/$USERNAME/.pyenv/313_env/bin:$PATH

# Upgrade pip/setuptools/wheels
RUN uv pip install --no-cache-dir --upgrade pip setuptools wheel
RUN uv pip install --no-cache-dir torch torchvision
RUN uv pip install cocotb numpy pandas pyarrow pyyaml tqdm lightning
RUN uv pip install transformers[torch] \
    datasets \
    accelerate \
    sentencepiece \
    huggingface_hub[cli] \
    safetensors \
    einops\
    pre-commit
    # uvicorn fastapi \
    # bitsandbytes \
    # peft \
    # triton \
    # xformers \
    # numpy scipy pandas tqdm \
    # jupyterlab ipykernel \

FROM python_313 AS app

WORKDIR /app