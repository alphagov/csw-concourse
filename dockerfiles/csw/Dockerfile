FROM gdscyber/cyber-chalice:2.0

LABEL ubuntu="18.04"
LABEL user="gdscyber"

ENV GK_VER v0.24.0
ENV GK_URL https://github.com/mozilla/geckodriver/releases/download

# load custom scripts
COPY bin /usr/local/bin

# create ssh file and config
RUN mkdir -p /root/.ssh && touch /root/.ssh/config

# install geckodriver and firefox
WORKDIR /tmp

RUN curl -s -L ${GK_URL}/${GK_VER}/geckodriver-${GK_VER}-linux64.tar.gz | tar -xz && \
    mv geckodriver /usr/bin/geckodriver

RUN apt-get update -y && apt-get -y install firefox

ENTRYPOINT ["bash"]
