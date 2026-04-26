# Base container: CUDA 12.21, cuDNN 8.9.4, Python 3.10, PyTorch 2.1.0
# ARG BASE_IMG=nvcr.io/nvidia/pytorch:23.08-py3

ARG BASE_IMG=nvcr.io/nvidia/pytorch:24.12-py3
# ARG BASE_IMG=nvcr.io/nvidia/pytorch:26.03-py3

FROM ${BASE_IMG}

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    # For development and debug (procps: `free -h`)
    curl git unzip wget net-tools vim procps \
    # For OpenGL dependencies
    mesa-common-dev libegl1-mesa-dev libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev \
    && rm -rf /var/lib/apt/lists/* \
    && apt clean && apt -y autoremove


# ====================================================================
# User Setup
# ====================================================================
# arguments when building image
ARG UID
ARG GID
ARG MACHINE_USER

# Ensure workspace paths are writable by runtime user before switching USER.
RUN mkdir -p /workspace /workspace/opt && \
    chown -R ${UID}:${GID} /workspace

# In case that the user is "ubuntu", you can add neither group nor user, since it already exists in the base image.
# In that case, just set the password for the existing user and add it to sudo group.
# --- For not ubuntu user ---
# RUN groupadd -g $id $user
# RUN useradd -s /bin/bash -u $id -g $id -m $user && echo "$user:$password" | chpasswd && adduser $user sudo
# --- For ubuntu user ---

# sudo 実行時にパスワード不要にする（entrypoint.sh での service 起動に必要）
RUN echo "$MACHINE_USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
USER $MACHINE_USER


# ====================================================================
# Python Environment Setup (uv & venv)
# ====================================================================
# Reference: https://docs.astral.sh/uv/guides/integration/docker/#installing-uv
# uv is used for efficient package management (faster than pip)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

# uv will automatically download Python 3.12 if not present in the base image
RUN uv venv /workspace/.venv --python 3.12
ENV PATH="/workspace/.venv/bin:$PATH"
ENV UV_PROJECT_ENVIRONMENT=/workspace/.venv

WORKDIR /workspace


# ====================================================================
# Install Bayes3d dependencies from pyproject optional group `docker`
# ====================================================================
COPY --chown=${UID}:${GID} ./pyproject.toml /workspace/opt/pyproject.toml
COPY --chown=${UID}:${GID} ./README.md /workspace/opt/README.md
# /workspace/opt does not include .git metadata during this step, so
# setuptools-scm needs an explicit fallback version to evaluate project metadata.
ENV SETUPTOOLS_SCM_PRETEND_VERSION_FOR_BAYES3D=0.0.0
RUN uv sync --project /workspace/opt --extra docker --no-dev --no-install-project

# Workaround: nvidia.cuda_nvcc is a namespace package on the base image with __file__ == None,
# which causes a crash in jax 0.4.35 at import time. A stub __init__.py fixes this.
RUN SITE=$(python -c "import site; print(site.getsitepackages()[0])") && \
    mkdir -p "$SITE/nvidia/cuda_nvcc" && \
    echo "# stub to prevent __file__ == None crash in some jax environments" > "$SITE/nvidia/cuda_nvcc/__init__.py"

# To not overwrite torch, exclude timm from requirement.txt as it depends on torch.
RUN uv pip install --no-deps timm


# ====================================================================
# Install genjax (local install)
# ====================================================================
# COPY ./docker/genjax  /workspace/genjax
# RUN uv pip install --system -r /workspace/genjax/requirements.txt
# RUN uv pip install --system /workspace/genjax

# # Install JAX (0.4.16)
# RUN uv pip install --upgrade "jax[cuda12_pip]" -f https://storage.googleapis.com/jax-releases/jax_cuda_releases.html


# ====================================================================
# Cleanup and prepare env variables for graphics
# ====================================================================
# /workspace/genjax

ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility,display

# # Skip building CUDA/OpenGL native extensions when CUDA is not available
# ENV BAYES3D_SKIP_EXTENSIONS=1
