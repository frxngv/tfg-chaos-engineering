# Feasibility study log

## Initial environment inspection - 2026-11-24

Read-only checks were performed by the assistant from the local Codex session.
These are preliminary environment checks, not Kubernetes workload measurements.

| Item | Observation |
| --- | --- |
| Operating system | Windows |
| CPU | AMD Ryzen 7 5800H with Radeon Graphics |
| Logical processors visible to the session | 16 |
| Physical memory visible to Windows | Approximately 15.35 GiB |
| Free space on C: at the initial inspection | Approximately 115 GiB |
| Docker CLI | Installed |
| Docker Engine | The CLI could not connect to the Docker Desktop Linux engine pipe |
| Kubernetes tools | `kubectl` and Kind were found on PATH |
| WSL inspection | `wsl --status` returned access denied from the assistant session |
| Kubernetes workload validation | Not performed |

The Docker connection error does not establish its cause. Docker Desktop may not
be running; its state must be checked before attempting a container build.
The WSL result is also limited to the assistant session and does not establish
that WSL is broken on the laptop.

## Initial decisions

- Begin with a small Python and Docker exercise to understand image construction,
  container execution, and port publishing.
- Keep the learning application separate from the eventual benchmark application.
- Choose the final scope after measuring a real local deployment.
- Treat memory as a constraint to investigate. Current hardware observations
  do not demonstrate that the complete proposed tool stack will fit.

## Next evidence to collect

1. Docker client/server connectivity and a first container run: completed; see
   the user verification below.
2. Direct Python execution, application image build, container execution, and
   lifecycle checks: completed; see the Lesson 01 user verification below.
3. The resulting application image ID and container recreation: completed; see
   the recreation check below. The user's Python version and HTTP behavior are
   also recorded.
4. Later, resource measurements for a minimal Kubernetes deployment.

At the initial inspection, no container build had been completed. Fault
injection, availability measurement, and recovery-time measurement remain pending.

## Lesson 01: assistant verification - 2026-11-24

The assistant started `app.py` directly using the Codex-bundled Python 3.12.14
interpreter, sent HTTP requests, and stopped the test process afterwards.

| Check | Observed result |
| --- | --- |
| Application starts as a Python script | Passed |
| `GET /` | HTTP 200; valid JSON with the expected message and a nonempty instance name |
| `GET /missing` | HTTP 404 |
| Test process cleanup | Process stopped after verification |

The first HTTP check was blocked by the assistant's restricted execution
environment. The successful rerun used approved execution outside that sandbox.
This does not establish whether the user's `python` command is configured.
At the time of this assistant check, the Dockerfile build and container execution
were unverified because the Docker Engine was not reachable during inspection.
The later user verification is recorded below.

## Docker: user verification - 2026-11-24

The user executed the following commands in Windows PowerShell and shared
screenshots of their output. These observations come from the user's terminal,
not from a rerun by the assistant.

### Client and server connection

```powershell
docker version
```

| Component | Observed result |
| --- | --- |
| Docker client | Version 29.9.1; windows/amd64 |
| Docker Desktop | Version 4.96.0 |
| Docker Engine | Version 29.9.1; linux/amd64 |
| Connection | Both client and server information returned successfully |

The engine was reachable at this point, unlike during the initial inspection.
The screenshots do not establish what changed between those checks.

### First container execution

```powershell
docker run --rm hello-world
```

The image was not available locally. Docker downloaded `hello-world:latest`
from Docker Hub and the container printed `Hello from Docker!`. The command
then returned to the PowerShell prompt without a reported error.

This verifies registry access for that pull and basic image download and
container execution. It does not yet verify building our application image,
publishing a web port, Kubernetes, or the capacity of the laptop for the full
experiment stack. The `latest` tag is mutable; no fixed image digest is recorded
for this exercise.

The next checks at this point were Python availability and direct execution of
the lesson application; their results are recorded below.

## Lesson 01: user verification - 2026-11-24

The user followed the exercise step by step. Screenshots show the Python version,
direct-run HTTP logs, successful image build, container JSON response, container
listing, and container logs. The user subsequently confirmed the expected stop
and image-persistence results in chat.

| Check | Observed result | Evidence |
| --- | --- | --- |
| `python --version` | Python 3.14.4 available in PowerShell | User screenshot |
| Direct Python execution | Server listening on port 8000; `GET /` returned HTTP 200 | User screenshot |
| Browser icon request | `GET /favicon.ico` returned HTTP 404, consistent with the implemented routes | User screenshot |
| `docker build -t tfg-hello:lesson-01 .` | Build finished successfully using `python:3.12-slim` | User screenshot |
| Container HTTP response | Expected message and instance hostname `eec264b69abd` | User screenshot |
| `docker ps --filter name=tfg-hello` | Container running from `tfg-hello:lesson-01`, command `python -u app.py`, port mapping `127.0.0.1:8000->8000/tcp` | User screenshot |
| `docker logs tfg-hello` | Startup message and `GET /` with HTTP 200 | User screenshot |
| `docker stop tfg-hello`, then filtered `docker ps -a` | Container stopped and no matching container remained, as expected with `--rm` | User confirmation in chat |
| `docker image ls tfg-hello` | Application image remained after the container was removed | User confirmation in chat |

