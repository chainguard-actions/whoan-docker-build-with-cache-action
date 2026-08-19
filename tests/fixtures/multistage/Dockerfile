FROM alpine:3.18 AS builder
RUN echo "building stage"

FROM alpine:3.18
COPY --from=builder /etc/alpine-release /etc/alpine-release
RUN echo "final stage"
