# Installation

## Dependencies

- You have to install [Docker](https://docs.docker.com/get-docker/) and [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) to run bayes3d in a container.
- GPU support is required to run bayes3d.

## Installation process

- Copy `.env` file:

    ```bash
    $ cd docker
    $ cp .env.example .env
    ```

- Edit `.env` as you like.
- Copy `docker-compose.override.yml`:

    ```bash
    $ cp docker-compose.override.yml.example docker-compose.override.yml
    ```
- Run following commands to install bayes3d:

    ```bash
    $ cd docker
    $ docker compose build
    $ docker compose run bayes3d bash

    # @ /workspace
    $ cd opt
    
    # @ /workspace/opt
    $ uv pip install -e . --no-deps
    ```

# Check installation result

- Run following command to check the result:

    ```bash
    # @ /workspace/opt
    $ uv run --no-sync python demo.py
    ```
