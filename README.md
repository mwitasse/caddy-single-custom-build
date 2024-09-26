# Caddy Docker Custom Builds
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/mwitasse/caddy-custom-builds?label=Release)](https://github.com/mwitasse/caddy-custom-builds/releases)
[![GitHub build status](https://img.shields.io/github/actions/workflow/status/mwitasse/caddy-custom-builds/update-tag-release.yml?label=Auto-update)](https://github.com/mwitasse/caddy-custom-builds/actions/workflows/update-tag-release.yml)
[![License](https://img.shields.io/github/license/mwitasse/caddy-custom-builds?label=License)](https://github.com/mwitasse/caddy-custom-builds/blob/main/LICENSE)

[Caddy](https://github.com/caddyserver/caddy) takes a [modular approach](https://caddyserver.com/docs/extending-caddy) to building Docker images, allowing users to include only the [modules](https://caddyserver.com/docs/modules/) they need. This repository aims to provide flexibility and convenience to run Caddy with specific combinations of modules by providing pre-built images according to the needs and preferences of the users.

All custom images are updated automatically when a [new version](https://github.com/caddyserver/caddy/releases) of Caddy is released using the official [Caddy Docker](https://hub.docker.com/_/caddy) image. This is done by using GitHub Actions to build and push the images for all Caddy supported platforms to Docker Hub, GitHub Packages and Quay container registries. In addition, since the update cycle of many modules is faster than Caddy's, all custom images are periodically re-built with the latest version of their respective modules on the first day of every month. Those who are already running Caddy's latest version can force the update by re-creating the container (i.e. running `docker compose up --force-recreate` if using Docker Compose).

All commits and tags are signed with a GPG key to ensure their integrity and authenticity, and 2FA is enabled in the accounts involved in the management of this repository and the container registries.

## Builds

If you are looking for a specific custom build not available yet in this repository, please open a new [Issue](https://github.com/mwitasse/caddy-custom-builds/issues) with your request. To make sure no broken or unsafe builds are created, the requested modules should be properly maintained and listed in the Caddy's [download page](https://caddyserver.com/download). Additional information and instructions can be found by clicking on the name of the Caddy images and modules listed below.

### Caddy Images:

- [**caddy-netcup-duckdns**](https://github.com/mwitasse/caddy-custom-builds/tree/main/caddy-netcup-duckdns): includes Netcup and DuckDNS Dynamic DNS module.
- [**caddy-netcup-duckdns-ddns-ratelimit-dockerproxy**](https://github.com/mwitasse/caddy-custom-builds/tree/main/caddy-netcup-duckdns-ddns-ratelimit-dockerproxy): includes Netcup, DuckDNS, Dynamic DNS, Rate Limit, Docker Proxy module.

### Modules:

- [**DuckDNS**](https://github.com/mwitasse/caddy-custom-builds?tab=readme-ov-file#dns-modules): for DuckDNS DNS-01 ACME validation support | [caddy-dns/duckdns](https://github.com/caddy-dns/duckdns)
- [**Netcup DNS**](https://github.com/mwitasse/caddy-custom-builds?tab=readme-ov-file#dns-modules): for Netcup DNS-01 ACME validation support | [caddy-dns/netcup](https://github.com/caddy-dns/netcup)
- [**Rate Limit**](https://github.com/mwitasse/caddy-custom-builds?tab=readme-ov-file#rate-limit): implements both internal and distributed HTTP rate limiting | [mholt/caddy-ratelimit](https://github.com/mholt/caddy-ratelimit)
- [**Docker Proxy**](https://github.com/mwitasse/caddy-custom-builds?tab=readme-ov-file#docker-proxy): enables Caddy to be used for Docker containers via labels | [lucaslorentz/caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy)

## Usage

Since all images from this repository are built off the official Caddy Docker image, the same [volumes](https://docs.docker.com/storage/volumes/) and/or [bind mounts](https://docs.docker.com/storage/bind-mounts/), ports mapping, environment variables, etc. can be used with this container. Please refer to the official [Caddy Docker](https://hub.docker.com/_/caddy) image and [docs](https://caddyserver.com/docs/) for more information on using Caddy.

Docker builds for all Caddy supported platforms are available at the following container registries:
- **Docker Hub** > `docker pull mwitasse/<caddy-build-name>:latest`
- **GitHub Packages** > `docker pull ghcr.io/mwitasse/<caddy-build-name>:latest`
- **Quay** > `docker pull quay.io/mwitasse/<caddy-build-name>:latest`

To pull a specific build, replace `<caddy-build-name>` with the desired one. For example, to pull the `caddy-cloudflare` build from Docker Hub, use `docker pull mwitasse/caddy-cloudflare:latest`.

### Tags

The following tags are available for all images:

- `latest`
- `<version>` (eg: `2.7.4`, including: `2.7`, `2`, etc.)

### Container Creation

Simply create the container using the `docker run` command, or a `docker-compose.yml` file including the necessary [environment variables](https://caddyserver.com/docs/caddyfile/concepts#environment-variables) depending on the modules used. The following blocks contain examples for both methods using `<caddy-build-name>` as the image name (replace it with the desired Caddy build name), and including all environment variables required by the modules listed above (some may not apply to your specific build).

#### Docker Run

```sh
docker run --rm -it \
  --name caddy \  # feel free to choose your own container name
  --restart unless-stopped \  # run container unless stopped by user (optional)
  -p 80:80 \  # HTTP port
  -p 443:443 \  # HTTPS port
  -p 443:443/udp \  # HTTP/3 port (optional)
  -v caddy-data:/data \  # volume mount for certificates data
  -v caddy-config:/config \  # volume mount for configuration data
  -v $PWD/Caddyfile:/etc/caddy/Caddyfile \  # to use your own Caddyfile
  -v $PWD/log:/var/log \  # bind mount for the log directory (optional)
  -v $PWD/srv:/srv \  # bind mount to serve static sites or files (optional)
  -e DUCKDNS_API_TOKEN=<token-value> \  # DuckDNS API token (if applicable)
  -e NETCUP_CUSTOMER_NUMBER=<number-value> \  # Netcup customer number (if applicable)
  -e NETCUP_API_KEY=<key-value> \  # Netcup API key (if applicable)
  -e NETCUP_API_PASSWORD=<password-value> \  # Netcup API password (if applicable)
  mwitasse/<caddy-build-name>:latest  # replace with the desired Caddy build name
```

The volume and bind mounts can be adjusted to meet to your needs, `$PWD` is used to reference the current working directory, but you can replace it with your preferred path. The environment variables are only required if the modules used in the build require them.

The default [Caddyfile](https://github.com/caddyserver/dist/blob/master/config/Caddyfile) that is included inside the Docker container is just a placeholder to serve a static Caddy welcome page with some useful instructions. So you will most likely want to mount your own `$PWD/Caddyfile` to configure Caddy according to your needs (the file must already exist in the specified path before creating the container).

The [restart policy](https://docs.docker.com/config/containers/start-containers-automatically/#use-a-restart-policy) can be adjusted to your needs. The policy `unless-stopped` ensures the container is always running (even at boot) unless it is explicitly stopped by the user.

#### Docker Compose

```yaml
services:
  caddy:
    image: mwitasse/<caddy-build-name>:latest  # replace with the desired Caddy build name
    container_name: caddy  # feel free to choose your own container name
    restart: "unless-stopped"  # run container unless stopped by user (optional) 
    ports:
      - "80:80"  # HTTP port
      - "443:443"  # HTTPS port
      - "443:443/udp"  # HTTP/3 port (optional)
    volumes:
      - caddy-data:/data  # volume mount for certificates data
      - caddy-config:/config  # volume mount for configuration data
      - $PWD/Caddyfile:/etc/caddy/Caddyfile  # to use your own Caddyfile
      - $PWD/log:/var/log  # bind mount for the log directory (optional)
      - $PWD/srv:/srv  # bind mount to serve static sites or files (optional)
    environment:
      - DUCKDNS_API_TOKEN=<token-value>  # DuckDNS API token (if applicable)
      - NETCUP_CUSTOMER_NUMBER=<number-value>  # Netcup customer number (if applicable)
      - NETCUP_API_KEY=<key-value>  # Netcup API key (if applicable)
      - NETCUP_API_PASSWORD=<password-value>  # Netcup API password (if applicable)
volumes:
  caddy-data:
    external: true
  caddy-config:
```

Defining the data volume as [external](https://docs.docker.com/compose/compose-file/compose-file-v3/#external) ensures that `docker compose down` does not delete the volume, but you may need to create it first using `docker volume create caddy-data`. This doesn't apply to bind mounts if you opt to use them instead of volumes.

### Graceful Reloads

Caddy does not require a full restart when the `Caddyfile` is modified. Caddy comes with a [caddy reload](https://caddyserver.com/docs/command-line#caddy-reload) command which can be used to reload its configuration with zero downtime.

When running Caddy in Docker, the recommended way to trigger a config reload is by executing the `caddy reload` command in the running container. First, you'll need to determine your container ID or name. Then, pass the container ID to docker exec. The working directory is set to /etc/caddy so Caddy can find your `Caddyfile` without additional arguments.

```sh
caddy_container_id=$(docker ps | grep caddy | awk '{print $1;}')  # use your container name if different from 'caddy'
docker exec -w /etc/caddy $caddy_container_id caddy reload
```

It is possible to create an [alias](https://phoenixnap.com/kb/linux-alias-command) for the `caddy reload` command to make it more convenient to use by adding the following line to your `~/.bashrc` or `~/.zshrc` file:
    
```sh
alias caddy-reload="docker exec -w /etc/caddy $(docker ps | grep caddy | awk '{print $1;}') caddy reload"
```

Once you have added the alias to the appropriate file, you will need to source it for the changes to take effect. You can do this by running `source ~/.bashrc` or `source ~/.zshrc` in your terminal. After this, you will be able to use the `caddy-reload` alias in your terminal sessions.

## Configuration

This section aims to provide some basic information on how to configure Caddy with the modules included in the custom builds, but it is not intended to be a comprehensive guide. All the examples are based on the official [Caddyfile](https://caddyserver.com/docs/caddyfile) syntax and the modules' documentation.

### DNS Modules

To make use of the different modules that provide DNS-01 ACME validation support at the server level, set the global [acme_dns](https://caddyserver.com/docs/caddyfile/options#acme-dns) directive in your `Caddyfile` using your DNS provider's name and the respective environment variable for the API token. The example shows the use case for Cloudflare DNS with the rest of the DNS providers commented out.

```Caddyfile
{
  acme_dns cloudflare {env.CLOUDFLARE_API_TOKEN} #  for Cloudflare
  # acme_dns duckdns {env.DUCKDNS_API_TOKEN} #  for DuckDNS
  # acme_dns netcup {  # for Netcup
  #   customer_number {env.NETCUP_CUSTOMER_NUMBER}
  #   api_key {env.NETCUP_API_KEY}
  #   api_password {env.NETCUP_API_PASSWORD}
  # }
  # acme_dns porkbun {  # for Porkbun
  #   api_key {env.PORKBUN_API_KEY}
  #   api_secret_key {env.PORKBUN_API_SECRET_KEY}
  # }
}
```

Alternatively, you can use the [`tls`](https://caddyserver.com/docs/caddyfile/directives/tls#tls) directive at each site. See the [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare) module for additional details.

```Caddyfile
my.domain.tld {
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}  # for Cloudflare
    # dns duckdns {env.DUCKDNS_API_TOKEN}  # for DuckDNS
    # dns netcup {  # for Netcup
    #   customer_number {env.NETCUP_CUSTOMER_NUMBER}
    #   api_key {env.NETCUP_API_KEY}
    #   api_password {env.NETCUP_API_PASSWORD}
    # }
    # dns porkbun {  # for Porkbun
    #   api_key {env.PORKBUN_API_KEY}
    #   api_secret_key {env.PORKBUN_API_SECRET_KEY}
    # }
  }
}
```

#### Creating a DuckDNS API Token

To generate a DuckDNS API token, login to your [DuckDNS](https://www.duckdns.org/) account, copy the token, and use it as the `DUCKDNS_API_TOKEN` environment variable. You can recreate the token by clicking on the three vertical lines in the top right corner next to your logged in email, and selecting the recreate token option.

#### Creating a Netcup API Token

To generate a Netcup API token follow the steps from the [Netcup API docs](https://helpcenter.netcup.com/en/wiki/general/our-api). Use the `NETCUP_CUSTOMER_NUMBER`, `NETCUP_API_KEY` and `NETCUP_API_PASSWORD` environment variables in the Docker Compose/Run and `Caddyfile` configuration.

### Dynamic DNS

To keep your DNS records updated with the public IP address of your instance, add the [dynamic_dns](https://caddyserver.com/docs/modules/dynamic_dns) directive to the [global options](https://caddyserver.com/docs/caddyfile/options) in your `Caddyfile`. This module regularly queries a service for your public IP address and updates the DNS records via your DNS provider's API whenever it changes. For additional details and advanced configuration examples refer to [mholt/caddy-dynamicdns](https://github.com/mholt/caddy-dynamicdns) repository. The example shows the use case for Cloudflare DNS with the rest of the DNS providers commented out.

```Caddyfile
{
  dynamic_dns {
    provider duckdns {env.DUCKDNS_API_TOKEN}  # for DuckDNS
    # dns netcup {  # for Netcup
    #   customer_number {env.NETCUP_CUSTOMER_NUMBER}
    #   api_key {env.NETCUP_API_KEY}
    #   api_password {env.NETCUP_API_PASSWORD}
    # }
    domains {
      domain.tld
    }
  }
}
```

Using the option [dynamic_domains](https://github.com/mholt/caddy-dynamicdns#dynamic-domains), it can also be configured to scan through the domains configured in the `Caddyfile` and try to manage those DNS records.

### Rate Limit

The [rate_limit](https://caddyserver.com/docs/modules/http.handlers.rate_limit#github.com/mholt/caddy-ratelimit) HTTP handler module lets you define rate limit zones, which have a unique name of your choosing. If a rate limit is exceeded, an HTTP error with status 429 will be returned. This error can be handled using the conventional error handling routes in your config.

Additional information and `Caddyfile` configuration examples can be found in the [mholt/caddy-ratelimit](https://github.com/mholt/caddy-ratelimit) repository.

### Docker Proxy

The plugin scans Docker metadata, looking for labels indicating that the service or container should be served by Caddy. Then, it generates an in-memory `Caddyfile` with site entries and proxies pointing to each Docker service by their DNS name or container IP. Every time a Docker object changes, the plugin updates the `Caddyfile` and triggers Caddy to gracefully reload with zero downtime.

Additional information and `Caddyfile` configuration examples can be found in the [lucaslorentz/caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy) repository.

## Contributing

Feel free to contribute, request additional Caddy images with your preferred modules, and make things better by opening an [Issue](https://github.com/mwitasse/caddy-custom-builds/issues) or [Pull Request](https://github.com/mwitasse/caddy-custom-builds/pulls).

## License

Software under [GPL-3.0](https://github.com/mwitasse/caddy-custom-builds/blob/main/LICENSE) ensures users' freedom to use, modify, and distribute it while keeping the source code accessible. It promotes transparency, collaboration, and knowledge sharing. Users agree to comply with the GPL-3.0 license terms and provide the same freedom to others.