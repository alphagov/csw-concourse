FROM gdscyber/cyber-chalice:1.0

LABEL ubuntu="18.04"
LABEL user="gdscyber"

# load custom scripts
COPY bin /usr/local/bin

RUN mkdir -p /root/.ssh && touch /root/.ssh/config

ENTRYPOINT ["bash"]
