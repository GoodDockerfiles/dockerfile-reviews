# Ivan and Kyle Are reviewing a Golang app Dockerfile for "gofile" with Felipe Cruz

[![Ivan and Kyle Are reviewing a Golang app Dockerfile for "gofile" with Felipe Cruz](/images/gofile_golang_app_dockerfile_review.png)](https://www.youtube.com/watch?v=v5E-enT_pyo)

YouTube Stream: https://www.youtube.com/watch?v=v5E-enT_pyo

## OVERVIEW

We reviewed the "gofile" ( https://github.com/felipecruz91/gofile ) application Felipe created and its Dockerfile ( https://github.com/felipecruz91/gofile/blob/main/Dockerfile ). The existing Dockerfile is interesting for a number of reasons. It has a number of good qualities we'll call out and discuss in addition to discussing what kind of improvements to make and what kind of gotchas to watch out for.

"gofile" is a statically compiled application and it uses Alpine for its container base image. We explored what you should be aware of working with compiled applications and using Alpine.

Finally, we also talked about "gofile" itself, what it does and building simple container images without Dockerfiles.

## REFERENCES

If you need to containerize a Golang-based application the following tutorials will be useful to you:

* `Building Container Images FROM Scratch: 6 Pitfalls That Are Often Overlooked` - https://labs.iximiuz.com/tutorials/pitfalls-of-from-scratch-images
* `How to Build Smaller Container Images: Docker Multi-Stage Builds` - https://labs.iximiuz.com/tutorials/docker-multi-stage-builds
* `Build a Production-Ready Go Container Image: A Dynamically Linked Application` - https://labs.iximiuz.com/challenges/dockerize-golang-application-dynamic-linking
* `What's Inside Distroless Container Images: Taking a Closer Look` - https://labs.iximiuz.com/tutorials/gcr-distroless-container-images

If you want to build your own Dockerfile alternative with BuildKit take a look at these resources:

* https://mattrickard.com/building-a-new-dockerfile-frontend
* https://github.com/moby/buildkit/blob/master/docs/dev/dockerfile-llb.md

If you want to containerize a Golang app without using Dockerfiles you have a number of options:

* `gofile` - https://github.com/felipecruz91/gofile 
* `mint` (MinToolkit aka DockerSlim) and its `imagebuild` command (example: `mint imagebuild --engine simple --exe-path ./local/path/to/appexe --image-name my-app-container:latest --runtime-load docker` ) - https://github.com/mintoolkit/mint
* `ko` - https://github.com/ko-build/ko

## DOCKERFILES

* [Distroless-based Dockerfile](./Dockerfile.distroless)
* [Distroless-based Dockerfile with a Shell](Dockerfile.distroless.shell)
* [Original Dockerfile](./Dockerfile.original)

[![Distroless-based Dockerfile with a Shell](/images/gofile_golang_app_dockerfile_distroless.png)](Dockerfile.distroless.shell)
