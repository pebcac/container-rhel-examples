GO_BUILD = github.com/golang/example/outyet
CONTEXT = acme
VERSION = v3.2
IMAGE_NAME = starter-scratch
REGISTRY = docker-registry.default.svc.cluster.local:5000
OC_USER=developer
OC_PASS=developer

# Allow user to pass in OS build options
ifeq ($(TARGET),centos7)
	BUILDER_IMAGE := docker.io/centos:7
else
	BUILDER_IMAGE := registry.access.redhat.com/rhel7
endif

all: build
build:
	chmod u+x build.sh
	./build.sh ${GO_BUILD} ${BUILDER_IMAGE}
	docker build --pull -t ${CONTEXT}/${IMAGE_NAME}:${VERSION} -t ${CONTEXT}/${IMAGE_NAME} .
	@if docker images ${CONTEXT}/${IMAGE_NAME}:${VERSION}; then touch build; fi

lint:
	dockerfile_lint -f Dockerfile

test:
	$(eval CONTAINERID=$(shell docker run -tdi -u $(shell shuf -i 1000010000-1000020000 -n 1) -p 8080:8080 \
	--cap-drop=KILL \
	--cap-drop=MKNOD \
	--cap-drop=SYS_CHROOT \
	--cap-drop=SETUID \
	--cap-drop=SETGID \
	${CONTEXT}/${IMAGE_NAME}:${VERSION}))
	@sleep 3
	@curl localhost:8080
	@docker rm -f ${CONTAINERID}

openshift-test:
	$(eval PROJ_RANDOM=test-$(shell shuf -i 100000-999999 -n 1))
	oc login -u ${OC_USER} -p ${OC_PASS}
	oc new-project ${PROJ_RANDOM}
	docker login -u ${OC_USER} -p ${OC_PASS} ${REGISTRY}
	docker tag ${CONTEXT}/${IMAGE_NAME}:${VERSION} ${REGISTRY}/${PROJ_RANDOM}/${IMAGE_NAME}
	docker push ${REGISTRY}/${PROJ_RANDOM}/${IMAGE_NAME}
	oc new-app -i ${IMAGE_NAME}
	oc rollout status -w dc/${IMAGE_NAME}
	oc status
	sleep 5
	oc describe pod `oc get pod --template '{{(index .items 0).metadata.name }}'`
	curl `oc get svc/${IMAGE_NAME} --template '{{.spec.clusterIP}}:{{index .spec.ports 0 "port"}}'`

run:
	docker run -tdi -u $(shell shuf -i 1000010000-1000020000 -n 1) -p 8080:8080 \
	--cap-drop=KILL \
	--cap-drop=MKNOD \
	--cap-drop=SYS_CHROOT \
	--cap-drop=SETUID \
	--cap-drop=SETGID \
	${CONTEXT}/${IMAGE_NAME}:${VERSION}

clean:
	rm -f help.1 main build
