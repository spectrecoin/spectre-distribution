FROM spectreproject/github-uploader:latest
MAINTAINER HLXEasy <hlxeasy@gmail.com>

ARG GITHUB_TOKEN=1234567
ARG SPECTRECOIN_RELEASE=latest
ARG SPECTRECOIN_REPOSITORY=spectre
ARG REPLACE_EXISTING_ARCHIVE=''
#ENV GITHUB_TOKEN=${GITHUB_TOKEN}
ENV ARCHIVE=Spectrecoin-${SPECTRECOIN_RELEASE}-Ubuntu.tgz

RUN mkdir -p /filesToUpload/usr/local/bin

# Unfortunately it is not possible to use parameters on "--from=...",
# so we need separate Dockerfiles
COPY --from=spectreproject/spectre-ubuntu:latest /usr/local/bin/spectrecoin  /filesToUpload/usr/local/bin/
COPY --from=spectreproject/spectre-ubuntu:latest /usr/local/bin/spectrecoind /filesToUpload/usr/local/bin/

RUN cd /filesToUpload \
 && tar czf ${ARCHIVE} . \
 && github-release upload \
        --user spectrecoin \
        --security-token "${GITHUB_TOKEN}" \
        --repo "${SPECTRECOIN_REPOSITORY}" \
        --tag "${SPECTRECOIN_RELEASE}" \
        --name "${ARCHIVE}" \
        --file "/filesToUpload/${ARCHIVE}" \
        ${REPLACE_EXISTING_ARCHIVE} \
 && rm -rf /filesToUpload/* \
 && export GITHUB_TOKEN=---
