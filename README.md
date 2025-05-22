[![Build Stable](https://github.com/frappe/frappe_docker/actions/workflows/build_stable.yml/badge.svg)](https://github.com/frappe/frappe_docker/actions/workflows/build_stable.yml)
[![Build Develop](https://github.com/frappe/frappe_docker/actions/workflows/build_develop.yml/badge.svg)](https://github.com/frappe/frappe_docker/actions/workflows/build_develop.yml)

Everything about [Frappe](https://github.com/frappe/frappe) and [ERPNext](https://github.com/frappe/erpnext) in containers.

# Getting Started

To get started you need [Docker](https://docs.docker.com/get-docker/), [docker-compose](https://docs.docker.com/compose/), and [git](https://docs.github.com/en/get-started/getting-started-with-git/set-up-git) setup on your machine. For Docker basics and best practices refer to Docker's [documentation](http://docs.docker.com).

# Deploy Custom App

## Build Image

### Load custom apps through apps.json file

Base64 encoded string of `apps.json` file needs to be passed in as build arg environment variable.

Create the following `./resources/apps.json` file:

```json
[
  {
    "url": "https://{{username}}:{{personal-access-token}}@git.example.com/project/repository.git",
    "branch": "main"
  }
]
```

Note:

- The `url` needs to be http(s) git url with personal access tokens in case of private repo.
- Add dependencies manually in `apps.json` e.g. add `erpnext` if you are installing `hrms`.
- Use fork repo or branch for ERPNext in case you need to use your fork or test a PR.

Generate base64 string from json file:

```shell
export APPS_JSON_BASE64=$(base64 -w 0 ./resources/apps.json)
```

Test the Previous Step: Decode the Base64-encoded Environment Variable

To verify the previous step, decode the `APPS_JSON_BASE64` environment variable (which is Base64-encoded) into a JSON file. Follow the steps below:

1. Use the following command to decode and save the output into a JSON file named apps-test-output.json:

```shell
echo -n ${APPS_JSON_BASE64} | base64 -d > apps-test-output.json
```

2. Open the apps-test-output.json file to review the JSON output and ensure that the content is correct.

### Configure google service account

Put service account to `resources/google-cloud-storage.json`

### Configure build

Common build args.

- `FRAPPE_PATH`, customize the source repo for frappe framework. Defaults to `https://github.com/frappe/frappe`
- `FRAPPE_BRANCH`, customize the source repo branch for frappe framework. Defaults to `version-15`.
- `APPS_JSON_BASE64`, correct base64 encoded JSON string generated from `apps.json` file.

Notes

- Use `buildah` or `docker` as per your setup.
- Make sure `APPS_JSON_BASE64` variable has correct base64 encoded JSON string. It is consumed as build arg, base64 encoding ensures it to be friendly with environment variables. Use `jq empty apps.json` to validate `apps.json` file.
- Make sure the `--tag` is valid image name that will be pushed to registry. See section [below](#use-images) for remarks about its use.
- `.git` directories for all apps are removed from the image.

### Custom build image

This method builds the base and build layer every time, it allows to customize Python and NodeJS runtime versions. It takes more time to build.

It uses `images/custom/Containerfile`.

```shell
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg BUILD_NO_CACHE=$(date +%s) \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=PYTHON_VERSION=3.11.9 \
  --build-arg=NODE_VERSION=18.20.2 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=frappe-bio-ga \
  --file=images/custom/Containerfile .
```

Custom build args,

- `PYTHON_VERSION`, use the specified python version for base image. Default is `3.11.6`.
- `NODE_VERSION`, use the specified nodejs version, Default `18.18.2`.
- `DEBIAN_BASE` use the base Debian version, defaults to `bookworm`.
- `WKHTMLTOPDF_VERSION`, use the specified qt patched `wkhtmltopdf` version. Default is `0.12.6.1-3`.
- `WKHTMLTOPDF_DISTRO`, use the specified distro for debian package. Default is `bookworm`.

## Docker Command

## Compose up

```shell
docker compose -f pwd.yml up -d
```

## Show logs

```shell
docker logs frappe_docker-create-site-1 -f
```

## Compose down

```shell
docker compose -f pwd.yml down
```

# Contributing

If you want to contribute to this repo refer to [CONTRIBUTING.md](CONTRIBUTING.md)

This repository is only for container related stuff. You also might want to contribute to:

- [Frappe framework](https://github.com/frappe/frappe#contributing),
- [ERPNext](https://github.com/frappe/erpnext#contributing),
- [Frappe Bench](https://github.com/frappe/bench).
