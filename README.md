# Forgejo deployment for Singularity

This project shows how to deploy Forgejo as Singularity containers using [singularity-compose](https://singularityhub.github.io/singularity-compose/#/), version 2.0 of spec.


## Why?

Singularity can run containers in fully rootless mode, making it suitable for public cluster environments. In contrast, Docker is a big security hole by design: a user in Docker group can easily elevate priviledges and become a root.


## Preparation

Install singularity-compose via Git:

```
python3 -m venv ./venv
source ./venv/bin/activate.fish
pip install git+https://github.com/singularityhub/singularity-compose.git
```

Create folders that shall be mounted into containers:

```
mkdir forgejo
mkdir mysql
```


## Deployment

```
singularity-compose --log-level DEBUG up
```


# Notes

In principle, `singularity-compose` is similar to `docker-compose`, but there are a few important differences:

* The main reason to bother with Singularity is an ability to execute rootless containers, by adding a `fakeroot` setting:

    ```
    start:
      options:
        - fakeroot
    ```

* Docker `services` are called `instances`

* The mapped folders are not created automatically if they do not exist

* Environment settings are not available, however a file with exported variables can be mapped to `/.singularity.d/env/`:

    ```
    volumes:
      - ./env_file.sh:/.singularity.d/env/env_file.sh
    ```

* By default, container does not execute `ENTRYPOINT` and `CMD` imported from the original Docker container, because they are connected to `runscript`, not to the `startscript`, which `singularity-compose` executes by default. In order to have `singularity-compose` to run `ENTRYPOINT` and `CMD` as in Docker, an empty `run` config must be added:

    ```
    run: []
    ```

* There is no `daemon` mode option similar to `docker-compose up -d`. Instead, the background execution has to be specified for each container (replacing the previous `run: []` setting):

    ```
    run:
       background: true
    ```

* In rootless mode is not possible to use network bridge, as required by a typical `network` setting. The `singularity-compose` does not offer any alternatives (perhaps, `passt` can be used?). Therefore, we disable the network, which means instances shall use the ports of host network directly:

    ```
    network:
      enable: false
    ```

Since the network is disabled, we can omit the `ports` setting as well.

Note that Forgejo server still tries to bind the host's port 22, unless we add the following to the `env_file.sh`:

```
export SSH_PORT=3022
```
