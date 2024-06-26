# Copyright (c) 2024 Contributors to the Eclipse Foundation
#
# This program and the accompanying materials are made available under the
# terms of the Apache License, Version 2.0 which is available at
# https://www.apache.org/licenses/LICENSE-2.0.
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations
# under the License.
#
# SPDX-License-Identifier: Apache-2.0

# syntax=docker/dockerfile:1.2

# Build stage, to create the executable
FROM --platform=$TARGETPLATFORM ghcr.io/eclipse-velocitas/devcontainer-base-images/python:v0.3 as builder
ARG TARGETARCH

# Install necessary dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    python3-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install grpcio and grpcio-tools
RUN pip3 install --no-cache-dir grpcio==1.64.1 grpcio-tools==1.64.1

# Create necessary directories and copy files
WORKDIR /workspace
COPY ./.velocitas.json /workspace/.velocitas.json
COPY ./app /workspace/app

# Check if the directory ./app/src/vehicle_model/proto exists
RUN ls -l ./app/src/vehicle_model/proto

# Copy proto files into the workspace
COPY ./app/src/vehicle_model/proto /workspace/proto

# Install Python requirements
RUN pip3 install --no-cache-dir -r /workspace/app/requirements.txt
RUN pip3 install --no-cache-dir -r /workspace/app/requirements-links.txt

# Initialize velocitas (assuming this is necessary for your application)
RUN velocitas init

# Set environment variable for proto directory
ENV PROTO_DIR=/workspace/proto

# Create the generated directory
RUN mkdir -p /workspace/app/generated

# Generate service SDK for hvac.proto
RUN python3 -m grpc_tools.protoc \
    -I${PROTO_DIR} \
    --python_out=/workspace/app/generated \
    --grpc_python_out=/workspace/app/generated \
    ${PROTO_DIR}/hvac.proto

# Install build tools one by one and check installation
RUN pip3 install --no-cache-dir pyinstaller==5.9.0 && pyinstaller --version
RUN pip3 install --no-cache-dir patchelf==0.17.0.0 && patchelf --version
RUN pip3 install --no-cache-dir staticx && staticx --version
RUN pip3 install --no-cache-dir scons && scons --version

WORKDIR /workspace/app

# Build the application using pyinstaller
RUN pyinstaller --clean -F -s --paths=src src/main.py

WORKDIR /workspace/app/dist

# Use staticx to create a statically linked executable
RUN staticx main run-exe

# Runner stage, to copy the executable
FROM scratch as runner

COPY --from=builder /workspace/app/dist/run-exe /app

WORKDIR /tmp

ENV PATH="/:$PATH"

LABEL org.opencontainers.image.source="https://github.com/eclipse-velocitas/vehicle-app-python-template"

CMD ["/app"]
