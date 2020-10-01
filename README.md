# FIRST Chesapeake RTMP Docker Server

**Docker image for video streaming server that supports RTMP, HLS, and DASH streams.**
**Designed to facilitate the streaming of remote events for [FIRST Chesapeake](https://firstchesapeake.org)**


[![Docker Automated build](https://img.shields.io/docker/cloud/automated/mglennon/chs-rtmp-server.svg)](https://hub.docker.com/r/mglennon/chs-rtmp-server/builds/)
[![Build Status](https://img.shields.io/docker/cloud/build/mglennon/chs-rtmp-server.svg)](https://hub.docker.com/r/mglennon/chs-rtmp-server)

## Description

This Docker image can be used to create a video streaming server that supports [**RTMP**](https://en.wikipedia.org/wiki/Real-Time_Messaging_Protocol), [**HLS**](https://en.wikipedia.org/wiki/HTTP_Live_Streaming), [**DASH**](https://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP) out of the box. 
It also allows adaptive streaming and custom transcoding of video streams.
All modules are built from source on Debian and Alpine Linux base images.

## Features
 * The backend is [**Nginx**](http://nginx.org/en/) with [**nginx-rtmp-module**](https://github.com/arut/nginx-rtmp-module).
 * [**FFmpeg**](https://www.ffmpeg.org/) for transcoding and adaptive streaming.
 * Default settings: 
	* RTMP is ON
	* HLS is ON (adaptive, 5 variants)
	* DASH is ON 
	* Other Nginx configuration files are also provided to allow for RTMP-only streams or no-FFmpeg transcoding. 
 * Statistic page of RTMP streams at `http://<server ip>:<server port>/stats`.
 * Available web video players (based on [video.js](https://videojs.com/) and [hls.js](https://github.com/video-dev/hls.js/)) at `/usr/local/nginx/html/players`. 

Current Image is built using:
 * Nginx 1.18.0 (compiled from source)
 * Nginx-rtmp-module 1.2.1 (compiled from source)
 * FFmpeg 4.2.3 (compiled from source)

This image was inspired by similar docker images from [tiangolo](https://hub.docker.com/r/tiangolo/nginx-rtmp/) and [alfg](https://hub.docker.com/r/alfg/nginx-rtmp/).
It was forked from [TareqAlqutami](https://github.com/TareqAlqutami/rtmp-hls-server). It has small build size, adds support for HTTP-based streams and adaptive streaming using FFmpeg.

## Usage

### To run the server
```
docker run -d -p 1935:1935 -p 8080:8080 mglennon/chs-rtmp-server
```

For Alpine-based Image use:
```
docker run -d -p 1935:1935 -p 8080:8080 mglennon/chs-rtmp-server:latest-alpine
```

To run with custom conf file:
```
docker run -d -p 1935:1935 -p 8080:8080 -v custom.conf:/etc/nginx/nginx.conf mglennon/chs-rtmp-server
```
where `custom.conf` is the new conf file for Nginx.

### To stream to the server
 * **Stream live RTMP content to:**
	```
	rtmp://<server ip>:1935/live/<stream_key>
	```
	where `<stream_key>` is any stream key you specify.

 * **Configure [OBS](https://obsproject.com/) to stream content:** <br />
Go to Settings > Stream, choose the following settings:
   * Service: Custom Streaming Server.
   * Server: `rtmp://<server ip>:1935/live`. 
   * Stream key: anything you want, however provided video players assume stream key is `test`

### To view the stream
 * **Using [VLC](https://www.videolan.org/vlc/index.html):**
	 * Go to Media > Open Network Stream.
	 * Enter the streaming URL: `rtmp://<server ip>:1935/live/<stream-key>`
	   Replace `<server ip>` with the IP of where the server is running, and
	   `<stream-key>` with the stream key you used when setting up the stream.
	 * For HLS and DASH, the URLs are of the forms: 
	 `http://<server ip>:8080/hls/<stream-key>.m3u8` and 
	 `http://<server ip>:8080/dash/<stream-key>_src.mpd` respectively.
	 * Click Play.

* **Using provided web players:** <br/>
The provided demo players assume the stream-key is called `test` and the player is opened in localhost. 
	* To play RTMP content (requires Flash): `http://localhost:8080/players/rtmp.html` 
	* To play HLS content: `http://localhost:8080/players/hls.html`
	* To play HLS content using hls.js library: `http://localhost:8080/players/hls_hlsjs.html`
	* To play DASH content: `http://localhost:8080/players/dash.html`
	* To play RTMP and HLS contents on the same page: `http://localhost:8080/players/rtmp_hls.html`

	**Notes:** 

	* These web players are hardcoded to play stream key "test" at localhost.
	* To change the stream source for these players. Download the html files and modify the `src` attribute in the video tag in the html file. You can then mount the modified files to the container as follows:
		```
		docker run -d -p 1935:1935 -p 8080:8080 -v custom_players:/usr/local/nginx/html/players mglennon/chs-rtmp-server
		```
		where `custom_players` is the directory holding the modified html files.

## Copyright
Released under MIT license.

## More info
 * **GitHub repo**: <https://github.com/mglennon/chs-rtmp-server.git>

 * **Docker Hub image**: <https://hub.docker.com/r/mglennon/chs-rtmp-server>
