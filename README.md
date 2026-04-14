# Fourier Audio Lab — DFT Music Visualizer

**Live:** https://dspdft.web.app  
**GitHub:** https://github.com/Alex-web0/dft_music_visualizer_dsp

---

## Introduction

Understanding Fouriers, especially the Fourier transform, was always a difficult thing for me. Yes I did well in the exams, I memorized all the formulas, but you only get real understanding when you apply things hands on.

From all my research, I understood that the simplest usage of the Fourier transform is with music, car radios, and all sorts of editors, apart from the various vector, image, video and audio processing use cases.

In this paper we build a web based app that lets you play with one of the earliest and most common applications of the Fourier transform: audio editing.

### But what is the Discrete Fourier Transform?

In short, it is a way of taking a signal that lives in time (a sound wave going up and down as seconds pass) and rewriting it as a sum of simple sine and cosine waves, each with its own frequency and amplitude. Any complicated wave, no matter how messy, can be broken down this way. So instead of asking "what is the signal doing at each moment?", the Fourier transform lets you ask "which frequencies are inside this signal, and how strong is each one?"

For discrete sampled audio we use the Discrete Fourier Transform:

$$X[k] \;=\; \sum_{n=0}^{N-1} x[n]\, e^{-j\,2\pi k n / N}, \qquad k = 0, 1, \dots, N-1$$

And you can always go back to the original signal with the inverse:

$$x[n] \;=\; \frac{1}{N}\sum_{k=0}^{N-1} X[k]\, e^{\,j\,2\pi k n / N}$$

In practice this is computed with the Fast Fourier Transform (FFT), which is what lets phones, radios and MP3 players do it in real time.

---

## Problem Statement

The Fourier transform is taught almost entirely through formulas and static plots. Students pass the exams but never actually see or hear what a single frequency component is, or what happens when you remove one. There is a real gap between knowing the integral and understanding what it means, and the usual way of teaching it does not close that gap.

---

## Methodology

The program works in the following way:

1. **Upload** — A music audio file is uploaded. The browser decodes the compressed file (mp3, wav, etc.) into an array of raw samples `x[n]`. No math yet, just reading the file.

2. **Waveform display** — We plot `x[n]` directly on a time axis. This is the time domain view, nothing but the amplitude of the sound at each moment.

3. **FFT spectrum** — We run the FFT on the whole song, which gives us `X[k]` for every bin `k`. The magnitude `|X[k]| = sqrt(Re² + Im²)` tells us how strong each frequency is, and we plot that on a log axis because audio spans many orders of magnitude.

4. **Parameter changes & remastering** — Operations like time shift use the Fourier shift property, $x[n - n_0] \longleftrightarrow X[k]\, e^{-j\,2\pi k n_0 / N}$, so shifting in time is just multiplying each bin by a phase factor. Multiplying the input by a window or an envelope before the FFT also changes the spectrum in predictable ways (convolution theorem).

5. **Band toggling** — Unchecking a band literally zeroes out those `X[k]` values (and their mirror bins `X[N-k]` to keep the signal real), then we run the inverse FFT and get back a new `x[n]` with those frequencies gone. The waveform visibly changes and the audio plays back without the removed band.

---

## Tech Used

- **Chart.js** (CDN) — renders the waveform and FFT spectrum charts
- **MathJax** (CDN) — renders the live LaTeX math panel
- **Vanilla JavaScript** — all FFT logic runs client-side in the browser
- **Web Audio API** — decodes audio files and handles playback
- **Firebase Hosting** — static deployment

About 400 lines total including logic, HTML, and comments.

---

## Main Features

- Upload any audio file and see its waveform and frequency spectrum side by side
- Apply the FFT on the whole song and scroll through the time-domain result
- Toggle 12 frequency bands on and off to hear and see the effect in real time
- A live math panel that shows the signal reconstructed as a sum of its top dominant cosines, plus all the DFT properties used

---

## Problems Solved & Use Cases

- Closes the gap between memorizing the integral and actually understanding what a frequency component is
- Lets students hear and see Fourier concepts instead of only reading them
- Same workflow is the basis for audio equalizers, noise removal, telephone and radio effects, hearing aid tuning, and the first stages of MP3 compression

---

## Future Plans

- **Lossy compression demo** — filter out high-frequency components inaudible to humans to demonstrate the core idea behind MP3 and similar codecs
- **Discretization controls** — expose window size, sample rate, and sampling skip so students can see aliasing and windowing effects directly

---

## Attached Code — JavaScript FFT Implementation

Cooley-Tukey radix-2 in-place FFT. Pass `inverse=true` for the IFFT.

```js
function fft(re, im, inverse) {
  const n = re.length;
  // bit-reversal permutation
  for (let i = 1, j = 0; i < n; i++) {
    let bit = n >> 1;
    for (; j & bit; bit >>= 1) j ^= bit;
    j ^= bit;
    if (i < j) { [re[i], re[j]] = [re[j], re[i]]; [im[i], im[j]] = [im[j], im[i]]; }
  }
  // butterfly stages
  for (let len = 2; len <= n; len <<= 1) {
    const ang = (inverse ? 2 : -2) * Math.PI / len;
    const wRe = Math.cos(ang), wIm = Math.sin(ang);
    for (let i = 0; i < n; i += len) {
      let cRe = 1, cIm = 0;
      for (let k = 0; k < len / 2; k++) {
        const tRe = cRe * re[i + k + len / 2] - cIm * im[i + k + len / 2];
        const tIm = cRe * im[i + k + len / 2] + cIm * re[i + k + len / 2];
        re[i + k + len / 2] = re[i + k] - tRe; im[i + k + len / 2] = im[i + k] - tIm;
        re[i + k] += tRe; im[i + k] += tIm;
        const nRe = cRe * wRe - cIm * wIm; cIm = cRe * wIm + cIm * wRe; cRe = nRe;
      }
    }
  }
  if (inverse) for (let i = 0; i < n; i++) { re[i] /= n; im[i] /= n; }
}
```

---

## Conclusion

Since the Fourier transform is a difficult topic that is not understood well by students, we now give them a hands on way to understand why and how it works. You also get to hear your favorite song edited and changed like an old car radio, except you see the math behind every step.

It closes the gap between memorizing the integral and knowing what a frequency component actually is. Students can hear and see Fourier concepts instead of only reading them. The same workflow is also the basis for audio equalizers, noise removal, telephone and radio effects, hearing aid tuning, and the first stages of MP3 compression.
