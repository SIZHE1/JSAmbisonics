# JSAmbisonics
A JS library for first-order ambisonic (FOA) and higher-order ambisonic (HOA) processing for  browsers, using Web Audio.

> Archontis Politis
> archontis.politis@aalto.fi

----
## Description
JSAmbisonics is a JavaScript library that implements a set of objects for real-time spatial audio processing, using the Ambisonics framework. The objects correspond to typical ambisonic processing blocks, and internally implement Web Audio graphs for the associated operations. The library is split into 3 parts: 
* WebAudio_FOA.js: A library for first-order ambisonics. It is based on the B-format signal set and it has no extra dependencies.
* WebAudio_HOA.js: A library for higher-order ambisonics. It is based on the ACN signal set specification, with N3D normalization of the ambisonic signals. It requires the JSHlib for spherical harmonics, and the [numeric.js](http://www.numericjs.com/) JavaScript library for matrix and vector operations.
* JSHlib.js: A library that computes real spherical harmonics, the spherical harmonic transform (SHT) and its inverse, and rotations in the spherical harmonic domain. It requires the [numeric.js](http://www.numericjs.com/) JavaScript library for matrix and vector operations.

The library is a work-in-progress. At the moment, demos seem to work in Mozilla Firefox and Google Chrome (FOA only) due to different limitations of the Web Audio specification at different browsers (the issue seems to be handling of multichannel WAVE audio, that the demos use at the moment. Firefox seems to handle these without problem. This issue can be circumvented possibly by splitting the files into groups of fewer channels. €-channel files seem to have no problem.)

---
## First-order (B-format) ambisonics

The WebAudio_FOA.js implements the following objects:
* Bformat_encoder: encodes a monophonic sound source to a B-format stream, at a user-defined panning direction.
* Bformat_rotator: rotates the sound scene of a B-format stream, at user-defined yaw, pitch, roll rotation angles.
* Bformat_vmic: implements standard virtual microphones to a B-format stream, with user-defined orientation.
* Bformat_binDecoder: implements a B-format to binaural conversion, using user-defined HRTF-based filters. If these are not provided, two plain opposing cardioids are used instead.
* Bformat_analyser: implements an acoustic intensity analysis for visualization of directional information captured in the B-format stream.

---
## Higher-order (ACN/N3D) ambisonics

The WebAudio_HOA.js implements the following objects:
* HOA_encoder: encodes a monophonic sound source to a HOA stream of a set order, at a user-defined panning direction.
* HOA_rotator: rotates the sound scene of a HOA stream, at user-defined yaw, pitch, roll rotation angles.
* HOA_vmic: implements higher-order virtual microphones to a HOA stream, with user-defined orientation.
* HOA_binDecoder: implements a HOA to binaural conversion, using user-defined HRTF-based filters. If these are not provided, two plain opposing cardioids are used instead.
* HOA_order_limiter: takes a HOA stream of order N, and outputs the channel limited HOA stream of order N'<=N
* HOA_bf2acn: converts a B-format stream to an ACN/N3D HOA stream
* HOA_acn2bf: converts the first-order channels of a HOA stream to B-format
 
---
## Usage

For more detailed usage information check out the page source code of the examples below. All objects have an input node *object.in* and an output node *object.out* which are used for the connections. The number of channels expected from each input depends on the object (for example a B-format encoder expects a monophonic signal, and outputs 4 channels, a B-format binaural decoder expects 4-channel input and outpus 2 channels, etc.). Example connections:
        soundBufferPlayer.connect(BFencoder.in);
        BFencoder.out.connect(BFrotator.in);
        BFrotator.out.connect(BFbinDecoder.in);
        BFbinDecoder.connect(audioContext.destination);
which implements a graph such as:
>
> soundBufferPlayer ------------->BFencoder------------>BFrotator---------->BFbinDecoder-------------->out
>                   mono stream            BF stream             BF stream              stereo stream
and 
        soundBufferPlayer.connect(BFvmic.in);
        BFvmic.out.connect(audioContext.destination);
with a graph such as:
>
> soundBufferPlayer ------------->BFvmic------------>out
>                   BF stream           mono stream  

### FOA
To use the FOA library, include it in the body of your html code as
        <script type="text/javascript" src="WebAudio_FOA.js"></script>

** B-format encoder ** is initialized inside your JavaScript code as
        var BFencoder = new Bformat_encoder(audioContext)
where *context* is the current Web Audio context. The input stream comes from an audio node to be spatialized. 
The azimuth and elevation of the encoded source can be updated at runtime by
        BFencoder.azi = azi_value_in_degrees;
        BFencoder.elev = elev_value_in_degrees;
        BFencoder.updateGains();

** B-format rotator ** is initialized inside your JavaScript code as
        var BFrotator = new Bformat_rotator(audioContext)
where *context* is the current Web Audio context. 
The yaw (Z-axis rotation), pitch (Y-axis) and roll (X-axis) rotation angles can be updated at runtime by
        BFrotator.yaw = yaw_value_in_degrees;
        BFrotator.pitch = pitch_value_in_degrees;
        BFrotator.roll = roll_value_in_degrees;
        BFrotator.updateRotMtx();

** B-format binaural decoder ** is initialized inside your JavaScript code as
        var BFbinDecoder = new Bformat_binDecoder(audioContext)
where *context* is the current Web Audio context. If no decoding filters are passed to the decoder, an initial decoding based on two opposing cardioids is defined by default.
In case binaural decoding filters are available, they should be loaded in a multichannel (4-channel) *audioBuffer* and passed to the decoder through
        BFbinDecoder.updateFilters(audioBuffer);
Example HRTF-based filters are included in the examples.

** B-format virtual microphone ** is initialized inside your JavaScript code as
        var BFvmic = new Bformat_vmic(audioContext)
where *context* is the current Web Audio context. The virtual microphone is initialized to a cardioid pointing to the front. The orientation can be updated at runtime by
        BFvmic.azi = azi_value_in_degrees;
        BFvmic.elev = elev_value_in_degrees;
        BFvmic.updateOrientation();
The pattern of the microphone can be also updated by
        BFvmic.vmicPattern = string;
        BFvmic.updatePattern();
where *string* can be one of the following:
* "subcardioid"
* "cardioid"
* "supercardioid"
* "hypercardioid"
* "dipole"


### HOA

To use the HOA library, include in the body of your html code the following
        <script type="text/javascript" src="numeric.js"></script> 
        <script type="text/javascript" src="JSHlib.js"></script> 
        <script type="text/javascript" src="WebAudio_HOA.js"></script>

The initialization and usage of all objects is the same as for the B-format with the following differences:
1. In the initialization the *order* should be passed after the *audioContext*, which defines the HOA order and the number of channels of the HOA stream (e.g. *var HOAencoder = new HOA_encoder(audioContext, order)*).
2. The *HOA_binDecoder* now requires a multichannel *audioBuffer* for the binaural filters that has *(order+1)^2* filters (e.g. 9 filters for *order=2*, 16 filters for *order=3*, etc.). Chrome seems to have problems at the moment to decode multichannel WAVE files of that many channels, so the decoding may have to be done in Web Audio on individual or smaller WAVE files, and then combined in the same *audioBuffer*.
3. The *HOA_vmic* has as available higher-order patterns the following: {"cardioid","hypercardioid","max-rE"}, where the *max-rE* specification is the pattern that maximizes the energy vector for a given order, with good front-to-back rejection ratio. The higher-order hypercardioid has the maximum directivity factor for a given order.

The HOA code is based on the larger Matlab and Spherical Harmonic Transform libraries contributed by the author in Github. The rotation algorithm is the fast recursive one by [Ivanic and Ruedenberg](http://pubs.acs.org/doi/abs/10.1021/jp953350u?journalCode=jpchax).

## Examples

### FOA (Firefox and Chrome)
1. B-format player and binaural decoder (Chrome and Firefox). The filters are based on the author's HRTFs, so performance may vary for people.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/FOA1.WebAudio_ambisonics_Bformat_player.html
2. B-format panner and binaural decoder.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/FOA2.WebAudio_ambisonics_Bformat_panner.html
3. B-format player with rotation.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/FOA3.WebAudio_ambisonics_Bformat_rotator.html
4. B-format panner with an attached B-format intensity analyser and visualization.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/FOA4.WebAudio_ambisonics_Bformat_panner_visualizer.html
5. B-format player with an attached B-format intensity analyser and visualization.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/FOA5.WebAudio_ambisonics_Bformat_player_visualizer.html
6. B-format virtual microphone
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/FOA6.WebAudio_ambisonics_Bformat_vmic.html

### HOA (Firefox)
1. HOA player and binaural decoder. The order can be switched to show improvement from low-orders on spatial blurring and colouration.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/HOA1.WebAudio_ambisonics_HOA_player.html
2. HOA panner and binaural decoder.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/HOA2.WebAudio_ambisonics_HOA_panner.html
3. HOA player with rotation.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/HOA3.WebAudio_ambisonics_HOA_rotator.html
4. HOA player with an attached B-format intensity analyser and visualization.
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/HOA4.WebAudio_ambisonics_HOA_player_visualizer.html
5. HOA virtual microphone
> https://dl.dropboxusercontent.com/u/6300538/WebAmbi/HOA5.WebAudio_ambisonics_HOA_vmic.html

HOA recordings are made by the author in the Acoustics laboratory of Aalto University, using the Eigenmike microphone.