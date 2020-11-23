# ARG instruction Dockerfile for Docker Quick Start
FROM alpine
LABEL maintainer="sangam biradar"

ENV lifecycle="production"
RUN echo ${lifecycle}
ARG username="35"
RUN echo ${username}
ARG appdir
RUN echo ${appdir}

ADD hello /${appdir}/
RUN chown -R ${username}:${username} ${appdir}
WORKDIR ${appdir}
USER ${username}

CMD ["./hello"]
