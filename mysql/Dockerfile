# FROM mysql:5.5.47
FROM mysql:5.7

COPY conf/localdb-run.sh /
RUN chmod 755 /localdb-run.sh

RUN usermod -u 1000 www-data
RUN usermod -G staff www-data

VOLUME ["/var/db"]

ENTRYPOINT ["/localdb-run.sh"]
