FROM gdscyber/cyber-security-concourse-base-image

RUN apt-get update -y
# cyber-security-concourse-base-image installs python3.7-venv, it *should* work
# RUN apt-get install -y python3-venv
RUN apt-get install -y nodejs npm
RUN apt-get install -y expect

# install n and upgrade node to n stable
RUN npm install -g n
RUN n 10.12.0

# install gulp-cli
RUN npm install -g gulp-cli

# install npm-reinstall
# RUN npm install -g npm-reinstall

ENTRYPOINT ["bash"]
