FROM rocker/shiny:latest

# Override the rocker defaults with our defaults
COPY shiny-server.conf /etc/shiny-server/shiny-server.conf

# Install rsync
RUN apt-get update && apt-get install -y rsync

