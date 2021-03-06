# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM debian:jessie
LABEL maintainer="Benjamin Elder <bentheelder@google.com>"

# Includes everything needed to build and test github.com/kubernetes/kubernetes
# and github.com/kubernetes/test-infra with bazel and run bazel as the host UID

ENV BAZEL_VERSION 0.5.4

WORKDIR "/workspace"
RUN mkdir -p "/workspace"

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    git \
    python \
    python-pip \
    unzip \
    wget \
    zip \
    zlib1g-dev \
    && apt-get clean && \
    # TODO(bentheelder): we should probably have these as bazel dependencies,
    # remove these from the container if kettle is fixed.
    pip install pylint pyyaml

RUN INSTALLER="bazel-${BAZEL_VERSION}-installer-linux-x86_64.sh"; \
    wget -q "https://github.com/bazelbuild/bazel/releases/download/${BAZEL_VERSION}/${INSTALLER}" && \
    chmod +x "${INSTALLER}" && "./${INSTALLER}" && rm "${INSTALLER}"

# It turns out git and bazel pkg_tar and a bunch of other things fail if we
# don't have a /etc/passwd with the user in it, so let's solve that.
# We're making a fake world-writable one *inside* the container so that the
# entrypoint can write an entry to it with the user matching the host user
# this means we don't need to run the container as root! :-)
# We care about not running as root because it means build files are owned by
# the host users, not root (and logs, etc).
RUN touch /etc/passwd && chmod a+rw /etc/passwd

COPY ./entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]
