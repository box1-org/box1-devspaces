# box1-devspaces

The Box1 Universal Developer Image (UDI) for OpenShift Dev Spaces, and the helper
commands that Box1 devfiles call. Tracking issue: box1-org/box1-docs#114.

## What's in the image

`udi/Containerfile` extends the UDI that ships with Dev Spaces 3.29.1
(`registry.redhat.io/devspaces/udi-rhel9`, pinned by digest).

| Already in the base image | Added by this image |
| --- | --- |
| `oc`, `kubectl` (includes kustomize), `tkn`, `tkn pac`, Maven, JDK 8/11/17/21, Node 22, podman/buildah, git, jq, yq | JDK 21 set as the default, JBang, Camel JBang (`camel`), the Claude Code CLI, the GitHub CLI (`gh`), and the helpers below |

Every version is pinned. The JBang and `gh` archives are also checked against their sha256.
`gh` starts logged out: run `gh auth login` in the workspace, or export `GH_TOKEN`.
Run `cat /etc/box1-udi-release` in a workspace to see the versions.

## Helpers (`udi/bin`)

| Command | What it does |
| --- | --- |
| `box1-pipeline validate\|logs\|list` | Resolves `.tekton/pull-request.yaml` for the **pushed** HEAD and starts it in `box1-ci`, then follows or lists its runs. Releases aren't started from here; they run from Pipelines as Code on push to `main`. |
| `box1-remote-dev deploy\|sync\|url\|delete` | Quarkus remote dev. Builds a mutable-jar image from the working tree, pushes it to your sandbox namespace (`<user>-sandbox`), applies the repository's `deploy/remote-dev`, then live-syncs your changes. |
| `box1-debug sandbox\|dev [service]` | Port-forwards JDWP (`127.0.0.1:5005` in the pod) to `localhost:5005` so the VS Code Java debugger can attach. |
| `box1-podman-socket start\|status\|stop` | Runs podman's Docker-compatible API at `/tmp/podman-run/podman.sock`. The image points `DOCKER_HOST` there and sets `TESTCONTAINERS_RYUK_DISABLED=true`, so Testcontainers (`-Pcontainers`) works. Devfiles start it in a postStart event. Needs container run enabled in the CheCluster. |

None of these helpers deploy to `<service>-dev`. Argo CD stays the only deployer there.

## Build and release

- Pull requests run `.tekton/pull-request.yaml`, which builds the image without pushing it. It runs as `ci-udi-check`.
- A push to `main` runs `.tekton/release.yaml`, which pushes `registry.apps.ocp.box1.org/box1-devtools/box1-udi:<sha12>` and prints the digest. It runs as `ci-udi-release`.
- Devfiles pin the image by digest, as `image-registry.openshift-image-registry.svc:5000/box1-devtools/box1-udi@sha256:…`. Moving to a new image means a reviewed digest bump in each repository that uses it.
- When Dev Spaces is upgraded, bump `BASE_IMAGE`. The command for finding the new digest is in the Containerfile header.

The Pipelines as Code `Repository`, service accounts, the `box1-devtools` image
project and pull access for workspaces are all defined in
`box1-gitops/platform/pipelines/ci/box1-devspaces.yaml`.
