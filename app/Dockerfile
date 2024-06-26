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

# Install required packages
RUN apt-get update && apt-get install -y python3-dev gcc

# Install Python dependencies
RUN pip3 install --no-cache-dir grpcio-tools pyinstaller==5.9.0 patchelf==0.17.0.0 staticx

# Copy the application code
COPY ./.velocitas.json /workspace/.velocitas.json
COPY ./app /workspace/app

# Install application dependencies
RUN pip3 install --no-cache-dir -r /workspace/app/requirements.txt \
    && pip3 install --no-cache-dir -r /workspace/app/requirements-links.txt

# Run Velocitas initialization
WORKDIR /workspace
RUN velocitas init

# Generate gRPC code from .proto files
WORKDIR /workspace/app
RUN python3 -m grpc_tools.protoc -I/workspace/app/src/vehicle_model/proto --python_out=/workspace/app/generated --grpc_python_out=/workspace/app/generated /workspace/app/protos/*.proto

# Package the application
RUN pyinstaller --clean -F -s --paths=src src/main.py

# Create a static executable
WORKDIR /workspace/app/dist
RUN staticx main run-exe

# Runner stage, to copy the executable
FROM scratch as runner

COPY --from=builder /workspace/app/dist/run-exe /app

WORKDIR /tmp

ENV PATH="/:$PATH"

LABEL org.opencontainers.image.source="https://github.com/eclipse-velocitas/vehicle-app-python-template"

CMD ["/app"]
