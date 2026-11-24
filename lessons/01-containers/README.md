# Lesson 01: Run a Python application with Docker

## Goal

Understand the difference between source code, a Dockerfile, an image, and a
running container. Run the same application directly and inside Docker.
Work through one step at a time; the commands below are not a single script.

This exercise uses Python's standard library and needs no third-party packages.
Its HTTP server is for learning and is not intended for a production deployment.

## 1. Check the tools

Open PowerShell and change to this lesson's directory inside your checkout:

```powershell
cd lessons/01-containers
python --version
docker version
```

The Docker command should report both a client and a server. If it reports only
the client and a connection error, check that Docker Desktop is running and its
Linux engine is ready before continuing. Do not change WSL settings to guess at
the cause. If `python` is unavailable, resolve that before the direct Python step;
having Python inside a Docker image does not install Python on Windows.

## 2. Read the application

Open `app.py` before running it:

- `Handler.do_GET` handles an HTTP GET request.
- A request to `/` returns JSON containing a message and the process's hostname.
- Other paths return HTTP 404.
- `ThreadingHTTPServer` listens on port 8000 and handles incoming requests.
- `0.0.0.0` makes the server listen on all IPv4 interfaces in its environment.
  Inside a container this allows published traffic to reach the application.

## 3. Run directly with Python

```powershell
python app.py
```

Keep the terminal open. Visit <http://localhost:8000/> in a browser. The expected
response resembles the following; the instance name depends on your machine:

```json
{"message": "Hello from the TFG application!", "instance": "your-hostname"}
```

Record the actual response. Stop the application with `Ctrl+C` before continuing,
so port 8000 is available for Docker. During direct execution the application
listens on all host IPv4 interfaces; actual network reachability also depends on
the host's network and firewall configuration.

## 4. Read the Dockerfile

| Instruction | Purpose |
| --- | --- |
| `FROM python:3.12-slim` | Start from an image containing Python and a small Linux user space. |
| `WORKDIR /app` | Set the working directory for subsequent instructions and the process. |
| `COPY app.py .` | Copy the application into the image. |
| `EXPOSE 8000` | Document the application's port; this does not publish it on the host. |
| `CMD [...]` | Define the default process to start when a container is created. |

The `-u` Python option sends output without buffering, making logs easier to see.
The `.dockerignore` file keeps unrelated files out of the build context.
The base image tag can change over time; this first exercise does not yet pin
the base image by digest. Record the resulting image ID for the local run.

## 5. Build the image

Run from the directory containing the Dockerfile:

```powershell
docker build -t tfg-hello:lesson-01 .
docker image inspect tfg-hello:lesson-01 --format '{{.Id}}'
```

The final `.` selects the current directory as the build context. `-t` assigns
the name `tfg-hello` and tag `lesson-01` to the resulting image. The first build
requires network access to download the base image if it is not already cached.
Building creates an image; it does not start the application as a running service.

## 6. Run a container

```powershell
docker run --rm --name tfg-hello -p 127.0.0.1:8000:8000 tfg-hello:lesson-01
```

- `--rm` removes the container after it stops; the image remains.
- `--name` gives this container a convenient name.
- `-p 127.0.0.1:8000:8000` publishes the container's port 8000 on the host's
  loopback address at port 8000.
- The last argument selects the image to run.

Visit <http://localhost:8000/> again and record the response. The hostname now
comes from the container's environment. It identifies that environment; it is
not a permanent application identifier.

In a second terminal, inspect the running container:

```powershell
docker ps --filter name=tfg-hello
docker logs tfg-hello
```

Stop it gracefully from the second terminal:

```powershell
docker stop tfg-hello
```

## Record the result

Keep expected output separate from your observations. For this exercise, record:

- Python and Docker versions.
- Whether the direct Python run worked and its JSON response.
- Whether the image build worked and the resulting image ID.
- Whether the container run worked and its JSON response.
- Any errors and the steps that resolved them.

Before moving on, explain in your own words why `docker build` and `docker run`
are different, why `EXPOSE` is not enough to access the container from the host,
and whether stopping a container also removes its image.
