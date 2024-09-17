################################################################################
# Dockerfile that builds 'yanwk/comfyui-boot:cu124-megapak'
# A big all-in-one package with many custom nodes.
# Using CUDA 12.4, Python 3.12, GCC 13.
################################################################################

FROM nvidia/cuda:12.1.0-base-ubuntu22.04

################################################################################
# Set environment variables to avoid interactive prompts
################################################################################

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

################################################################################
# Install essential system tools, video libraries, OpenGL, and add Python 3.12
################################################################################

# Install dependencies for add-apt-repository
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    software-properties-common \
    python3-apt \
    curl \
    ca-certificates

# Add deadsnakes PPA for Python 3.12
RUN add-apt-repository ppa:deadsnakes/ppa && apt-get update

# Install Python 3.12
RUN apt-get install -y python3.12 python3.12-distutils

# Set up Python alternatives
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 1 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python3.12 1



# Install Python 3.12, GCC 13, and other system dependencies
RUN apt update && apt install -y \
    libglib2.0-0 \
    make \
    ninja-build \
    git \
    aria2 \
    fish \
    fd-find \
    vim \
    libopencv-dev \
    ffmpeg \
    libx264-dev \
    libx265-dev \
    fonts-noto-core \
    fonts-noto-cjk \
    fonts-noto-color-emoji \
    && ln -sf /usr/bin/fdfind /usr/bin/fd \
    && ln -sf /usr/bin/ninja-build /usr/bin/ninja \
    && ln -sf /usr/lib/x86_64-linux-gnu/libx264.so /usr/lib/x264 \
    && ln -sf /usr/lib/x86_64-linux-gnu/libx265.so /usr/lib/x265


RUN which aria2c || echo "aria2c not found"


# Install pip for Python 3.12
RUN curl https://bootstrap.pypa.io/get-pip.py | python3.12

################################################################################
# Set environment variables for CUDA paths
################################################################################

ENV PATH="${PATH}:/usr/local/cuda-12.4/bin" \
    LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/usr/local/cuda-12.4/lib64" \
    LIBRARY_PATH="${LIBRARY_PATH}:/usr/local/cuda-12.4/lib64/stubs" \
    CUDA_HOME="/usr/local/cuda-12.4"

################################################################################
# Install Python packages using pip
################################################################################

RUN pip install --no-cache-dir \
    wheel \
    setuptools \
    cython \
    aiohttp \
    gitpython \
    httpx \
    joblib \
    matplotlib \
    numba \
    numpy \
    onnx \
    opencv-python-headless \
    pandas \
    qrcode \
    rich \
    scikit-build \
    scikit-learn \
    scipy \
    svglib \
    tqdm \
    ffmpeg-python \
    lark-parser

################################################################################
# PyTorch, xFormers
################################################################################

RUN --mount=type=cache,target=/root/.cache/pip \
    pip list && \
    pip install --upgrade pip wheel setuptools && \
    pip install xformers torchvision torchaudio \
    --index-url https://download.pytorch.org/whl/cu124 \
    --extra-index-url https://pypi.org/simple

################################################################################
# Install additional fonts
################################################################################

#RUN apt-get update && apt-get install -y --no-install-recommends \
#    fonts-noto \
#    fonts-noto-cjk \
#    fonts-noto-color-emoji \
#    google-noto-sans-fonts \
#    google-noto-sans-cjk-fonts \
#    google-noto-coloremoji-fonts && \
#    apt-get clean && \
#    rm -rf /var/lib/apt/lists/*

################################################################################
# Additional Setup for CUDA libraries (optional, if needed)
################################################################################

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    --allow-change-held-packages \
    libcublas-12-4 \
    libcublas-dev-12-4 \
    libnpp-12-4 \
    libnpp-dev-12-4 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

################################################################################
# Copy builder scripts and install custom packages (ComfyUI & custom nodes)
################################################################################

COPY builder-scripts/.  /builder-scripts/

# 1. Install dependencies for ComfyUI & custom nodes.
# 2. Fix ONNX Runtime "missing CUDA provider" and add support for CUDA 12.
# 3. Fix MediaPipe's broken dependency (protobuf<4).
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install \
        -r /builder-scripts/pak3.txt && \
    pip install \
        -r /builder-scripts/pak5.txt && \
    pip install \
        -r /builder-scripts/pak7.txt && \
    pip uninstall --break-system-packages --yes \
        onnxruntime-gpu && \
    pip --no-cache-dir install --break-system-packages \
        onnxruntime-gpu \
        --index-url https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/onnxruntime-cuda-12/pypi/simple/ \
        --extra-index-url https://pypi.org/simple && \
    pip install \
        mediapipe && \
    pip list

################################################################################
# Bind CUDA-related shared libraries for Python
################################################################################

ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}\
:/usr/local/lib64/python3.12/site-packages/torch/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cuda_cupti/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cuda_runtime/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cudnn/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cufft/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cublas/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cuda_nvrtc/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/curand/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cusolver/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/cusparse/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/nccl/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/nvjitlink/lib\
:/usr/local/lib/python3.12/site-packages/nvidia/nvtx/lib"

################################################################################
# Cleanup files to minimize image size
################################################################################

RUN du -ah /root && \
    find /root/ -mindepth 1 -delete

################################################################################
# Copy runner scripts and set up final environment
################################################################################

COPY runner-scripts/.  /runner-scripts/

USER root
VOLUME /root
WORKDIR /root
EXPOSE 8188
ENV CLI_ARGS=""

CMD ["bash","/runner-scripts/entrypoint.sh"]

################################################################################
# End of Dockerfile
################################################################################
