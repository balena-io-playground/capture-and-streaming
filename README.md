# capture-and-streaming
Example of using gstreamer to take a camera output and stream with RTSP, then convert that into WebRTC.

## Usage
In the Device Configuration, add the custom configuration variable `BALENA_HOST_CONFIG_start_x` and set the value to `1`. You also need to increase the default "Define device GPU memory in megabytes" using the dashboard. 192 MB seems to be the minimum, but your results may vary.

### Capture Block

The Capture Block takes a video source (usually a camera) as an input and converts it to an RTSP stream.

Input: The block will search for a Pi Camera and use that by default. If it does not find a Pi Camera, it will look for a USB camera and use the first one it finds. If the camera supports YUYV it will use that, otherwise it will use mjpeg. You can override this automatic selection process by specifying your own Gstreamer pipeline using the service variable `GST_RTSP_PIPELINE`. 

A RTSP stream will be available on `rtsp://localhost:8554/server` to other containers in the application. (Replace localhost with the device's IP address to view the stream outside the device)

### Streaming Block

The Streaming Block takes an RTSP stream as an input and produces a WebRTC stream as an output. The input stream is selected automatically but can be overriden. If a processing block is running on the device, it will use that block's RTSP output stream as an input. If no processing block is found, it will look for a capture block and use that as the input. If neither are found, or you want to override this behavior, you can specify an RTSP input stream with the service variable `WEBRTC_RTSP_INPUT`. By default, the output WebRTC stream will be on port 80, but you can change that by specifying the service variable `WEBRTC_PORT`.

The streaming block utilizes [webrtc-streamer](https://github.com/mpromonet/webrtc-streamer) so all of its features and API are available to use as well.

ssh into the streaming-block and issue the following command: 

`./webrtc-streamer rtsp://localhost:8554/server -H 0.0.0.0:80`

### Processing Block
The processing block allows you to transform an RTSP stream. It will automatically use the capture block as its input if it exists on the device. Otherwise, specify an RTSP input stream with the `PROC_RTSP_INPUT` service variable. The output of the processing block will be available on `rtsp://localhost:8558/proc` to other containers in the application. (Replace localhost with the device's IP address to view the stream outside the device.) This block is ideally suited for use with the capture and streaming blocks, and video will automatically be routed through this block if the other two are present.

An API is used to control the processing block. The block is written in python using OpenCV and it's relatively easy to add additional functionality. The base API supports:
```
/api/process
/api/flip
/api/mirror
/api/rotate
/api/negate
/api/monochrome
/api/text
```

