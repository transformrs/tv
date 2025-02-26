# trv

Transform slides and speaker notes into video.

## Installation

```sh
cargo install trv
```

Or with [`cargo binstall`](https://github.com/cargo-bins/cargo-binstall):

```sh
cargo binstall trv
```

## Usage

This tool is designed to work with [Typst](https://github.com/typst/typst) presentations.
Typst is a new typesetting system that is similar to LaTeX.
To create a video, create a Typst presentation with speaker notes (we show only the first slide here):

```typ
#import "@preview/polylux:0.4.0": *

#set page(paper: "presentation-16-9")

#set text(size: 25pt)

#slide[
    #toolbox.pdfpc.speaker-note(
    ```md
    What if you could easily generate videos from text?
    ```
    )
    #set page(fill: black)
    #set text(fill: white)
    #v(6em)
    #set text(size: 35pt)
    #align(center)[*Text to video*]
]
```

Next, run the following command:

```raw
$ trv --input=examples/first.typ \
    --provider='openai-compatible(kokoros.transformrs.org)' \
    --model=tts-1 \
    --voice=af_sky \
    --speed=0.95 \
    --audio-format=wav \
    --release
```

This generates the following video:

[![First demo video](https://transformrs.github.io/trv/first.png)](https://transformrs.github.io/trv/first.mp4)

## Offline

To create a video without an API key nor an internet connection, you can self-host [Kokoros](https://github.com/lucasjinreal/Kokoros).
See the [Kokoros section](#kokoros) for more information.

## Via DeepInfra

For more voices and faster audio generation, you can use the Kokoro models hosted at DeepInfra.

```raw
$ export DEEPINFRA_KEY="<YOUR KEY>"

$ trv --input=presentation.typ
 INFO Generating audio file for slide 0
 INFO Generating audio file for slide 1
 INFO Creating video clip _out/1.mp4
 INFO Created video clip _out/1.mp4
 INFO Creating video clip _out/2.mp4
 INFO Created video clip _out/2.mp4
 INFO Concatenated video clips into _out/out.mp4
```


To create a video without an API key nor an internet connection, you can self-host [Kokoros](https://github.com/lucasjinreal/Kokoros).
See the [Offline section](#offline) for more information.
Or for a state-of-the-art model with voice cloning capabilities, see the [Zyphra Zonos section](#zyphra-zonos).

## Offline

To use Kokoros locally, the easiest way is to use the Docker image.

```raw
$ git clone https://github.com/lucasjinreal/Kokoros.git

$ cd Kokoros/

$ docker build --rm -t kokoros .

$ docker run -it --rm -p 3000:3000 kokoros openai
```

Then, you can use the Docker image as the provider:

```raw
$ trv --input=presentation.typ --provider=openai-compatible(localhost:3000)
```

## Via Google

Google has some high-quality voices available via their API:

```raw
$ export GOOGLE_KEY="<YOUR KEY>"

$ trv --input=examples/google.typ \
    --provider=google \
    --voice=en-US-Chirp-HD-D \
    --language-code=en-US \
    --release
```

[![Google demo video](https://transformrs.github.io/trv/google.png)](https://transformrs.github.io/trv/google.mp4)

See the [Google section](#google) for more information about the Google API.

Google, meanwhile, has the best text-to-speech engine that I've found as part of Gemini 2.0 Flash Experimental.
However, audio output is not yet available via the API.

## Zyphra Zonos

To use the Zyphra Zonos model, you need 8 GB of VRAM.
So it's probably easiest to use DeepInfra:

```raw
$ export DEEPINFRA_KEY="<YOUR KEY>"

$ trv --input=presentation.typ \
    --model='Zyphra/Zonos-v0.1-hybrid' \
    --voice='american_male' \
    --release
```

Do note that Zonos is way more unstable than Kokoros at the time of writing.
For example, sometimes it will add random sounds like the sound of swallowing.
So in practice, the Kokoro model is probably the better option for now.

## Portrait Video

To create a portait video, like a YouTube Short, you can set the page to

```typst
#set page(width: 259.2pt, height: 460.8pt)
```

The rest should work as usual.
This will automatically create slides with 1080 x 1920 resolution since Typst is set to 300 DPI.
Next, ffmpeg will automatically scale the video to a height of 1920p so in this case the height will not be changed.
For landscape videos, it might scale the image down to 1920p.

## About Audio

Audio is generated using the [transformrs](https://github.com/transformrs/transformrs) crate.
It supports multiple providers, including DeepInfra, OpenAI, and Google.

So `trv` should also work with providers other than DeepInfra.
However, during testing, I got the best results with Kokoros or DeepInfra for the lowest price.

For example, OpenAI text-to-speech requires any video to contain a "clear disclosure" that the voice they are hearing is AI-generated.
