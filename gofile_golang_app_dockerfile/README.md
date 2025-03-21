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

## NOTEBOOKLM SUMMARY OF THE STREAM

This YouTube video transcript documents a detailed review of the Dockerfile used by the "gofile" project, a command-line interface (CLI) tool created by Felipe Cruz to build container images for Go applications without relying on traditional Dockerfiles. The review is hosted by Ivan and Kyle from the "Cloud Native Container Craft" YouTube channel, with Felipe joining them to discuss his project and its Dockerfile.


### Introduction to Gofile and its Motivation:

* Felipe explains that gofile was developed as a learning experience to explore BuildKit features and alternative ways to package Go applications without Dockerfiles.
* He acknowledges the power and usefulness of Dockerfiles but notes that crafting lightweight and slim images with them can be challenging, especially for beginners. This difficulty applies across various technologies, not just Go.
* The gofile project aims to abstract this complexity by allowing developers to define image specifications using a different syntax (YAML) that is then translated into BuildKit's lower-level instructions.
* Felipe mentions that gofile is similar to another project called "goer file" but includes a few minor enhancements for his learning purposes.


### Gofile's Alternative Syntax:

* The discussion highlights an example of gofile's YAML-based syntax, which includes fields like repolpath, ref, and scratch.
* This YAML file is an alternative to a Dockerfile and represents what BuildKit terms a "frontend," a different way to express container image building instructions.
* The specific fields in the YAML are arbitrary and can be customized.
* A crucial line in the YAML file (line 14 in the example) acts as the "brain" that interprets these fields and converts them into a low-level build (LLB) format that BuildKit can understand and execute.
* The reviewers and Felipe compare this YAML syntax to a more traditional Dockerfile that would achieve a similar outcome of building a Go application from a GitHub repository and packaging it into a minimal "from scratch" container. This comparison demonstrates the conciseness and higher-level nature of gofile's approach.
* Ultimately, gofile is a CLI tool that acts as a client for the BuildKit API. Distributing such CLI tools often involves containerizing them using Docker, which leads to the review of gofile's own Dockerfile.

### Review of Gofile's Dockerfile - High-Level Structure:

* Felipe's Dockerfile for gofile is identified as a multi-stage Dockerfile.
* It consists of two distinct FROM statements, defining two separate build stages.
◦ The Build Stage: This stage utilizes a standard Golang image that includes all the necessary developer tools to compile the gofile binary.
◦ The Runtime Stage: This stage is based on a minimal Alpine Linux image and is responsible for shipping the compiled binary along with essential runtime dependencies.


### Detailed Examination of the Build Stage:

•The build stage starts by running go mod download to fetch all the Go packages defined in the go.mod file.
* The reviewers highlight the use of the RUN --mount=type=cache instruction. This leverages BuildKit's cache mount feature to store downloaded Go packages within the build container. This significantly speeds up subsequent builds by reusing cached dependencies, as only new or changed dependencies will need to be downloaded.
* Two bind mounts are used with the --mount=type=bind flag for go.sum and go.mod files. This allows the BuildKit container to access these files directly from the host context without copying them into the image layers. This is beneficial because these files are only needed during the dependency resolution phase and are not part of the final build artifact, thus keeping the image leaner.
* The discussion clarifies that after the go mod download operation, the bind-mounted go.sum and go.mod files do not persist in the container image layer.


### Transition to the Runtime Stage and Base Image Choice:

* The conversation then shifts to why a multi-stage build is crucial. The Go compiler image used in the build stage is large (around 830MB compressed, potentially 1GB uncompressed) and contains many development tools and dependencies that are not required to run the final gofile binary.
* The goal is to create a final container image that only includes the compiled gofile binary and the absolute necessary runtime dependencies, resulting in a much smaller and more secure image.
* The runtime stage begins with FROM alpine:latest. Alpine Linux is chosen as the base image for its small size (around 5MB) and the availability of a package manager (apk).
* The apk package manager is used to install ca-certificates and tzdata. ca-certificates are necessary for the gofile tool to make secure HTTPS requests, while tzdata provides time zone information if needed. The --no-cache flag is used with apk add to prevent the package index from being stored in the final image, further reducing its size.
* The use of RUN --mount=type=cache is also applied to the apk add instruction to cache the downloaded packages, similar to how Go dependencies are cached in the build stage.
* The reviewers and Felipe discuss alternative base images:
◦ Scratch: While the smallest possible base, it's not suitable here without explicitly adding CA certificates, as a statically linked Go binary accessing HTTPS endpoints would fail without them.
◦ Distroless Images: Images from gcr.io/distroless are also very minimal and often include CA certificates and time zone data but lack a shell and package manager. This makes adding users or performing other system-level tasks more complex, requiring manual manipulation of files like /etc/passwd. The trade-off is between a smaller, more secure image and ease of use and debugging.
* The discussion about Alpine touches on its use of musl libc instead of the more common glibc. While this can sometimes cause compatibility issues, it's not a concern for this statically linked Go application. BusyBox is also briefly mentioned as another small Linux distribution, but it may not include certificates by default.
* The choice of Alpine in this case is deemed a pragmatic and user-experience-friendly decision, balancing small size with the convenience of a package manager and pre-installed certificates.


