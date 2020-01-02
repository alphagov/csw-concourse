FROM ubuntu:18.04

ENV TF_VERSION 0.11.14
ENV TF_CHECKSUM 9b9a4492738c69077b079e595f5b2a9ef1bc4e8fb5596610f69a6f322a8af8dd

# expect need local time zone
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

LABEL ubuntu="18.04"
LABEL terraform="$TF_VERSION"
LABEL user="gdscyber"

RUN apt-get update -y
RUN apt-get install -y python3.6 python3-distutils python3-pip
RUN apt-get install -y build-essential libssl-dev libffi-dev python3-dev
RUN apt-get install -y python3-venv
RUN apt-get install -y nodejs npm
RUN apt-get install -y expect awscli jq curl unzip git

# install n and upgrade node to n stable
RUN npm install -g n
RUN n 10.12.0

# install gulp-cli
RUN npm install -g gulp-cli

# install npm-reinstall
# RUN npm install -g npm-reinstall

# Python 3 encoding set
ENV LANG C.UTF-8

# install terraform
WORKDIR /tmp

RUN curl https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip > terraform.zip && \
    echo "${TF_CHECKSUM}  terraform.zip"  > terraform.sha && \
    sha256sum -c terraform.sha && unzip terraform.zip && mv terraform /usr/bin/terraform                    && \
    rm terraform.zip && rm terraform.sha

ENTRYPOINT ["bash"]
