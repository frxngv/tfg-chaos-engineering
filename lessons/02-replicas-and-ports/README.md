# Lesson 02: Run two replicas and understand ports

## Goal

Create two simultaneous instances of the Lesson 01 application from its existing
Docker image. Understand host ports, container ports, detached execution, and what
happens to each endpoint when one instance stops.

Read each section before executing its commands. This guide describes expected
behavior; execution results will be recorded in the [study log](../../docs/study-log.md)
after they are observed.

## 1. Inspect the starting state

Open PowerShell and run:

```powershell
docker ps -a --filter name=tfg-hello
docker ps -a --filter name=tfg-replica-
docker image inspect tfg-hello:lesson-01 --format '{{.Id}}'
```

If the previous lesson's `tfg-hello` container is still running, stop it with
`docker stop tfg-hello`. If earlier attempts left containers named
`tfg-replica-a` or `tfg-replica-b`, inspect their state before repeating this lesson.
Do not stop unrelated containers. If the image is missing, return to Lesson 01
and build it there; no new Dockerfile is needed for this lesson.

`Clear-Host` (or `clear`) only clears the terminal display. It does not stop a
container, delete an image, change the current directory, or erase command history.

## 2. Understand the port mapping

Both application processes listen on port 8000 inside their own containers.
Each container has its own network environment, so their internal ports can be
the same. The ports published on the host must be different for these two
containers to use the same host IP and TCP protocol simultaneously.

| Browser address on the host | Container | Application port inside the container |
| --- | --- | --- |
| `http://localhost:8001/` | `tfg-replica-a` | 8000 |
| `http://localhost:8002/` | `tfg-replica-b` | 8000 |

The publishing format used here is `HOST_IP:HOST_PORT:CONTAINER_PORT`.
`127.0.0.1` restricts the published endpoint to the host's loopback interface.
Changing the published host port does not change the port the application listens
on inside its container.

## 3. Start replica A

Run this command from any directory; Docker uses the existing local image:

```powershell
docker run -d --rm --name tfg-replica-a -p 127.0.0.1:8001:8000 tfg-hello:lesson-01
```

The new option `-d` means detached mode: Docker starts the container in the
background and prints its container ID. PowerShell becomes available for the
next command. Closing this terminal does not stop a detached container.
The application still runs as the container's main process; the container stops
when that process exits. Use `docker logs tfg-replica-a` to read its output.

Visit <http://localhost:8001/> and record the instance hostname in the JSON.

## 4. Start replica B

```powershell
docker run -d --rm --name tfg-replica-b -p 127.0.0.1:8002:8000 tfg-hello:lesson-01
```

Visit <http://localhost:8002/>. Expect the same message and a different instance
hostname. Both instances use the same application image.

If Docker reports that a host port is already allocated, inspect the existing
listeners or container mappings before proceeding. Do not change the internal
port to 8001 or 8002: this application is still listening on 8000.

## 5. Inspect both replicas

```powershell
docker ps --filter name=tfg-replica-
docker inspect tfg-replica-a tfg-replica-b --format '{{.Name}} {{.Image}}'
docker logs tfg-replica-a
docker logs tfg-replica-b
```

Expect two running containers with different names, IDs, and published host ports.
The image IDs printed by `docker inspect` should match. After visiting both
addresses, each application's logs should show a successful request.

The name passed with `--name` identifies a container for Docker commands. The
hostname returned by this application defaults to a value derived from the
container ID; it does not automatically equal the `--name` value.

## 6. Stop one replica and check both endpoints

```powershell
docker stop tfg-replica-a
```

Make fresh HTTP requests from PowerShell so browser display caching does not
confuse this check:

```powershell
Invoke-RestMethod -Uri http://localhost:8001/ -TimeoutSec 3
Invoke-RestMethod -Uri http://localhost:8002/ -TimeoutSec 3
```

The first command should report a connection failure because replica A stopped.
The second should still return JSON from replica B. Record the actual outputs.
The stopped replica should also disappear from `docker ps -a` because of `--rm`.

These separate addresses do not provide automatic traffic redistribution. A
client using port 8001 does not switch to port 8002 by itself. This exercise
demonstrates independent instances; a shared entry point with routing and health
checks is a further step toward maintaining service during an instance failure.

This is a controlled manual stop, not yet an automated chaos experiment or a
measurement of availability or recovery time.

## 7. Replace the stopped replica manually

```powershell
docker run -d --rm --name tfg-replica-a -p 127.0.0.1:8001:8000 tfg-hello:lesson-01
```

Request port 8001 again. Expect the application to respond with a new instance
hostname. Creating this replacement required an explicit command: nothing in
this setup automatically maintains the desired number of replicas.

## 8. Clean up

```powershell
docker stop tfg-replica-a tfg-replica-b
docker ps -a --filter name=tfg-replica-
docker image ls tfg-hello
```

Expect no matching containers and an application image that is still available.

## Record and explain

Record both image IDs, both initial hostnames, the container port mappings,
the two HTTP outcomes after stopping A, the replacement hostname, and cleanup.
Explain why both containers can listen on 8000, why their published host ports
differ, what `-d` changes, and why two replicas alone did not redirect requests
when one stopped. All observed outcomes remain pending until actually checked.
