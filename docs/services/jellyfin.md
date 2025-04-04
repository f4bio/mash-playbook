# Jellyfin

[Jellyfin](https://jellyfin.org/) Jellyfin is the volunteer-built media solution that puts you in control of your media. Stream to any device from your own server, with no strings attached. Your media, your server, your way.

## Dependencies

This service requires the following other services:

- a [Traefik](traefik.md) reverse-proxy server

## Configuration

To enable this service, add the following configuration to your `vars.yml` file and re-run the [installation](../installing.md) process:

```yaml
########################################################################
#                                                                      #
# jellyfin                                                                 #
#                                                                      #
########################################################################

jellyfin_enabled: true

jellyfin_hostname: jellyfin.example.com

# The path where media files are stored on the host system (defaults to /mash/jellyfin/media)
jellyfin_media_path: "{{ jellyfin_base_path }}/media"

# The path at which jellyfin_media_path is mounted to inside the container
# Takes a path value (e.g. "/media"), or empty string to not mount.
jellyfin_media_bind_path: "/media"

# Since the container is NOT run in 'host' networking mode
# it is required that a claim token be provided during first time setup
#
# Link to obtain -> https://jellyfin.tv/claim
# Keep in mind that the claim token expires after 4 minutes.
jellyfin_claim_token: ""

########################################################################
#                                                                      #
# /jellyfin                                                                #
#                                                                      #
########################################################################
```

### URL

In the example configuration above, we configure the service to be hosted at `https://jellyfin.example.com`.

A `jellyfin_path_prefix` variable can be adjusted to host under a subpath (e.g. `jellyfin_path_prefix: /jellyfin`), but this hasn't been tested yet.

### Exposing ports

By default no ports are exposed, but you'll most likely want to adjust this. The below defines what these ports are and why you may want to expose them.

```yaml
# The main Jellyfin webserver port, you'll want to set this variable (and configure port-forwarding in your router) if you want to access Jellyfin from https://app.jellyfin.tv
# Or if you want to access Jellyfin via TV and phone apps
jellyfin_container_http_bind_port: 8096

# Access to the Jellyfin DLNA server
# You probably don't need this
jellyfin_container_dlna_udp_bind_port: 1900
jellyfin_container_dlna_tcp_bind_port: 32469
```

Upstream documentation: <https://jellyfin.org/docs/general/networking>

### Hardware Acceleration

To enable hardware acceleration you'll first need to determine your GPU brand. Once you've done this, read the corresponding section below:

#### Intel/ATI/AMD

For Intel/ATI/AMD GPUs enabling hardware acceleration is as easy as mounting the device into the container:

```yaml
# The path where the Intel/ATI/AMD GPU is on the host system
jellyfin_gpu_path: "/dev/dri"

# The path to mount the Intel/ATI/AMD GPU to in the container.
# Takes a path value (e.g. "/dev/dri"), or empty string to not mount.
jellyfin_gpu_bind_path: "{{ jellyfin_gpu_path }}"
```

Upstream documentation: <https://jellyfin.org/docs/general/administration/hardware-acceleration/intel>

#### NVIDIA

For NVIDIA GPUs enabling hardware acceleration is a little bit tricky, since it (currently) requires the manual installation of the [NVIDIA container runtime](https://github.com/NVIDIA/nvidia-container-toolkit). Consult your distributions documentation on installing this.

Once the runtime is installed and available, add the following configuration:

```yaml
# The container runtime that the container engine should use
jellyfin_container_runtime: "nvidia"

# To enable NVIDIA GPU hardware acceleration this value should either be 'all' or the UUID value of the GPU
# which can obtained with the command -> 'nvidia-smi --query-gpu=gpu_name,gpu_uuid --format=csv'
jellyfin_nvidia_visible_devices: "all"
```

Upstream documentation: <https://jellyfin.org/docs/general/administration/hardware-acceleration/nvidia>

---

To verify Jellyfin is detecting your GPU navigate to `Settings -> Transcoder -> Hardware transcoding device` and select your GPU. If you do not see the `Hardware transcoding device` drop-down make sure you have ticked the `Use hardware acceleration when available` checkbox. If everything is working right you should see something like this:

![Jellyfin Configure Transcoding](../assets/jellyfin/transcoder.png)

### Mounting additional media directories

To mount additional media directories, or to simply mount a directory without changing its permissions, the following configuration is available:

```yaml
jellyfin_container_additional_volumes:
  - type: bind
    src: /path/on/the/host/movies
    dst: /movies
  - type: bind
    src: /another-path/on/the/host/anime
    dst: /anime
    options: readonly
```

### Jellyfin Pass updates

To enable Jellyfin Pass updates you will (unfortunately) have to run the container as a root user AND have to disable the container being in read-only mode. You'll also want to set `jellyfin_version_environment_variable` to `latest` or `public`:

```yaml
# The user/group to run the application as
# In the below example, '0:0' indicates the root user and root group
jellyfin_uid: '0'
jellyfin_gid: '0'

# Controls whether the container filesystem is read-only
jellyfin_container_read_only: false

# Valid settings for 'jellyfin_version_environment_variable' are:
#
# 1. docker: Let Docker handle the Jellyfin Version, we keep our Dockerhub Endpoint up to date with the latest public builds.
# 2. latest: will update jellyfin to the latest version available that you are entitled to.
# 3. public: will update jellyfinpass users to the latest public version, useful for jellyfinpass users that don't want to be on the bleeding edge but still want the latest public updates.
# 4. <specific-version>: will select a specific version (eg 0.9.12.4.1192-9a47d21) of jellyfin to install, note you cannot use this to access jellyfinpass versions if you do not have jellyfinpass.
#
# NOTE -> You cannot update to a JellyfinPass only (beta) version if you are not logged in with a JellyfinPass account
jellyfin_version_environment_variable: latest
```

## Usage

After [installation](../installing.md), you should access your new Jellyfin instance at the URL you've chosen. Follow the prompts to finish setup. When prompted to add your media libraries keep in mind that it will be the path **inside** the container, most likely some variation of your `jellyfin_media_bind_path` variable.

If it is your first time setting up the server, make sure you have set your Jellyfin claim token with the variable `jellyfin_claim_token` which can be obtained from <https://jellyfin.tv/claim>. You only have to do this once, so once the server it setup you can remove the variable.

## Recommended other services

Consider these other supported services that are also in the [*Arr stack](https://wiki.servarr.com/) of media automation tools:

- [Radarr](radarr.md)
- [Sonarr](sonarr.md)
- [Jackett](jackett.md)
- [qBittorrent](qbittorrent.md)
- [Overseerr](overseerr.md)
