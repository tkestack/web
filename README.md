# TKEStack website

The [TKEStack](https://tkestack.github.io/web/) site, built using [Hugo](https://gohugo.io/) and [Docsy theme](https://github.com/google/docsy).
## Running the website locally in container

You can run this website inside a [Docker](https://docs.docker.com/)
container. This approach doesn't require you to install any dependencies other
than [Docker Desktop](https://www.docker.com/products/docker-desktop) on
Windows and Mac, and [Docker Compose](https://docs.docker.com/compose/install/)
on Linux.

1. Clone this repo:

    ```bash
    git clone --recurse-submodules --depth 1 https://github.com/tkestack/web.git
    ```

2. Build the docker image and run 

   ```bash
   docker-compose up --build
   ```

3. Verify that the service is working. 

   Open your web browser and type `http://localhost:1313`

### Cleanup

To stop Docker Compose, on your terminal window, press **Ctrl + C**. 

To remove the produced images run:

```console
docker-compose rm
```

For more information see the [Docker Compose
documentation](https://docs.docker.com/compose/gettingstarted/).

