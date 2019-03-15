# This Dockerfile derives a customized Tripal image for DivSeek Canada

# This is the docker-tripal "master" image, which is now at release 3.x
FROM quay.io/galaxy-genome-annotation/tripal:latest

MAINTAINER Richard Bruskiewich <richard@starinformatics.com>

# Set user selected admin account name
ARG DIVSEEK_ADMIN_USER
ENV ADMIN_USER $DIVSEEK_ADMIN_USER

# Point to actual Chado database host
ARG DIVSEEK_DB_HOST
ENV DB_HOST $DIVSEEK_DB_HOST

COPY ./DivSeekCanadaLogo.* /

# Overlay DivSeek Canada specific installation scripts
# added to the core docker-tripal start-up scripts in the image
# to be run in the prescribed order
COPY ./scripts /scripts/

# Overrides the docker-tripal image CMD entrypoint.sh
# with a slightly different version here?
COPY entrypoint.sh  /entrypoint.sh

# Make sure that all our scripts are executable
RUN chmod -R +x /scripts/ /entrypoint.sh

CMD ["/entrypoint.sh"]