The observed container response was:

```json
{"message": "Hello from the TFG application!", "instance": "eec264b69abd"}
```

The local Python interpreter was version 3.14.4, whereas the Dockerfile selects
the separate Python 3.12 image family. These checks demonstrate basic execution
in both environments; they are not a general cross-version compatibility study.
The full image ID was subsequently recorded in the recreation check below. The
exact Python patch version inside the image has not been collected. Native-run
hostname and raw JSON were not captured in the evidence.

### Findings and next step

- The laptop can build and run this small application image and publish its HTTP
  port locally.
- The running application's logs can be inspected using Docker.
- Removing the container leaves the image available for another run.
- This result does not establish Kubernetes or microservices benchmark capacity.
- The recreation check below completes the first lesson's execution steps.
  Multiple simultaneous replicas and traffic distribution remain untested.

### Image identity and container recreation - 2026-11-24

After being instructed to inspect the image and create another container from
the existing tag without rebuilding it, the user supplied this image ID in chat:

```text
sha256:ed597ae786096700a3a8531ee18552791327dea6bcebc0ad4bfe536467f25edb
```

The user also shared a browser screenshot showing this response:

```json
{"message": "Hello from the TFG application!", "instance": "3aee0cb33eea"}
```

The message is unchanged and the instance hostname differs from the first
container's `eec264b69abd`. This is consistent with creating a new container
from the existing application image. No image ID was captured before the first
run, so this is not a before/after comparison of image IDs.

The image ID identifies the local application image content. It is separate
from both the container hostname and a registry repository digest.

Lesson 01 execution is complete. The final shutdown of the recreated container
has not yet been confirmed. Next: stop that container, then study simultaneous
replicas and the distinction between host ports and container ports.

## Lesson 02: planned exercise - 2026-11-24

The [second lesson](../lessons/02-replicas-and-ports/README.md) has been prepared
for guided execution. It reuses the existing image and introduces two detached
containers with separate published host ports, inspection of their image IDs,
a manual stop of one replica, independent endpoint checks, manual replacement,
and cleanup.

At preparation time, all execution results were pending. The user subsequently
performed the checks recorded below. This guide does not configure a shared
entry point, automatic traffic redistribution, or automatic replica replacement.
The assistant has not run the lesson's containers on the user's behalf.

## Lesson 02: user verification - 2026-11-24

The user shared screenshots of the commands and responses while following the
exercise. The following are observed results, not expected output copied from
the guide.

### Initial replicas and image identity

The starting `docker ps` output showed no running containers. The user then
created two containers with `docker run -d --rm`, using `tfg-hello:lesson-01`:

| Container | Initial hostname | Published host endpoint | Internal application port |
| --- | --- | --- | --- |
| `tfg-replica-a` | `3e2c39b77530` | `127.0.0.1:8001` | 8000/TCP |
| `tfg-replica-b` | `e66b9c1c39d0` | `127.0.0.1:8002` | 8000/TCP |

Both browser endpoints returned the application message with their respective
hostnames. The container listing showed both instances running simultaneously.

The user checked their image identities with:

```powershell
docker inspect --format '{{.Name}} -> {{.Image}}' tfg-replica-a tfg-replica-b
```

Both containers referenced the same image ID, also matching the Lesson 01 value:

```text
sha256:ed597ae786096700a3a8531ee18552791327dea6bcebc0ad4bfe536467f25edb
```

### Manual stop and independent endpoint checks

The user ran `docker stop tfg-replica-a`, which returned the container name, then
made fresh requests using `Invoke-RestMethod` with a three-second timeout.

| Request after stopping A | Observed result |
| --- | --- |
| `http://localhost:8001/` | PowerShell reported that the operation exceeded its timeout |
| `http://localhost:8002/` | Application message returned with instance `e66b9c1c39d0` |

The timeout was a client-side request failure, not an HTTP error response from
the application. B remained reachable at its separate endpoint. Requests to A's
published port were not automatically redirected to B.

### Manual replacement

The user explicitly created another container named `tfg-replica-a` from the
existing image tag, again publishing host port 8001 to container port 8000.
The subsequent request to port 8001 returned:

```json
{"message": "Hello from the TFG application!", "instance": "994d5d2b84e0"}
```

The new hostname differed from the original A. A subsequent `docker ps` output
showed the new A and the original B running simultaneously with their expected
port mappings. The image ID of the replacement A was not separately inspected.

### Findings and remaining checks

- Two instances of this small application ran simultaneously on the laptop.
- Both instances used the same image while exposing different host ports.
- During the observed manual stop of A, B continued serving a direct request.
- An explicit creation command restored access through A's host port.
- There was no automatic failover or controller maintaining a replica count.
- This was a controlled manual stop and manual recovery, not an automated chaos
  experiment. Availability and recovery time have not been measured.
- Cleanup of the two running replicas has not yet been confirmed. Their separate
  application logs were not supplied as part of the Lesson 02 evidence.
- Kubernetes capacity and behavior remain to be evaluated in the next stage.
