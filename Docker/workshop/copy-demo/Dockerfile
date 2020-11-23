# COPY instruction Dockerfile for Docker Quick Start
FROM alpine:latest
LABEL version=1.0
COPY file* theqsg/files/
COPY folder1 theqsg/
WORKDIR theqsg
COPY --chown=35:35 special1 special-files/
WORKDIR /
CMD ["sh"]
