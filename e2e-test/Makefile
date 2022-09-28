CLUSTER_NAME := imago-test
KUBECTL := kubectl --context k3d-$(CLUSTER_NAME)

all: create load deploy trigger

create:
	k3d cluster list | grep -q '^$(CLUSTER_NAME)' || \
		k3d cluster $@ $(CLUSTER_NAME)

delete:
	if k3d cluster list | grep -q '^$(CLUSTER_NAME)'; then \
		k3d cluster $@ $(CLUSTER_NAME); \
	fi

load:
	# This is 1.21.5 at time of writing
	crane pull \
		nginx:1.21@sha256:ee89b00528ff4f02f2405e4ee221743ebc3f8e8dd0bfd5c4c20a2fa2aaa7ede3 \
		nginx.tar

	k3d image import -c $(CLUSTER_NAME) nginx.tar

	@rm -f nginx.tar

deploy:
	kustomize build . --load-restrictor LoadRestrictionsNone \
		| $(KUBECTL) apply -f -

trigger:
	-$(KUBECTL) delete job imago-run
	sleep 10
	$(KUBECTL) create job \
		--from=cronjob/imago imago-run
	sleep 10

	POD=$$($(KUBECTL) get pods -o name | grep 'imago-run'); \
	if ! $(KUBECTL) logs $$POD | grep -q 'nginx need to be updated from nginx:1.21.0 to'; then \
		$(KUBECTL) logs $$POD ; \
		exit 1; \
	fi
