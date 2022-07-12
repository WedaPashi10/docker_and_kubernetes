FROM cpp-build-base:0.1.0 AS build

WORKDIR /src

COPY main.cpp Makefile ./

RUN make

FROM ubuntu:focal

WORKDIR /opt/hello-world

COPY --from=build /src/a.out ./

CMD ["./a.out"]