### Copying the Binary and Final Configuration:

* The compiled gofile binary is transferred from the build stage to the runtime stage using the COPY --from=build /bin/gofile /bin/ instruction. The --from=build flag specifies the source stage for the copy operation. This demonstrates the ability to copy artifacts between different stages of a multi-stage build and even from other images.
* A non-root user named gofile is added using RUN adduser -D gofile for security reasons. Running containers as non-root users minimizes the potential damage if the container is compromised.
* The USER gofile instruction sets the subsequent commands and the container's main process to run as the gofile user.
* Finally, ENTRYPOINT ["/bin/gofile"] defines the command that will be executed when the container starts, which is running the gofile binary.


### Pinning Base Image Versions:

* The reviewers and Felipe strongly advise against using the latest tag for base images in production environments. The latest tag can be updated with potentially breaking changes.
* The recommended best practice is to pin to a specific index digest of the base image for maximum reproducibility and to ensure compatibility across different architectures.
* For personal projects, using alpine:latest might be acceptable due to the low likelihood of breaking changes for a simple, statically linked Go application.
* For larger, more complex applications, pinning to a specific major, minor, or even patch version (e.g., debian:11.9.19) offers a trade-off between stability and receiving security updates. Pinning to just the major version (e.g., debian:11) allows for automatic updates of minor and patch versions, including security fixes, within that major release.
* Completely pinning to a specific digest ensures immutability but requires manual updates to receive even security patches.


### Building Go Applications Without Dockerfiles (Revisited):

* The conversation returns to the benefits of tools like gofile, KO, and Mint Toolkit for building container images for Go applications without the complexities of writing and maintaining Dockerfiles.
* These tools offer a more streamlined and often opinionated approach, especially useful for developers who are not Docker experts.
* Gofile's YAML syntax is presented as a higher-level abstraction that simplifies the definition of build processes compared to a detailed Dockerfile. A complex Dockerfile with many instructions can often be represented by just a few lines in gofile's syntax.
* The trade-off of using such tools is the potential loss of fine-grained control over the build process. Customizations beyond the tool's capabilities might be difficult or require extending the tool itself.
* These tools can be particularly valuable for organizations that want to enforce a consistent and simplified containerization process for their Go applications.


### Debugging Containers Without a Shell (Further Discussion):

* The lack of a shell in minimal base images like distroless can make debugging challenging.
* However, various tools exist to facilitate debugging in such environments, including cdebug (Docker Desktop), CD Bug, Minikube's debugging features, and Kubectl debug for Kubernetes.
* The awareness and capabilities of these debugging tools are growing, making shell-less containers more practical for production deployments.
* Features like ephemeral containers in Kubernetes are also helping to address the debugging challenges of containers without shells.


### Purpose of the "Good Dockerfiles" Series:

* The hosts emphasize that the "Good Dockerfiles" series aims to showcase effective Dockerfile practices, discuss different options and their trade-offs, and educate viewers on how to build better container images. The goal is not to criticize specific Dockerfiles but to learn from them and highlight good patterns that others can adopt.
* The episode concludes with recommendations to explore the gooddockerfiles.com website and specific tutorials on building smaller container images, building from scratch, and best practices for containerizing Go applications.


This summary covers the key aspects of the YouTube stream, from the introduction of the gofile project and its Dockerfile to the detailed analysis of its structure, base image considerations, version pinning, alternative build tools, and debugging strategies. It also highlights the purpose and goals of the "Good Dockerfiles" series.
