#
# Dockerfile to build Docker CI image for DIRBS
#
# Assumes that the DIRBS git repo is bind-mounted to /home/dirbs/git
#
# Copyright (c) 2018 Qualcomm Technologies, Inc.
#
# All rights reserved.
#
#
#
# Redistribution and use in source and binary forms, with or without modification, are permitted (subject to the
# limitations in the disclaimer below) provided that the following conditions are met:
#
#
# * Redistributions of source code must retain the above copyright notice, this list of conditions and the following
# disclaimer.
#
# * Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following
# disclaimer in the documentation and/or other materials provided with the distribution.
#
# * Neither the name of Qualcomm Technologies, Inc. nor the names of its contributors may be used to endorse or promote
# products derived from this software without specific prior written permission.
#
# NO EXPRESS OR IMPLIED LICENSES TO ANY PARTY'S PATENT RIGHTS ARE GRANTED BY THIS LICENSE. THIS SOFTWARE IS PROVIDED BY
# THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
# THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
# COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
# OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR
# TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
#

FROM dirbs-base

ENV HLL_VERSION=2.10.2

# Install Postgres 10, git and eslint
RUN set -x && \
    echo 'deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main' > /etc/apt/sources.list.d/pgdg.list && \
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - && \
    apt-get -q update --fix-missing && \
    apt-get -q install -y --no-install-recommends postgresql-10 postgresql-contrib-10 postgresql-server-dev-10 \
                                                  postgresql-common git nodejs npm rsync wget unzip \
                                                  build-essential && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# NPM packages for verifying CSS/JS
RUN set -x && \
    ln -s nodejs /usr/bin/node && \
    npm install -g eslint && \
    npm install -g uncss && \
    npm cache clean

# Symlink workspace assets
RUN set -x && \
    mkdir -p /opt/dirbs && \
    ln -s /workspace/etc /opt/dirbs/etc && \
    ln -s /workspace/etc/config.yml /home/dirbs/.dirbs.yml

# Install opensource dependencies
COPY opensource_requirements.txt /home/dirbs
RUN set -x && \
    pip3 install -r /home/dirbs/opensource_requirements.txt && \
    chown -R dirbs.dirbs /home/dirbs/dirbs-venv

# install hll
RUN set -x && \
    wget -q -O hll.zip https://github.com/citusdata/postgresql-hll/archive/v$HLL_VERSION.zip && \
    unzip hll.zip && \
    cd postgresql-hll-$HLL_VERSION && \
    make install

# Expose port 5000 for dev API server
EXPOSE 5000

# Mark volumes
VOLUME ["/workspace"]

# Copy entrypoint script
COPY docker/dev/entrypoint.sh /entrypoint.sh

# Make sure permissions are set properly on entrypoint
RUN set -x && chmod a+x /entrypoint.sh

# Set entrypoint
ENTRYPOINT ["gosu", "dirbs", "/entrypoint.sh"]

# Set default command - interpreted by entrypoint
CMD ["shell"]
