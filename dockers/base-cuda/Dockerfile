# Copyright The PyTorch Lightning team.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Existing images:
# --build-arg PYTHON_VERSION=3.7 --build-arg PYTORCH_VERSION=1.7 --build-arg CUDA_VERSION=10.2
# --build-arg PYTHON_VERSION=3.7 --build-arg PYTORCH_VERSION=1.6 --build-arg CUDA_VERSION=10.2
# --build-arg PYTHON_VERSION=3.7 --build-arg PYTORCH_VERSION=1.5 --build-arg CUDA_VERSION=10.2
# --build-arg PYTHON_VERSION=3.7 --build-arg PYTORCH_VERSION=1.4 --build-arg CUDA_VERSION=10.1
# --build-arg PYTHON_VERSION=3.7 --build-arg PYTORCH_VERSION=1.3 --build-arg CUDA_VERSION=10.1

ARG CUDNN_VERSION=8
ARG CUDA_VERSION=10.2

# FROM nvidia/cuda:${CUDA_VERSION}-devel-ubuntu20.04
FROM nvidia/cuda:${CUDA_VERSION}-cudnn${CUDNN_VERSION}-devel-ubuntu18.04
# FROM nvidia/cuda:${CUDA_VERSION}-devel-ubuntu18.04

ARG PYTHON_VERSION=3.7
ARG PYTORCH_VERSION=1.6

SHELL ["/bin/bash", "-c"]
# https://techoverflow.net/2019/05/18/how-to-fix-configuring-tzdata-interactive-input-when-building-docker-images/
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Europe/Prague

ENV PATH="$PATH:/root/.local/bin"
ENV CUDA_TOOLKIT_ROOT_DIR="/usr/local/cuda"

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        pkg-config \
        cmake \
        git \
        wget \
        ca-certificates \
        software-properties-common \
    && \

# Install python
    add-apt-repository ppa:deadsnakes/ppa && \
    apt-get install -y \
        python${PYTHON_VERSION} \
        python${PYTHON_VERSION}-distutils \
        python${PYTHON_VERSION}-dev \
    && \

    update-alternatives --install /usr/bin/python${PYTHON_VERSION%%.*} python${PYTHON_VERSION%%.*} /usr/bin/python${PYTHON_VERSION} 1 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python${PYTHON_VERSION} 1 && \

# Cleaning
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /root/.cache && \
    rm -rf /var/lib/apt/lists/*

ENV HOROVOD_GPU_OPERATIONS=NCCL
ENV HOROVOD_WITH_PYTORCH=1
ENV HOROVOD_WITHOUT_TENSORFLOW=1
ENV HOROVOD_WITHOUT_MXNET=1
ENV HOROVOD_WITH_GLOO=1
ENV HOROVOD_WITHOUT_MPI=1
#ENV MAKEFLAGS="-j$(nproc)"
ENV MAKEFLAGS="-j1"
ENV TORCH_CUDA_ARCH_LIST="3.7;5.0;6.0;7.0;7.5"

COPY ./requirements.txt requirements.txt
COPY ./requirements/ ./requirements/

# conda init
RUN \
    wget https://bootstrap.pypa.io/get-pip.py --progress=bar:force:noscroll --no-check-certificate && \
    python${PYTHON_VERSION} get-pip.py && \
    rm get-pip.py && \

    # Disable cache
    pip config set global.cache-dir false && \
    # eventualy use pre-release
    #pip install "torch==${PYTORCH_VERSION}.*" --pre && \
    # set particular PyTorch version
    python -c "import re ; fname = 'requirements.txt' ; req = re.sub(r'torch[>=]+[\d\.]+', 'torch==${PYTORCH_VERSION}.*', open(fname).read()) ; open(fname, 'w').write(req)" && \

    # Install all requirements
    pip install -r requirements/devel.txt --upgrade-strategy only-if-needed --use-feature=2020-resolver && \
    rm -rf requirements*

RUN \
    # install NVIDIA AMP
    git clone https://github.com/NVIDIA/apex && \
    pip install --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./apex && \
    rm -rf apex

RUN \
    # Show what we have
    pip --version && \
    pip list && \
    python -c "import sys; assert sys.version[:3] == '$PYTHON_VERSION', sys.version" && \
    python -c "import torch; assert torch.__version__[:3] == '$PYTORCH_VERSION', torch.__version__"