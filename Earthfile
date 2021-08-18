cpp:
    FROM ubuntu:20.10
    ## for apt to be noninteractive
    ENV DEBIAN_FRONTEND noninteractive
    ENV DEBCONF_NONINTERACTIVE_SEEN true
    WORKDIR /code
    RUN apt-get update && apt-get install -y build-essential cmake zlib1g-dev

proxy:
    FROM +cpp
    COPY services/proxy .
    RUN make
    SAVE ARTIFACT wsproxy AS LOCAL "build/wsproxy"

server:
    FROM +cpp
    COPY services/server .
    RUN cmake .
    # cache cmake temp files to prevent rebuilding .o files
    # when the .cpp files don't change
    RUN --mount=type=cache,target=/code/CMakeFiles make
    SAVE ARTIFACT qserv AS LOCAL "build/qserv"

game:
    FROM emscripten/emsdk:1.40.0
    WORKDIR /cube2
    COPY services/game/cube2 cube2
    RUN cd cube2/src/web && emmake make client -j8
    RUN mkdir dist && \
        cp -r cube2/*.html cube2/game cube2/js cube2/*.js cube2/*.wasm cube2/*.data dist
    SAVE ARTIFACT dist /dist AS LOCAL "build/game"

client:
    FROM node:14.17.5
    WORKDIR /client
    COPY services/client .
    RUN --mount=type=cache,target=/code/node_modules yarn install
    RUN rm -r dist && yarn build && cp src/index.html dist
    SAVE ARTIFACT dist /dist AS LOCAL "build/client"

docker:
  FROM ubuntu:20.10
  # For the SimpleHTTPServer
  RUN apt-get update && apt-get install -y python
  COPY +server/qserv /bin/qserv
  COPY +proxy/wsproxy /bin/wsproxy
  COPY +game/dist /app/
  COPY +client/dist /app/
  COPY services/server/config /qserv/config
  COPY entrypoint /bin/entrypoint
  CMD ["/bin/entrypoint"]
  SAVE IMAGE sour:latest

push:
  FROM +docker
  SAVE IMAGE --push registry.digitalocean.com/cfoust/sour:latest
