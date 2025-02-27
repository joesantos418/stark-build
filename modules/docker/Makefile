## #####################################
##
## Module: Docker
## This module handles docker image creation and publishing.

## Images to be created are listed by DOCKER_IMAGES variable,
## and defaults to all directories found directly under 'cmd'
## path.
##
## By default, a scratch image (see stark-build/docker/Dockerfile)
## will be used. If a custom Dockerfile is needed, a Dockerfile
## can be added (i.e 'cmd/mybinary/Dockerfile'). Then, the provided
## Dockerfile will be used instead

#
# TODO: Translate to english
# Esse módulo permite construir imagens docker e publicá-las.
# O módulo provê um arquivo `Dockerfile` próprio, porém é possível
# sobrescrever esse comportamento para binários específicos. Para
# utilizar um `Dockerfile` próprio, basta criar um `Dockerfile`
# dentro do diretório `cmd/<binário>`.
#
# Por exemplo, suponha 2 binários `api` e `worker` respectivamente
# em `cmd/api` e `cmd/worker`. Caso exista o arquivo
# `cmd/api/Dockerfile`, ao ser invocado o comando de construir a
# imagems, por exemplo, com `docker-images`, a imagem do binaŕio
# `api` usará `cmd/api/Dockerfile` mas a imagem do binário `worker`
# usará `<stark-build>/modules/docker/Dockerfile`.


## Docker Module Variables

## List of images the will be built or published. It defaults
## to cmd/ directory content.
DOCKER_IMAGES ?= $(shell ls ./cmd/ 2> /dev/null)

## Name of the project. This will compose the image's name like
## 'DOCKER_BASE_IMAGE/CMD:VERSION'. It defaults to the projets
## diretory name.
DOCKER_BASE_IMAGE ?= $(GO_PROJECT)

## GCR_BASE_URL is the googles registry prefix that will be
## pre-appended at the image name.
## You propably don't want to change this.
GCR_BASE_URL ?= gcr.io/$(GCR_PROJECT)

$(info [Stark Build] Initializing docker module...)
$(info [Stark Build]   DOCKER_IMAGES = $(DOCKER_IMAGES))
$(info [Stark Build]   DOCKER_BASE_IMAGE = $(DOCKER_BASE_IMAGE))
$(info [Stark Build]   GCR_PROJECT = $(GCR_PROJECT))
$(info [Stark Build]   GCR_BASE_URL = $(GCR_BASE_URL))

##
## Docker Module Targets

## Builds all images listed in DOCKER_IMAGES var.
.PHONY: docker-images
docker-images: $(foreach img,$(DOCKER_IMAGES),docker-image-$(img))

## Builds a tar file containing all images
out/docker_images.tar: docker-images require-DOCKER_BASE_IMAGE
	docker save --output $@ $(foreach img,$(DOCKER_IMAGES),$(DOCKER_BASE_IMAGE)/$(img):$(VERSION))

## Builds single docker image
.PHONY: docker-image-%
docker-image-%: require-DOCKER_BASE_IMAGE
	$(eval BINARY = $(@:docker-image-%=%))
	$(eval DOCKERFILE = $(shell [ -f cmd/$(BINARY)/Dockerfile ] && echo cmd/$(BINARY)/Dockerfile || echo $(STARK_BUILD_DIR)modules/docker/Dockerfile ) )
	docker build \
		--build-arg BINARY=out/bin/$(BINARY) \
		--file $(DOCKERFILE) \
		--pull \
		--tag '$(DOCKER_BASE_IMAGE)/$(BINARY):$(VERSION)' \
		.


## Publishes all docker images using VERSION as tag.
.PHONY: docker-publish
docker-publish: $(foreach img,$(DOCKER_IMAGES),docker-publish-version-$(img))

## Publishes all docker images using 'latest' as tag.
.PHONY: docker-publish-latest
docker-publish-latest: $(foreach img,$(DOCKER_IMAGES),docker-publish-latest-$(img))

## Publishes a single image using 'latest' as tag.
docker-publish-latest-%: require-DOCKER_BASE_IMAGE
ifndef GCR_PROJECT
	$(error Please set GCR_PROJECT variable)
endif
	$(eval IMAGE = $(@:docker-publish-latest-%=$(DOCKER_BASE_IMAGE)/%))
	$(eval LOCAL_TAG = $(IMAGE):$(VERSION))
	$(eval REMOTE_TAG = $(GCR_BASE_URL)/$(IMAGE):latest)
	docker tag $(LOCAL_TAG) $(REMOTE_TAG)
	docker push $(REMOTE_TAG)

## Publishes the generated images into Google's Registry using version as tag.
docker-publish-version-%: require-DOCKER_BASE_IMAGE
ifndef GCR_PROJECT
	$(error Please set GCR_PROJECT variable)
endif
	$(eval IMAGE = $(@:docker-publish-version-%=$(DOCKER_BASE_IMAGE)/%))
	$(eval LOCAL_TAG = $(IMAGE):$(VERSION))
	$(eval REMOTE_TAG = $(GCR_BASE_URL)/$(LOCAL_TAG))
	docker tag $(LOCAL_TAG) $(REMOTE_TAG)
	docker push $(REMOTE_TAG)


## Removes all local images matching the same base name.
.PHONY: docker-clean
docker-clean:
	docker images -a | grep $(DOCKER_BASE_IMAGE) | awk '{print $$3}' | xargs --no-run-if-empty docker rmi

$(info [Stark Build] Docker module initialized.)
