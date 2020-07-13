MAKEFLAGS += --warn-undefined-variables
IMAGE ?= cesanta/docker_auth
COMPRESS_BINARY ?= false
CA_BUNDLE = /etc/ssl/certs/ca-certificates.crt
VERSION = $(shell cat version.txt)

BUILDER_IMAGE ?= golang:1.13-alpine

.PHONY: %

all: build

deps:
	go get -v github.com/a-urth/go-bindata/go-bindata

python-deps:
	pip install GitPython

generate:
	go generate -v . \
	  ./authn/... \
	  ./authz/... \
	  ./mgo_session/... \
	  ./server/...

build: deps python-deps
	CGO_ENABLED=0 go build -v --ldflags=--s -mod=vendor

ca-certificates.crt:
	cp $(CA_BUNDLE) .

build-release: ca-certificates.crt
	docker run --rm -v $(PWD)/..:/src \
	  $(BUILDER_IMAGE) sh -x -c \
			"apk update && \
			apk add git make py3-pip && \
	    cd /src/auth_server && \
	    umask 0 && \
			make build"
	@echo === Built version $$(cat version.txt) ===

auth_server:
	@echo
	@echo Use build or build-release to produce the auth_server binary
	@echo
	@exit 1

docker-build:
	docker build -t $(IMAGE):latest .
	docker tag $(IMAGE):latest $(IMAGE):$(VERSION)

docker-tag-%:
	docker tag $(IMAGE):latest $(IMAGE):$*

docker-push:
	docker push $(IMAGE):latest
	docker push $(IMAGE):$(VERSION)

docker-push-%: docker-tag-%
	docker push $(IMAGE):$*

clean:
	rm -rf auth_server
