FROM python:3.10-alpine

ARG REPO_DIR

EXPOSE 8000

ENV CRYPTIXD_HOST_1=127.0.0.1:19201
ARG version=1.0.0
ENV VERSION=${version:-1.0.0}

RUN apk --no-cache add \
  git \
  gcc \
  libc-dev \
  build-base \
  linux-headers \
  libpq-dev \
  dumb-init

RUN pip install \
  pipenv \
  grpcio-tools==1.63.0

RUN addgroup -S -g 55747 api \
  && adduser -h /app -S -D -g '' -G api -u 55747 api

WORKDIR /app
USER api

COPY --chown=api:api . /app

# Generate protobuf files and fix imports
RUN cd cryptixd && python -m grpc_tools.protoc \
    -I./protos \
    --python_out=. \
    --grpc_python_out=. \
    ./protos/rpc.proto ./protos/messages.proto \
    && sed -i 's/^import rpc_pb2/from . import rpc_pb2/' messages_pb2.py \
    && sed -i 's/^import messages_pb2/from . import messages_pb2/' messages_pb2_grpc.py

RUN pipenv install --deploy

ENTRYPOINT ["/usr/bin/dumb-init", "--"]

CMD pipenv run gunicorn -b 0.0.0.0:8000 -w 1 -k uvicorn.workers.UvicornWorker main:app --timeout 120
