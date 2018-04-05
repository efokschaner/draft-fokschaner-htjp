FROM fedora:27

# Install binaries from Fedora needed for python, xml2rfc, ruby
RUN dnf -y install python python-lxml git ruby && \
    dnf clean all

# Update pip and install xml2rfc, creating the default cache directory
RUN pip install --upgrade pip && \
    pip install --compile xml2rfc && \
    mkdir -p /var/cache/xml2rfc

RUN gem install --no-doc \
    certified kramdown-rfc2629 && \
    certified-update

# Specify the working directory when a container is started
WORKDIR /rfc

CMD ["kdrfc", "draft-fokschaner-htjp.md"]