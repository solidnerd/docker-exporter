FROM alpine:3.4

RUN apk add --no-cache socat 

ENV "IN=172.18.0.1:5050" \
    "OUT=5050"

ENTRYPOINT socat -d -d TCP-L:$OUT,fork TCP:$IN