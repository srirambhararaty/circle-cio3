FROM 017353844903.dkr.ecr.us-east-1.amazonaws.com/cbre-dt-golang:{{ GOLANG_VERSION }} AS build

MAINTAINER CBRE Data Technologies

RUN apt-get update && \
    apt-get install -y python-pip && \
    pip install slacker && \
    rm -rf /var/lib/apt/lists/*

# Adding lymbic dependent micro-service code to GOPATH
ADD . /go/src/{{ SOURCE_DIR }}

# Moving gi-dp dependencies into GOPATH
# lymbic-utils can be removed if you're not referring package from it
COPY /build/lymbic-utils /go/src/lymbic-utils
COPY /build/lymbic /go/src/lymbic
COPY /build/lymbic/vendor /go/src/
COPY /vendor /go/src/

# Building gi-dp lymbic binary
WORKDIR $GOPATH/src/{{ SOURCE_DIR }}
RUN go install -ldflags "-X main.MajorVersion={{ MAJOR_VERSION }} -X main.MinorVersion={{ MINOR_VERSION }} -X main.PatchVersion={{ PATCH_VERSION }} -X main.BuildNumber={{ BUILD_NUMBER }} -X 'main.BuildTime={{ BUILD_TIMESTAMP }}'" lymbic.go

## This results in a single layer image
FROM 017353844903.dkr.ecr.us-east-1.amazonaws.com/cbre-dt:1.0

RUN apt-get update && \
    apt-get install -y python-pip && \
    pip install slacker && \
    pip install awscli && \
    rm -rf /var/lib/apt/lists/*

ENV GOLANG_VERSION {{ GOLANG_VERSION }}
ENV GOBIN /go/bin
ENV PATH $PATH:${GOBIN}

COPY --from=build /go/bin /go/bin
COPY /build-scripts/docker-entrypoint.sh /docker-entrypoint.sh
#for runnnig in MESOS, comment the following line and for running locally uncomment the below line.
#COPY /app-configs /opt/app-configs/

RUN mkdir -p /opt/app-configs/ && \
    chmod -R 777 /opt/app-configs && \
    groupadd -r -g 999 appgroup && \
    useradd --no-log-init -r -u 999 -g appgroup appuser
USER appuser

ENTRYPOINT ["/docker-entrypoint.sh"]
