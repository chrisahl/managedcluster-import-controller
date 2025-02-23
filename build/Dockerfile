# Copyright Contributors to the Open Cluster Management project

FROM registry.ci.openshift.org/open-cluster-management/builder:go1.15-linux-amd64 AS builder

ENV REMOTE_SOURCE='.'
ENV REMOTE_SOURCE_DIR='/remote-source'

COPY $REMOTE_SOURCE $REMOTE_SOURCE_DIR/app/
WORKDIR $REMOTE_SOURCE_DIR/app
RUN GOFLAGS="" go build ./cmd/manager
RUN GOFLAGS="" go test -covermode=atomic -coverpkg=github.com/open-cluster-management/managedcluster-import-controller/pkg/... -c -tags testrunmain ./cmd/manager -o manager-coverage

FROM registry.access.redhat.com/ubi8/ubi-minimal:latest
RUN microdnf update
ENV REMOTE_SOURCE_DIR='/remote-source'

ENV OPERATOR=/usr/local/bin/rcm-controller \
    USER_UID=1001 \
    USER_NAME=rcm-controller \
    KLUSTERLET_CRD_FILE=/usr/local/resources/agent.open-cluster-management.io_v1beta1_klusterlet_crd.yaml

# install operator binary
COPY --from=builder $REMOTE_SOURCE_DIR/app/manager ${OPERATOR}
COPY --from=builder $REMOTE_SOURCE_DIR/app/manager-coverage ${OPERATOR}-coverage
COPY --from=builder $REMOTE_SOURCE_DIR/app/build/bin /usr/local/bin
COPY --from=builder $REMOTE_SOURCE_DIR/app/build/resources /usr/local/resources

RUN  /usr/local/bin/user_setup

ENTRYPOINT ["/usr/local/bin/entrypoint"]

USER ${USER_UID}
