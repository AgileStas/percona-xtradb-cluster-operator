FROM goboring/golang:1.16.7b7 AS go_builder
WORKDIR /go/src/github.com/percona/percona-xtradb-cluster-operator

COPY . .

ARG GIT_COMMIT
ARG GIT_BRANCH
ARG BUILD_TIME
ARG GO_LDFLAGS
ARG GOOS=linux
ARG GOARCH=amd64
ARG CGO_ENABLED=0

RUN ls -la && mkdir -p build/_output/bin \
    && GOOS=$GOOS GOARCH=$GOARCH CGO_ENABLED=$CGO_ENABLED GO_LDFLAGS=$GO_LDFLAGS \
       go build -mod=vendor -ldflags "-w -s -X main.GitCommit=$GIT_COMMIT -X main.GitBranch=$GIT_BRANCH -X main.BuildTime=$BUILD_TIME" \
           -o build/_output/bin/percona-xtradb-cluster-operator \
               cmd/manager/main.go \
    && cp -r build/_output/bin/percona-xtradb-cluster-operator /usr/local/bin/percona-xtradb-cluster-operator

RUN go get k8s.io/apimachinery/pkg/util/sets \
    && go build -o build/_output/bin/peer-list cmd/peer-list/main.go \
    && cp -r build/_output/bin/peer-list /usr/local/bin/

FROM redhat/ubi8-minimal

LABEL name="Percona XtraDB Cluster Operator" \
      vendor="Percona" \
      summary="Percona XtraDB Cluster is an active/active high availability and high scalability open source solution for MySQL clustering" \
      description="Percona XtraDB Cluster is a high availability solution that helps enterprises avoid downtime and outages and meet expected customer experience." \
      maintainer="Percona Development <info@percona.com>"

# Check https://access.redhat.com/errata/RHBA-2020:4536
# and https://bugzilla.redhat.com/show_bug.cgi?id=1832743
# for sources of enabling FIPS-140-2 support
RUN set -ex; \
    microdnf update; \
    microdnf -y install crypto-policies-scripts; \
    microdnf clean all; \
    fips-mode-setup --enable

COPY LICENSE /licenses/
COPY --from=go_builder /usr/local/bin/percona-xtradb-cluster-operator /usr/local/bin/percona-xtradb-cluster-operator
COPY --from=go_builder /usr/local/bin/peer-list /peer-list
COPY build/pxc-entrypoint.sh /pxc-entrypoint.sh
COPY build/pxc-init-entrypoint.sh /pxc-init-entrypoint.sh
COPY build/unsafe-bootstrap.sh /unsafe-bootstrap.sh
COPY build/pxc-configure-pxc.sh /pxc-configure-pxc.sh
COPY build/liveness-check.sh /liveness-check.sh
COPY build/readiness-check.sh /readiness-check.sh
COPY build/get-pxc-state /get-pxc-state

USER 2
