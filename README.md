# Soccer-cam Director

Camera-path format documentation and example data for Soccer-cam player chrome extension.

Chrome Web Store link: TODO

This repository is a public documentation and data placeholder. It does **not** contain
the Chrome extension, the player source code, or the camera-path generation pipeline.
The generator currently runs locally and has not been prepared
for public release; no hosted generation service is available.

## Try the examples

Download `director.json` or use its GitHub raw URL with the matching video below.
Each file includes its playback calibration.

| Example | YouTube video | Camera path | Sample timeline |
| --- | --- | --- | --- |
| 11v11 field | [Watch video](https://www.youtube.com/watch?v=-0_K-9SM1qw) | [director.json](examples/11v11/director.json) | 0–5998.801 seconds; 59,990 samples |
| 7v7 field | [Watch video](https://youtu.be/saqDgDwmB5w) | [director.json](examples/7v7/director.json) | 0–1799.798 seconds; 17,999 samples |

1. Open the matching YouTube video in a browser with the extension installed and WebGPU available.
2. Click the extension's **browser toolbar icon**. It enables the current video and opens the camera panel.
3. Choose **Load file…** and select the example's **director.json**, or load its raw URL.
4. Play the video. The player follows the saved camera path.
5. Use the camera-mode button to switch to manual control: drag to pan and use the mouse wheel to zoom.
6. Double-click the view, or use the camera-mode button, to return to the director path.

## Repository contents

- [Director format](docs/director-format.md): schema, coordinates, timing, interpolation, and calibration.
- `examples/`: two camera-path files with embedded playback calibration.
- [Privacy](PRIVACY.md): current player data handling and this repository's scope.

Use this repository's Issues page for documentation and example-data feedback.
