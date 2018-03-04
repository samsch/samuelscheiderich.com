---
title: Using the Web Audio API
date: 2018-03-04 11:00:29
tags:
- js
- Web API
---
Have you ever wanted to make some noise in your browser? I wanted to generate some sound effects, and it made me curious about what tools are available in the browser.

What I found is the [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API). I'm going to show how to play basic sounds, and lay the groundwork for building more advanced sound generators.

> All examples can be directly pasted into [jsfiddle](https://jsfiddle.net/) and run. The first couple are just JavaScript, but the others use a short bit of HTML. I'm running these in Firefox, but Chrome and Edge should also work fine. Webkit-based browsers may not work, and IE has no support.

The WebAudio API is built around a "node" system. It somewhat parallels how you would connect sound modules or effects pedals together in real life.

The base object you need to work with is the [AudioContext](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext), created with `new AudioContext()` (or `new webkitAudioContext()` for Webkit-based browsers). This context object has factory methods to create the other nodes you use.

## Basic sound output

Here's how you output a simple tone (**Warning**, this is rather loud if run directly):
```js
const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const oscillator = audioContext.createOscillator();
oscillator.connect(audioContext.destination);
oscillator.frequency.setValueAtTime(440, audioContext.currentTime);
oscillator.start();
window.setTimeout(() => {
	oscillator.stop();
}, 1000);
```
We already learned about the audioContext, and the rest is fairly straightforward as well. The `audioContext.createOscillator()` creates an [OscillatorNode](https://developer.mozilla.org/en-US/docs/Web/API/OscillatorNode) instance, which generates a tone. OscillatorNodes has [AudioParam](https://developer.mozilla.org/en-US/docs/Web/API/AudioParam) properties to control frequency (directly, and via a "detune" value, which seems to be more suitable for note-based music generation) and wave-shape.

OscillatorNode implement [AudioNode](https://developer.mozilla.org/en-US/docs/Web/API/AudioNode), which is the base class. AudioNodes use the AudioParam interface for properties, which gives controls like `setValueAtTime()` used above. So an oscillator has a `frequency` property which has the AudioParam properties and methods for reading and writing. You can't write to the frequency property directly (you can-ish, because it's JavaScript, but it won't "work"), instead you read from `oscillator.frequency.value`, and you can either write to the `value` property or use the time-based methods such as `.setValueAtTime()`. Generally, using the time-based methods is better, because they will overwrite any value written to the value property directly.

To use the output of an AudioNode, you call the `.connect()` method and pass the target, which can be another AudioNode, an AudioParam (for dynamic input), or `audioContext.destination` which represents the browser's sound output (speakers, headphones). We connect our oscillator to audioContext.destination so we can hear the output.

We call `.setValueAtTime()` with `440` and `audioContext.currentTime`, which says we want a frequency of 440hz, and we want that to be applied *now*. To sets value in the future, you add time in seconds to the currentTime value.

To start the oscillator sound output we call `.start()`. After a 1 second we stop the output with `.stop()`. These are not start/pause controls like you might expect. Once you call `.stop()`, an OscillatorNode can not be re-started. For repeatedly doing something, you either need to use multiple oscillators, or use a single oscillator and disconnect it's output or set the output volume to zero using a [GainNode](https://developer.mozilla.org/en-US/docs/Web/API/GainNode) before the destination.

## Fixing the volume problem
By default, the oscillator output is rather loud. Lets fix that by adding a GainNode.
```js
const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const volume = audioContext.createGain();
volume.connect(audioContext.destination);
volume.gain.setValueAtTime(.2, audioContext.currentTime);
const oscillator = audioContext.createOscillator();
oscillator.connect(volume);
oscillator.frequency.setValueAtTime(440, audioContext.currentTime);
oscillator.start();
window.setTimeout(() => {
	oscillator.stop();
}, 1000);
```
Now, instead of connecting the oscillator to the output directly, we connect it to a GainNode which then connects to the destination. The GainNode has a gain AudioParam which takes a float value and multiplies the amplitude. So a value of 1 would be no change, .5 would be half, and 2 would be double. .2 (or 20%) makes for a fairly reasonable volume.

## Doing something more interesting
Lets make a button that plays a sound!

```html
<!-- HTML -->
<button id="play-sound">Play Sound</button>
```
```js
// JavaScript
const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const volume = audioContext.createGain();
volume.connect(audioContext.destination);
volume.gain.setValueAtTime(.2, audioContext.currentTime);

const button = document.getElementById('play-sound');

button.addEventListener('click', () => {
  const oscillator = audioContext.createOscillator();
  oscillator.connect(volume);
  oscillator.frequency.setValueAtTime(440, audioContext.currentTime);
  oscillator.start();
  window.setTimeout(() => {
    oscillator.stop();
    oscillator.disconnect();
  }, 500);
});
```
Our button click handler creates a new oscillator, connects it, sets the frequency, starts it, and then stops and disconnects it after 500ms.

This is the basis for how you play sounds on a trigger. Just create your oscillator, play it, and destroy it. Notice that if you click the button again before the sound stops, it does create a new oscillator and plays them both, which you can usually hear as a louder sound.

## Exciting sounds
So far, we've only made boring single-frequency sounds. Lets do something a bit more interesting. If we just wanted to change the pitch played on the button press, that would be as simple as changing "440" to the desired frequency. This could be set from a dynamic input as well.

More interesting would be to play a series of sounds. Lets make a varying wave-form.

The html will be the same as above, with this JavaScript:
```js
const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const volume = audioContext.createGain();
volume.connect(audioContext.destination);
volume.gain.setValueAtTime(.2, audioContext.currentTime);

const button = document.getElementById('play-sound');

const frequencyList = [];
for (let i = 0; i < 1000; i++) {
	frequencyList.push(Math.sin(i * Math.PI / 100) * 100 + 400);
}

button.addEventListener('click', () => {
  const oscillator = audioContext.createOscillator();
  oscillator.connect(volume);
  frequencyList.forEach((frequency, time) => {
  	oscillator.frequency.setValueAtTime(frequency, audioContext.currentTime + time*.001);
  });
  oscillator.start();
  window.setTimeout(() => {
    oscillator.stop();
  }, 1000);
});
```
Instead of just setting a frequency and calling it done, we run over an array of 1000 frequency values, setting them at intervals of 1ms so that we get a (to human perception) smooth variation of pitch. Any AudioParams can be set like this, so you can create arrays of frequency and volume (gain) values to play over time.

## More!
Naturally, this is just the tip of the iceberg. Besides gain and oscillators, you can do other types of filtering and effects, including reverberation, delays, and distortion; you can use microphone input and uploaded audio files; and you can even do visualizations of the waveform output. A good place to start for more in-depth study is the [MDN Web Audio API Guide](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API). Especially look through the types of AudioNodes listed in *Related pages for Web Audio API*.
