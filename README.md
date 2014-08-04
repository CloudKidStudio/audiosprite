[![Build Status](https://secure.travis-ci.org/tonistiigi/audiosprite.png)](http://travis-ci.org/tonistiigi/audiosprite)

This is a `ffmpeg` wrapper that will take in **multiple audio files** and combines them **into a single file**. Silent gaps will be put between the parts so that every new part starts from full second and there is at least 1 second pause between every part. The final file will be exported in `mp3`, `ogg`, `ac3`, `m4a` and `caf`(IMA-ADPCM) to support as many devices as possible. This tool will also generate a `JSON` file that is compatible with [SwishSprite](https://github.com/CloudKidStudio/SwishSprite).

## Purpose

iOS, Windows Phone and some Android phones have very limited HTML5 audio support. They only support playing single file at a time and loading in new files requires user interaction and has a big latency. To overcome this there is a technique to combine all audio into single file and only play/loop certain parts of that file.

## Installation

```
npm install -g audiosprite
```

### Hints for Windows users

- You need to install [Node.js](https://www.nodejs.org/)
- Use [Git Bash](http://git-scm.com/download/win) instead of Command Line or Powershell
- Download [ffmpeg](http://ffmpeg.zeranoe.com/builds/) and include it in your path `export PATH=$PATH:path/to/ffmpeg/bin`
- IMA-ADPCM(the fastest iPhone format) will only be generated if you are using OSX.

## Usage

```
> audiosprite --help
info: Usage: audiosprite [options] file1.mp3 file2.mp3 *.wav
info: Options:
  --output, -o      Name for the output file.                                    [default: "output"]
  --priority, -y    The JSON list of audio aliases by priority.                  [default: ""]
  --export, -e      Limit exported file types. Comma separated extension list.   [default: ""]
  --log, -l         Log level (debug, info, notice, warning, error).             [default: "info"]
  --autoplay, -a    Autoplay sprite name                                         [default: null]
  --silence, -s     Add special "silence" track with specified duration.         [default: 0]
  --samplerate, -r  Sample rate.                                                 [default: 44100]
  --channels, -c    Number of channels (1=mono, 2=stereo).                       [default: 1]
  --rawparts, -p    Include raw slices(for Web Audio API) in specified formats.  [default: ""]
  --bitrate, -b     The bitrate quality for the exported audio.                  [default: 128]
  --help, -h        Show this help message.


> audiosprite --autoplay bg_loop --output mygameaudio bg_loop.wav *.mp3
info: File added OK file=bg_loop.wav
info: 1.25s silence gap added OK
info: File added OK file=click.mp3
info: 1.70s silence gap added OK
info: Exported caf OK file=mygameaudio.caf
info: Exported ac3 OK file=mygameaudio.ac3
info: Exported mp3 OK file=mygameaudio.mp3
info: Exported m4a OK file=mygameaudio.m4a
info: Exported ogg OK file=mygameaudio.ogg
info: Exported json OK file=mygameaudio.json
info: All done


> cat mygameaudio.json
{
  "resources": [
    "mygameaudio.caf",
    "mygameaudio.ac3",
    "mygameaudio.mp3",
    "mygameaudio.m4a",
    "mygameaudio.ogg"
  ],
  "spritemap": {
    "bg_loop": {
      "start": 0,
      "end": 3.75,
      "loop": true,
      "priority": 0
    },
    "click": {
      "start": 5,
      "end": 5.3,
      "loop": false,
      "priority": 0
    }
  },
  "autoplay": "bg_loop"
}
```

### Setting autoplay track

You can use `--autoplay` option to set a track that will start playing automatically. This track is then marked as autoplay and looping in the JSON. This syntax is Jukebox framework specific.

### Custom silent track

On some cases starting and pausing a file has bigger latency than just setting playhead position. You may get better results if your file is always playing. `--silence <duration>` option will generate extra track named *silence* that you can play instead of pausing the file.

### Usage with [Swish Sprite](https://github.com/CloudKidStudio/SwishSprite)

Generated JSON file can be passed straight into `cloudkid.SwishSprite` constructor. Check out [Swish Sprite documentation](http://cloudkidstudio.github.io/SwishSprite/) more info.

```js
// Load the audiosprite map JSON with jQuery
$.getJSON("sprite.json", function(data){

	// Create new SwishSprite from map
	var audio = new cloudkid.SwishSprite(data);
	
	
	// Trigger a user load on a button press
	// for mobile browser that require a user's
	// input before initiating the audio load
	$('button#load').click(function(){
		audio.load();
	});
	
	// Add event for when audio finishes loading
	audio.on('loaded', function(){
		audio.play('bg_loop');
	});
});
```
