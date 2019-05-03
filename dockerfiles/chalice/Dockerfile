FROM amazonlinux:latest

ENV TF_VERSION 0.11.13

# expect need local time zone
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

LABEL amazonlinux="latest"
LABEL terraform="$TF_VERSION"
LABEL user="gdscyber"

RUN yum update -y && \
        yum groupinstall -y "Development Tools" && \
        yum install -y \
        https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && \
        yum install -y \
        --enablerepo=epel \
        -y \
        awscli \
        expect \
        gcc \
        gcc-c++ make \
        git \
        jq \
        nodejs \
        npm \
        openssl-devel \
        python36 \
        python37 \
        unzip

# install n and upgrade node to n stable
RUN npm install -g n
RUN n 10.12.0

# install gulp-cli
RUN npm install -g gulp-cli

# Python 3 encoding set
ENV LANG C.UTF-8

# install terraform
WORKDIR /tmp

RUN curl https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip > terraform.zip && \
    echo '5925cd4d81e7d8f42a0054df2aafd66e2ab7408dbed2bd748f0022cfe592f8d2  terraform.zip'  > terraform.sha && \
    sha256sum -c terraform.sha && unzip terraform.zip && mv terraform /usr/bin/terraform                    && \
    rm terraform.zip && rm terraform.sha

ENTRYPOINT ["bash"]
