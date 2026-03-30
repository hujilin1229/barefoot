

# Copyright (C) 2015, BMW Car IT GmbH
#
# Author: Sebastian Mattheis <sebastian.mattheis@bmw-carit.de>
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
# in compliance with the License. You may obtain a copy of the License at
# http://www.apache.org/licenses/LICENSE-2.0 Unless required by applicable law or agreed to in
# writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific
# language governing permissions and limitations under the License.
#

FROM ubuntu:16.04


MAINTAINER sebastian.mattheis@bmw-carit.de

ADD /pgsql/ /opt/pgsql/
#RUN apt-get update && apt-get -y install python3 patch git 
RUN apt-get update && apt-get -y install python3 python3-psycopg2 python3-numpy python3-gdal patch git postgresql-17-postgis-3
#RUN pip install psycopg2 numpy gdal
RUN patch /etc/postgresql/17/main/postgresql.conf < /opt/pgsql/postgresql.conf.patch && patch /etc/postgresql/17/main/pg_hba.conf < /opt/pgsql/pg_hba.conf.patch && echo "export HOME=/root" >> /root/.bashrc
RUN apt-get install osmosis && echo "export PATH=${PATH}:/opt/osmosis/package/bin" >> /root/.bashrc && echo "host all all all md5" >> /etc/postgresql/17/main/pg_hba.conf
CMD service postgresql start && /bin/bash --rcfile /root/.bashrc
