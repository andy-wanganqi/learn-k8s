# Build container image
FROM alpine as cmatrixbuilder

WORKDIR cmatrix

RUN apk update --no-cache && \
    apk add git autoconf automake alpine-sdk ncurses-dev ncurses-static && \
    git clone https://github.com/andy-wanganqi/cmatrix.git . && \
    autoreconf -i && \
    mkdir -p /usr/lib/kbd/consolefonts /usr/share/consolefonts && \
    ./configure LDFLAGS="-static" && \
    make


#cmatrix container image
FROM alpine

LABEL org.opencontainers.image.authors="Andy Wang" \
      org.opencontainers.image.description="Container image for https://github.com/abishekvashok/cmatrix"

RUN apk update --no-cache && \
    apk add ncurses-terminfo-base


COPY --from=cmatrixbuilder /cmatrix/cmatrix /cmatrix

# USER sets the user name and optionally the user group to use for the remainer of the current stage
# The specified user is used for RUN instructions at runtime
RUN adduser -g "Andy" -s /usr/sbin/nologin -D -H andy
# This RUN command can be merged into the RUN above
USER andy

# ENTRYPOINT allows you to configure a container that will run an executable file
# Command line arguments to 'docker run <image>' will be appended after all elements in an exec form of ENTRYPOINT
ENTRYPOINT ["./cmatrix"]
CMD ["-b"]
# Above, with ENTRYPOINT, you can try: docker run --rm -it andywanganqi/cmatrix -ab -u 2 -C magenta
