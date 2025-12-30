FROM swr.cn-south-1.myhuaweicloud.com/ascendhub/cann:8.5.0.alpha002-a3-ubuntu22.04-py3.11 as base

# 设置驱动路径环境变量
ARG ASCEND_BASE=/usr/local/Ascend
ENV LD_LIBRARY_PATH=\
$ASCEND_BASE/driver/lib64:\
$ASCEND_BASE/driver/lib64/common:\
$ASCEND_BASE/driver/lib64/driver:\
$ASCEND_BASE/driver/tools/hccn_tool/:/lib64:\
$LD_LIBRARY_PATH
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai
RUN sed -i 's/ports.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list && \
    apt-get update -y && \
    apt-get install -y net-tools wget git vim curl libnuma-dev unzip && \
    apt-get install git cmake build-essential libgl1-mesa-dev libsdl2-dev \
        libsdl2-image-dev libsdl2-ttf-dev libsdl2-gfx-dev libboost-all-dev \
        libdirectfb-dev libst-dev mesa-utils xvfb x11vnc apt-utils -y && \
    apt-get clean && \
    git config --global http.sslVerify "false" && \
    pip config set global.index-url https://mirrors.huaweicloud.com/repository/pypi/simple && \
    pip config set global.trusted-host mirrors.huaweicloud.com && \
    pip config set global.timeout 120 && \
    pip config set global.extra-index-url "https://mirrors.huaweicloud.com/ascend/repos/pypi" && \
    pip install --upgrade pip setuptools wheel psutil seaborn  --root-user-action=ignore && \
    pip cache purge

FROM base as mmapo

WORKDIR /workspace
RUN git clone https://github.com/Yanguan619/on-policy-npu.git on-policy && \
    cd on-policy && pip install -e .

# wget https://blzdistsc2-a.akamaihd.net/Linux/SC2.4.10.zip && \
# unzip SC2.4.10.zip && \
RUN cd /workspace/on-policy && \
    echo "export SC2PATH=~/StarCraftII/" >> ~/.bashrc && \
    wget https://gh-proxy.top/https://github.com/oxwhirl/smac/releases/download/v0.1-beta1/SMAC_Maps.zip && \
    unzip SMAC_Maps.zip && \
    mkdir -p ~/StarCraftII/Maps/ && \
    cp -r ./SMAC_Maps/* ~/StarCraftII/Maps/ && \
    cp ./assets/stableid.json ~/StarCraftII/

RUN cd /workspace/on-policy/onpolicy && \
    pip install cffi && \
    cd envs/hanabi && \
    mkdir -p build && cd build && cmake .. && make -j

RUN pip install gfootball --no-build-isolation && \
    cd /usr/local/python3.11.13/lib/python3.11/site-packages/gfootball_engine && \
    cmake . && make -j `nproc` && \
    ln -s libgame.so _gameplayfootball.so -f && \
    python3 -m gfootball.play_game --action_set=full

RUN pip install numpy==1.26.4 --root-user-action=ignore
