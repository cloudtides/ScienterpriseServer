FROM boinc/server_apache:4.0.1-b2d
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
	vim \
	emacs \
	tree \
    && rm -rf /var/lib/apt/lists

RUN rm /etc/apache2/sites-enabled/000-default.conf
COPY --chown=1000 apache2.conf /etc/apache2/
