FROM python:3.10-slim
ARG MIRROR
LABEL authors="nexent"

# Set correct permissions as root
USER root

RUN rm -f /etc/apt/sources.list.d/* && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware" > /etc/apt/sources.list && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware" >> /etc/apt/sources.list && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware" >> /etc/apt/sources.list && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware" >> /etc/apt/sources.list

RUN apt-get update && \
    apt-get install -y --no-install-recommends --fix-missing \
        curl \
        libmagic1 \
        libmagic-dev \
        libreoffice \
        libgl1 \
        coreutils \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN pip install --no-cache-dir uv $(test -n "$MIRROR" && echo "-i $MIRROR")
# Layer 0: copy model assets
COPY model-assets/clip-vit-base-patch32 /opt/models/clip-vit-base-patch32
COPY model-assets/nltk_data /opt/models/nltk_data

WORKDIR /opt/backend
# Layer 1: install base dependencies
COPY backend/pyproject.toml /opt/backend/pyproject.toml
RUN uv sync --extra data-process $(test -n "$MIRROR" && echo "-i $MIRROR")
# Layer 2: install sdk in link mode
COPY sdk /opt/sdk
RUN uv pip install /opt/sdk $(test -n "$MIRROR" && echo "-i $MIRROR") && \
    uv cache clean
# Layer 3: copy backend code
COPY backend /opt/backend

ENV VIRTUAL_ENV=/opt/backend/.venv
ENV PATH="$VIRTUAL_ENV/bin:/usr/bin:/bin:/usr/local/bin:$PATH"

WORKDIR /opt

# Expose the service port
EXPOSE 5012
