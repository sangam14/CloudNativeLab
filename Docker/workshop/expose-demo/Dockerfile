# EXPOSE instruction Dockerfile for Docker Quick Start
FROM alpine
EXPOSE 80/tcp
EXPOSE 81/tcp
EXPOSE 8080/udp
CMD while true; do echo 'DQS Expose Demo' | nc -l -p 80; done
