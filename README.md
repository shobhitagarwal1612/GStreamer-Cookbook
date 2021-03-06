## GStreamer Cookbook

The GStreamer API is difficult to work with. Remember, data in GStreamer flows through pipelines quite analogous to the way water flows through pipes.

This repository is a collection of C snippets and commandline pipelines using the GStreamer 1.0 API to perform video operations. 

## Capturing & Recording
- The webcam (v4l2src) as the input stream
- Autovideosink is used to display. In these examples, autovideosink is preceded by videoconvert.
- Recording is H.264 encoded using the x264enc element.
- Mux the recording with mp4mux to make the output an mp4 file.

#### Let's display a video
`gst-launch-1.0 v4l2src ! videoconvert ! autovideosink`

The most basic pipeline to display video from webcam to a window on the screen.


#### Now let's record a video
`gst-launch-1.0 v4l2src ! x264enc ! mp4mux ! filesink location=/home/xyz/Desktop/recorded.mp4 -e`

A basic pipeline to record video from webcam to a file on specified location. The `-e` tage instructs GStreamer to flush EoS(End of Stream) before closing the recorded stream. This allows proper closing of the saved file. 

[Implementation of recording a video](/C/simple-recording.c).

#### Display Recorded Video
`gst-launch-1.0 filesrc location=/home/xyz/Desktop/recorded.mp4 ! decodebin ! videoconvert ! autovideosink`

A pipeline to display the file saved using the above pipeline. This uses the element `decodebin` which is a super-powerful all-knowing decoder. It decodes any stream of a legal format to the format required by the next element in the pipeline, automagically. 

#### Display + Record Video Together

`gst-launch-1.0 v4l2src ! tee name=t t. ! queue ! x264enc ! mp4mux ! filesink location=/home/rish/Desktop/okay.264 t. ! queue ! videoconvert ! autovideosink`

A pipeline to display and record the incoming webcam stream. This uses another powerful element called `tee`. The `tee` can be thought of as a T-joint, splitting the source into two or more sub-pipes. Remember, you must put a `queue` element after each branch to provide separate threads to each branch.

[Implementation of recording + displaying a video using tee](/C/tee-recording-and-display.c).

#### Start/Stop Recording at will

Here comes one of the more difficult parts of GStreamer. Dynamic pipelines. These are useful in cases where you would like to alwasy display the stream, but record at will (say on the click of a button). You cannot use a command line pipeline for this.

[Implementation of dynamic pipelines in C](/C/dynamic-recording.c).

Use Ctrl+C to start and stop streaming. Every time you start a new recording, a fresh file is created with a new name.