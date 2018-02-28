tag = gcr.io/nginsako/backend:latest

build:
	docker build -t $(tag) .


gpush:
	gcloud docker -- push $(tag)

build-clean:
	docker build --no-cache -t $(tag) .

push:
	docker push $(tag)

test:
	echo "Tests not yet implemented"
