FROM ghcr.io/arm64-compat/confluentinc/cp-kafka-connect-base:7.1.1

USER root

#COPY REPOSITORY CONTENTS
COPY distributed.properties /etc



#INSTALL CONFLUENT HUB to access more connectors
#RUN ["curl", "-o", "confluent-hub-client-latest.tar.gz", "http://client.hub.confluent.io/confluent-hub-client-latest.tar.gz"]
#RUN ["/bin/sh", "-c", "mkdir -p confluent-hub && tar -xvf confluent-hub-client-latest.tar.gz -C confluent-hub"]
#RUN ["/bin/sh", "-c", "rm -f confluent-hub-client-latest.tar.gz"]

# INSTALL EXTRA CONNECTORS
#RUN ["./confluent-hub/bin/confluent-hub", "install", "confluentinc/kafka-connect-elasticsearch:4.1.1", "--no-prompt"]
#RUN confluent-hub install --no-prompt confluentinc/kafka-connect-elasticsearch:4.1.1

ENTRYPOINT ["/bin/bash","/bin/connect-distributed", "/etc/distributed.properties"]