project  := graphql-engine
registry := hasura
version  := $(shell ../scripts/get-version.sh)

stack_resolver := lts-11.15
packager_ver := 1.1
project_dir := $(shell pwd)
build_dir := $(project_dir)/$(shell stack --system-ghc path --dist-dir)/build

build_output := /build/_server_output

image: $(project).cabal
	docker build -t "$(registry)/$(project):$(version)" \
		-f packaging/Dockerfile \
		--build-arg build_flags=--fast \
		--build-arg project=$(project) \
		--build-arg stack_resolver=$(stack_resolver) \
		--build-arg packager_version=$(packager_ver) \
		.

release-image: $(project).cabal
	docker build -t "$(registry)/$(project):$(version)" \
		-f packaging/Dockerfile \
		--build-arg project=$(project) \
		--build-arg stack_resolver=$(stack_resolver) \
		--build-arg packager_version=$(packager_ver) \
		.

# assumes this is built in circleci
ci-binary-and-test: $(project).cabal
	mkdir -p packaging/build/rootfs
	stack test --coverage --test-arguments --database-url=$(DATABASE_URL)
	mkdir -p $(build_output)
	cp $(build_dir)/$(project)/$(project) $(build_output)
	echo "$(version)" > $(build_output)/version.txt

# assumes this is built in circleci
ci-image:
	docker create -v /root/ --name dummy alpine:3.4 /bin/true
	docker cp $(build_dir)/$(project)/$(project) dummy:/root/
	docker run --rm --volumes-from dummy $(registry)/graphql-engine-packager:$(packager_ver) /build.sh $(project) | tar -x -C packaging/build/rootfs
	strip --strip-unneeded packaging/build/rootfs/bin/$(project)
	upx packaging/build/rootfs/bin/$(project)
	docker build -t $(registry)/$(project):$(version) packaging/build/

ci-save-image:
	docker save -o $(build_output)/image.tar $(registry)/$(project):$(version)

ci-load-image:
	docker load -i $(build_output)/image.tar

push:
	docker push $(registry)/$(project):$(version)

push-latest:
	docker tag $(registry)/$(project):$(version) $(registry)/$(project):latest
	docker push $(registry)/$(project):latest

packager: packaging/packager.df
	docker build -t "$(registry)/graphql-engine-packager:$(packager_ver)" -f packaging/packager.df ./packaging/

.PHONY: image release-image push packager ci-binary-and-test ci-image ci-save-image ci-load-image
