# syntax=docker/dockerfile:1
FROM debian:bullseye

RUN sh -c echo "deb http://deb.debian.org/debian bullseye-backports main" >> /etc/apt/sources.list
RUN sh -c echo "deb-src http://deb.debian.org/debian bullseye-backports main" >> /etc/apt/sources.list

RUN apt-get update && apt-get install -y gnupg2 && apt-get install -y apt-utils  && apt-get install -y wget
# Add the PostgreSQL PGP key to verify their Debian packages.
# It should be the same key as https://www.postgresql.org/media/keys/ACCC4CF8.asc
#RUN apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8

# Create the file repository configuration:
RUN sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt bullseye-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Import the repository signing key:
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -

# Update the package lists:
RUN apt-get update

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
RUN apt-get -y install postgresql-12

RUN yes '' | /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

RUN sh -c "echo 'deb [signed-by=/usr/share/keyrings/timescale.keyring] https://packagecloud.io/timescale/timescaledb/debian/ bullseye main' > /etc/apt/sources.list.d/timescaledb.list"

RUN wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | gpg --dearmor -o /usr/share/keyrings/timescale.keyring

RUN apt update

RUN apt install -y timescaledb-2-postgresql-12

RUN apt install -y timescaledb-tools

RUN sh -c 'echo "shared_preload_libraries = 'timescaledb'" >> /etc/postgresql/12/main/postgresql.conf' 

RUN useradd -ms /bin/bash -p evita opennms

# Run the rest of the commands as the ``postgres`` user created by the ``postgres-9.3`` package when it was ``apt-get installed``
USER postgres

# Create a PostgreSQL role named ``docker`` with ``docker`` as the password and
# then create a database `docker` owned by the ``docker`` role.
# Note: here we use ``&&\`` to run commands one after the other - the ``\``
#       allows the RUN command to span multiple lines.
RUN    /etc/init.d/postgresql start &&\
    psql --command "CREATE USER opennms WITH PASSWORD 'opennms';" &&\
    createdb -O opennms opennms

# Adjust PostgreSQL configuration so that remote connections to the
# database are possible.
RUN echo "host all  all    0.0.0.0/0  trust\n\ 
local all  all 	     trust" >> /etc/postgresql/12/main/pg_hba.conf

# And add ``listen_addresses`` to ``/etc/postgresql/12/main/postgresql.conf``
RUN echo "listen_addresses='*'" >> /etc/postgresql/12/main/postgresql.conf

# Expose the PostgreSQL port
EXPOSE 5432

# Add VOLUMEs to allow backup of config, logs and databases
VOLUME  ["/etc/postgresql", "/var/log/postgresql", "/var/lib/postgresql"]

# Set the default command to run when starting the container
CMD ["/usr/lib/postgresql/12/bin/postgres", "-D", "/var/lib/postgresql/12/main", "-c", "config_file=/etc/postgresql/12/main/postgresql.conf"]
