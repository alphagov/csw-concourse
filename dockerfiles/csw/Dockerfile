FROM ubuntu:18.04

ENV TF_VERSION 0.11.13

# expect need local time zone
ENV TZ=Europe/London
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

LABEL ubuntu="18.04"
LABEL terraform="$TF_VERSION"
LABEL user="gdscyber"

RUN apt-get update -y  && \
    apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
    libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
    xz-utils tk-dev libffi-dev liblzma-dev python-openssl git
RUN apt-get install -y unzip nodejs npm
RUN apt-get install -y expect awscli jq dnsutils

# install n and upgrade node to n stable
RUN npm install -g n
RUN n 10.12.0

# install gulp-cli
RUN npm install -g gulp-cli

# install npm-reinstall
RUN npm install -g npm-reinstall

# Python 3 encoding set
ENV LANG C.UTF-8

# install pyenv
RUN curl https://pyenv.run | bash

ENV PATH="/root/.pyenv/shims:/root/.pyenv/bin:${PATH}"

RUN pyenv install 3.6.5
RUN pyenv global 3.6.5
RUN pyenv rehash

# load custom scripts
COPY bin/aws-assume-role /usr/local/bin/
COPY bin/buildcsw /usr/local/bin/

RUN mkdir -p /root/.ssh && touch /root/.ssh/config

# install terraform
WORKDIR /tmp

RUN curl https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip > terraform.zip && \
    echo '5925cd4d81e7d8f42a0054df2aafd66e2ab7408dbed2bd748f0022cfe592f8d2  terraform.zip'  > terraform.sha && \
    sha256sum -c terraform.sha && unzip terraform.zip && mv terraform /usr/bin/terraform                    && \
    rm terraform.zip && rm terraform.sha

ENTRYPOINT ["bash"]
