# Daily Cable Access TV

> a [/daily](https://daily.co/) hackathon hack

![](./src/daily-cable-tv-logo.png)

A coworker and I were talking about how we enjoyed learning from our other coworkers, even if it was just watching them screenshare and squash a bug or delve through logs or push a small feature. And sometimes we missed just general office chatter. Wouldn't it be neat to have a constant live stream of content like this? I imagined a cable access channel TV-like experience. (`Wayne's World` was a big inspiration here.) I wanted to keep a small browser window always open in the corner of my screen, much like having a TV on in the background, that would show my coworkers doing a lunch and learn or hosting open office hours. There wouldn't be 24 hours everyday of content, so when nothing was streaming (or nothing was "on TV"), a static logo would appear. The nice thing here is that other than navigating to the site, I wouldn't have to do anything to passively access the content. Live streams would start and end and I could just sit back and watch. I imagined a blend of live and "syndicated", or pre-recorded, content and maybe some fun /daily logo bumpers, but in true Hack form, what lies below is just a POC. Essentially I wanted to blend real time video with VOD (video on demand), and basically, that's what this does, albeit niavely so.

I knew I wanted to learn more about nginx configuration, especially WRT rtmp streaming, so I started there. After I got the server config to handle live stream, hls, and VOD, I made a web 1.0 simple html page that loops an .mp4 file (the static /daily TV logo). Then I poll for RTMP stats (which we get for free from nginx, thanks nginx!). We parse the RTMP streaming stats XML for when a stream has started and when one has, we switch the video src and type on the player to HLS, and the live stream replaces the looping mp4 content. When the stream ends (again determined from polling the RTMP stats), we switch back to the mp4 video source and the player goes back to static.

Config files and DSL can be difficult to work with and create a higher than usual barrier of entry. A pretty exhaustive [list](https://github.com/arut/nginx-rtmp-module/wiki/Directives) of `nginx-rtmp-module` directives was helpful here. I had hoped to accomplish the switching between live and VOD logic _in_ the nginx config, but didn't quite get there. I'm pretty sure it is possible, at least with commerically available modules and/or the correct mix of nginx modules and more knowledge and experience than I have at the moment, with more complex configurations. For now, Javascript, my go-to programming language, gets the job done.

## Dependencies
- [docker](https://docs.docker.com/get-docker/)
- [ngrok](https://ngrok.com/)
- [/daily account](https://dashboard.daily.co/signup)

## build and run server locally

```
$ docker build . -t daily-cable-tv-server && \
docker run --rm -p 1935:1935 \
-p 3737:3737 \
--name daily-cable-tv-server-nginx-rtmp  \
daily-cable-tv-server
```

## run ngrok
- set up ngrok config [`example-ngrok.yml`](./example-ngrok.yml).
> ensure ngrok config is in correct place - for macos, that is probably `$HOME/.ngrok2/ngrok.yml`

- Check out [Phil's awesome blog post](https://www.daily.co/blog/run-your-own-live-streaming-server/) on using `ngrok` to stream rtmp through a local tunnel. Seriously, this post is gold in terms of information and useful tips. Five stars. Would read again. Run, don't walk, to check it out.

```
$ ngrok start --all
```

## 🌴 "turn on TV"
- navigate to `https://daily-cable-access.ngrok.io/hls/index.html` *in Safari or Edge*
- Only Safari and Edge video players handle HLS natively, so out of the box, only those browsers will work. (Implementing an HLS player to be able to use Chrome/Firefox is left as an exercise to the reader :)
- click the video to play (because, at least in Safari, autoplay is verboten and a user gesture is needed.)

## live stream from daily call
- Join a meeting as owner (for example, with an `is_owner` meeting token)
- Start a live stream ([documentation here](https://docs.daily.co/reference/daily-js/instance-methods/start-live-streaming#main).)

```
call.startLiveStreaming({rtmpUrl: 'rtmp://0.tcp.ngrok.io:12345/live/a_clever_stream_name'})
```
> note: "0.tcp.ngrok.io:12345" is obtained from ngrok; see [blog post](https://www.daily.co/blog/run-your-own-live-streaming-server/).

## more info and other useful links
- [inspiration](https://www.imdb.com/title/tt0105793/)
- https://hackmd.io/@quency/rker8bgZw#Config-nginx-as-RTMP
- https://www.nginx.com/blog/video-streaming-for-remote-learning-with-nginx/
- https://gist.github.com/CSRaghunandan/ce2394cd5a9c8a412f8ff5ee1478560a

- streams links:
- https://web.dev/streams/
- https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream/getReader

- nginx directives:
- https://github.com/arut/nginx-rtmp-module/wiki/Directives

-------------------

# Original [nginx-rtmp-docker](https://github.com/tiangolo/nginx-rtmp-docker) README

[![Deploy](https://github.com/tiangolo/nginx-rtmp-docker/workflows/Deploy/badge.svg)](https://github.com/tiangolo/nginx-rtmp-docker/actions?query=workflow%3ADeploy)

## Supported tags and respective `Dockerfile` links

* [`latest` _(Dockerfile)_](https://github.com/tiangolo/nginx-rtmp-docker/blob/master/Dockerfile)

**Note**: Note: There are [tags for each build date](https://hub.docker.com/r/tiangolo/nginx-rtmp/tags). If you need to "pin" the Docker image version you use, you can select one of those tags. E.g. `tiangolo/nginx-rtmp:latest-2020-08-16`.

# nginx-rtmp

[**Docker**](https://www.docker.com/) image with [**Nginx**](http://nginx.org/en/) using the [**nginx-rtmp-module**](https://github.com/arut/nginx-rtmp-module) module for live multimedia (video) streaming.

## Description

This [**Docker**](https://www.docker.com/) image can be used to create an RTMP server for multimedia / video streaming using [**Nginx**](http://nginx.org/en/) and [**nginx-rtmp-module**](https://github.com/arut/nginx-rtmp-module), built from the current latest sources (Nginx 1.15.0 and nginx-rtmp-module 1.2.1).

This was inspired by other similar previous images from [dvdgiessen](https://hub.docker.com/r/dvdgiessen/nginx-rtmp-docker/), [jasonrivers](https://hub.docker.com/r/jasonrivers/nginx-rtmp/), [aevumdecessus](https://hub.docker.com/r/aevumdecessus/docker-nginx-rtmp/) and by an [OBS Studio post](https://obsproject.com/forum/resources/how-to-set-up-your-own-private-rtmp-server-using-nginx.50/).

The main purpose (and test case) to build it was to allow streaming from [**OBS Studio**](https://obsproject.com/) to different clients at the same time.

**GitHub repo**: <https://github.com/tiangolo/nginx-rtmp-docker>

**Docker Hub image**: <https://hub.docker.com/r/tiangolo/nginx-rtmp/>

## Details

## How to use

* For the simplest case, just run a container with this image:

```bash
docker run -d -p 1935:1935 --name nginx-rtmp tiangolo/nginx-rtmp
```

## How to test with OBS Studio and VLC

* Run a container with the command above


* Open [OBS Studio](https://obsproject.com/)
* Click the "Settings" button
* Go to the "Stream" section
* In "Stream Type" select "Custom Streaming Server"
* In the "URL" enter the `rtmp://<ip_of_host>/live` replacing `<ip_of_host>` with the IP of the host in which the container is running. For example: `rtmp://192.168.0.30/live`
* In the "Stream key" use a "key" that will be used later in the client URL to display that specific stream. For example: `test`
* Click the "OK" button
* In the section "Sources" click the "Add" button (`+`) and select a source (for example "Screen Capture") and configure it as you need
* Click the "Start Streaming" button


* Open a [VLC](http://www.videolan.org/vlc/index.html) player (it also works in Raspberry Pi using `omxplayer`)
* Click in the "Media" menu
* Click in "Open Network Stream"
* Enter the URL from above as `rtmp://<ip_of_host>/live/<key>` replacing `<ip_of_host>` with the IP of the host in which the container is running and `<key>` with the key you created in OBS Studio. For example: `rtmp://192.168.0.30/live/test`
* Click "Play"
* Now VLC should start playing whatever you are transmitting from OBS Studio

## Debugging

If something is not working you can check the logs of the container with:

```bash
docker logs nginx-rtmp
```

## Extending

If you need to modify the configurations you can create a file `nginx.conf` and replace the one in this image using a `Dockerfile` that is based on the image, for example:

```Dockerfile
FROM tiangolo/nginx-rtmp

COPY nginx.conf /etc/nginx/nginx.conf
```

The current `nginx.conf` contains:

```Nginx
worker_processes auto;
rtmp_auto_push on;
events {}
rtmp {
    server {
        listen 1935;
        listen [::]:1935 ipv6only=on;

        application live {
            live on;
            record off;
        }
    }
}
```

You can start from it and modify it as you need. Here's the [documentation related to `nginx-rtmp-module`](https://github.com/arut/nginx-rtmp-module/wiki/Directives).

## Technical details

* This image is built from the same base official images that most of the other official images, as Python, Node, Postgres, Nginx itself, etc. Specifically, [buildpack-deps](https://hub.docker.com/_/buildpack-deps/) which is in turn based on [debian](https://hub.docker.com/_/debian/). So, if you have any other image locally you probably have the base image layers already downloaded.

* It is built from the official sources of **Nginx** and **nginx-rtmp-module** without adding anything else. (Surprisingly, most of the available images that include **nginx-rtmp-module** are made from different sources, old versions or add several other components).

* It has a simple default configuration that should allow you to send one or more streams to it and have several clients receiving multiple copies of those streams simultaneously. (It includes `rtmp_auto_push` and an automatic number of worker processes).

## Release Notes

### Latest Changes

* 👷 Add GitHub Action for Docker Hub description. PR [#45](https://github.com/tiangolo/nginx-rtmp-docker/pull/45) by [@tiangolo](https://github.com/tiangolo).
* Bump tiangolo/issue-manager from 0.3.0 to 0.4.0. PR [#42](https://github.com/tiangolo/nginx-rtmp-docker/pull/42) by [@dependabot[bot]](https://github.com/apps/dependabot).
* Bump actions/checkout from 2 to 3. PR [#43](https://github.com/tiangolo/nginx-rtmp-docker/pull/43) by [@dependabot[bot]](https://github.com/apps/dependabot).
* 🎨 Format CI config. PR [#44](https://github.com/tiangolo/nginx-rtmp-docker/pull/44) by [@tiangolo](https://github.com/tiangolo).
* 👷 Add Dependabot and funding configs. PR [#41](https://github.com/tiangolo/nginx-rtmp-docker/pull/41) by [@tiangolo](https://github.com/tiangolo).
* ✨ Allow using debug directives, enable ` --with-debug` compile option. PR [#16](https://github.com/tiangolo/nginx-rtmp-docker/pull/16) by [@agconti](https://github.com/agconti).
* ⬆️ Upgrade Nginx to 1.23.2 and OS to bullseye. PR [#40](https://github.com/tiangolo/nginx-rtmp-docker/pull/40) by [@tiangolo](https://github.com/tiangolo).
* ⬆ Upgrade to nginx-1.19.7. PR [#26](https://github.com/tiangolo/nginx-rtmp-docker/pull/26) by [@cesarandreslopez](https://github.com/cesarandreslopez).
* 👷 Add scheduled CI. PR [#39](https://github.com/tiangolo/nginx-rtmp-docker/pull/39) by [@tiangolo](https://github.com/tiangolo).
* ⬆ Update RTMP module version to 1.2.2. PR [#28](https://github.com/tiangolo/nginx-rtmp-docker/pull/28) by [@louis70109](https://github.com/louis70109).
* 👷 Add alls-green GitHub Action. PR [#38](https://github.com/tiangolo/nginx-rtmp-docker/pull/38) by [@tiangolo](https://github.com/tiangolo).
* 👷 Build to test on CI for PRs, update GitHub Actions. PR [#37](https://github.com/tiangolo/nginx-rtmp-docker/pull/37) by [@tiangolo](https://github.com/tiangolo).
* ✏️ Fix a typo in README. PR [#20](https://github.com/tiangolo/nginx-rtmp-docker/pull/20) by [@Irishsmurf](https://github.com/Irishsmurf).
* 👷 Add Latest Changes GitHub Action. PR [#29](https://github.com/tiangolo/nginx-rtmp-docker/pull/29) by [@tiangolo](https://github.com/tiangolo).
* Add CI with GitHub actions. PR [#15](https://github.com/tiangolo/nginx-rtmp-docker/pull/15).
* Upgrade Nginx to version 1.18.0. PR [#13](https://github.com/tiangolo/nginx-rtmp-docker/pull/13) by [@Nathanael-Mtd](https://github.com/Nathanael-Mtd).

## License

This project is licensed under the terms of the MIT License.
