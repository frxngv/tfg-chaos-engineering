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
2. Successful execution of the lesson directly with Python and inside Docker.
3. The Python version, Docker image ID, and observed HTTP responses.
4. Later, resource measurements for a minimal Kubernetes deployment.

No container build, fault injection, availability measurement, or recovery-time
measurement has been completed at this stage.

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
The Dockerfile has been prepared but its build and container execution remain
unverified because the Docker Engine was not reachable during inspection.
The user's step-by-step execution of the lesson is still pending.

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

The next user check is whether a Python interpreter is available in PowerShell,
followed by direct execution of the lesson application.
