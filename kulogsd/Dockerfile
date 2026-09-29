FROM debian:trixie-slim
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    iproute2 net-tools vim \
    iputils-ping telnet dnsutils curl \
    python3-pip python3-venv \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY ./kulogsd_config_example.json ./kulogsd_config.json
COPY ./kulogsd .
COPY ./kulogsd_loop.sh .

ENTRYPOINT ["./kulogsd_loop.sh"]
