# cartogram-docker: Docker Application for Web-based Cartogram Generator

cartogram-docker provides a Docker Compose setup for easily installing and running [cartogram-web](https://github.com/go-cart-io/cartogram-web), a web-based cartogram generator. You can use the tool online at https://go-cart.io.

We recommend starting with the production setup described in the [Production](#production) section. This configuration is suitable for most users who simply want to run go-cart.io locally. Once you have the production setup working, if you plan to modify the go-cart.io codebase, you can then explore the development setup in the [Development](#development) section. The development setup enables hot-reloading, allowing you to see code changes reflected in the website in real-time.

These instructions are for Linux and macOS. Windows users should use [WSL 2](https://learn.microsoft.com/en-us/windows/wsl/install).

## Production

1. [Install Docker and Docker Compose](https://docs.docker.com/engine/install/)

2. Clone this repository and go into the cloned directory:

```shell script
git clone https://github.com/go-cart-io/cartogram-docker.git
cd cartogram-docker
```

3. Copy `password.txt.dist` to `password.txt` and `.env.dist` to `.env` using:

```shell script
cp password.txt.dist password.txt && cp .env.dist .env
```

4. Modify `password.txt` and `.env` as needed, leaving `password.txt.dist` and `.env.dist` unchanged.

5. Run the following commands from the root directory of this repository (i.e., the folder containing this readme).

```shell script
export TAG=":$(git ls-tree HEAD cartogram-web | awk '{print $3}' | xargs git rev-parse --short)"
docker compose up -d
```

The first time you run this command, it may take a while to download and install dependencies. Once everything stats up, you should be able to access the locally-running instance of go-cart.io website at [http://localhost:5001](http://localhost:5001).

6. When you would like to stop the go-cart.io local instance, use the command:

```shell script
docker compose stop
```

## Development

1. Follow step 1 - 6 in [Production](#production)

2. Initialize and update the submodules using:

```shell script
cd cartogram-docker
git submodule update --init --recursive
```

3. [Install Node.js](https://nodejs.org).

4. Start frontend development server:

```shell script
cd cartogram-web/frontend
npm install
npm run dev
```

5. Next, open another terminal and run the following commands in the root directory of this repository (i.e., the folder containing this readme):

```shell script
cd cartogram-docker
bash cartogram-web/tools/pull-executable.sh
export TAG=":$(git -C cartogram-web/ rev-parse --short HEAD)"
docker compose -f docker-compose-dev.yml up
```

The first command pulls the same version of the executable (provided by [cartogram-cpp](https://github.com/mgastner/cartogram-cpp)) as required by currently checked-out HEAD of the `cartograb-web` submodule.

The second command starts all the services (i.e. docker containers) listed in `docker-compose-dev.yml` under the `web` profile (`web`, `redis`, and `postgres`), while ensuring the same image of `cartogram-web` is pulled as the submodule's version. We achive this by making sure the short SHA of the submodule's HEAD is the same as the docker image's tag.

Once all the Docker containers have started, their output will be displayed in the terminal window you opened in step 5. This window will let you monitor how the web application responds to requests and identify any errors. You can access the locally running go-cart.io website at [http://localhost:5001](http://localhost:5001).

When you make changes to the code in the `cartogram-web` repository, their respective servers will reload automatically when you save your changes. When you make changes to the code in the `cartogram-docker` repository, you should restart the docker compose:

```shell script
docker compose -f docker-compose-dev.yml restart
```

When you would like to shut down the go-cart.io web application, simply press `Ctrl-C` in the terminal window you started Docker Compose and npm in. After a few moments, the state of all the Docker containers will be saved and the application will gracefully come to a halt.

### Useful tips

If you mostly are in the development mode, you may modify `.env` by replace `COMPOSE_FILE=docker-compose.yml` with `COMPOSE_FILE=docker-compose-dev.yml`. This way, you may simply use `docker compose up -d` or `docker compose up` for starting the application.

You may find the following commands useful when developing:

- `docker compose up -d` to start docker containers in the background (so you can use command line for other things);
- `docker exec -it cartogram-docker-web-1 /bin/bash` to access the cartogram-web container (so you can use the command line in the container, say to run `python addmap.py init test`);
- `docker start cartogram-docker-web-1` to start the cartogram-web container (which is useful when the container stops because of, say, an error in your python code);
- `docker logs --follow cartogram-docker-web-1` to see the logs in the cartogram-web container.

### Tablet testing via local network

If you want to access the local instance via tablets, please follow the instructions [here](https://github.com/go-cart-io/cartogram-docker/blob/main/docs/testing_on_tablet.md).

### Pushing changes to go-cart.io server

While you can make changes and test them locally, you need to follow the instructions [here](https://github.com/go-cart-io/cartogram-docker/blob/main/docs/deploy.md) to push changes to the go-cart.io server. If you are not part of the core development team, you may not have permission to do so. Please create a pull request in the relevant repository (e.g., [go-cart-io/cartogram-web](https://github.com/go-cart-io/cartogram-web) if you made changes in the `cartogram-web` folder) so we can push the changes for you.

# Contributing

Contributions are highly encouraged, and we love pull requests! Feel free to take a stab at any of the open issues. If there are no issues, don't hesitate to suggest new features, or report bugs under the issues tab (and, optionally, even subsequently send in pull requests with said enhancements or fixes). If you need help getting setup or more guidance contributing, please @ any of the main contributors under the issue you'd like to help solve, and we'll be happy to guide you!
