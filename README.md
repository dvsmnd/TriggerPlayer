# TriggerPlayer
A html program that runs in the browser that triggers video/audio through Arduino sensor values
 
A single-file web interface that turns Arduino sensor readings into video and audio playback. It was built for teaching: students program the Arduino, and this page handles the media side, with no extra software, no installation and no need to learn TouchDesigner, Max or similar tools first.
 
Open `sensor-trigger.html` in Chrome or Edge, connect your Arduino, set a threshold per sensor, choose a video and/or audio file, and you're done.
 
## Features
 
- **Connect to any Arduino** through the browser's built-in serial port picker (Web Serial API)
- **Multiple sensors on one serial line**, up to 8, each with its own card
- **Per-sensor settings**
  - threshold, with an adjustable value range
  - trigger when the value goes above or below the threshold
  - play video, audio or both
  - its own video file and its own audio file
  - cooldown, loop, ignore-while-playing, stop-when-value-leaves-the-zone
- **Live feedback**: large value readout, level meter, threshold marker, shared history graph and a raw serial log
- **Simulate values** with a slider, so you can demo or debug without any hardware
- **Exhibition mode**: fullscreen video, hidden cursor, screen kept awake, and on-site threshold tuning with the keyboard
- **Automatic reconnect** to the last port, and settings remembered between sessions
- **Clear error messages** when a file can't be played, including a ready-made `ffmpeg` command to convert it
- **No dependencies**: one HTML file, works offline
## Requirements
 
- A desktop computer with **Google Chrome** or **Microsoft Edge** (Firefox and Safari don't support Web Serial)
- An Arduino (or any board that prints values over serial)
- Media files your browser can play (see [Supported formats](#supported-formats))
## Quick start
 
1. Download `sensor-trigger.html`.
2. Upload the example sketch below to your Arduino.
3. Close the Arduino IDE's Serial Monitor and Serial Plotter, because only one program can use the port at a time.
4. Open `sensor-trigger.html` in Chrome or Edge. Opening the file directly works. If you host it, it must be served over HTTPS or from `localhost`.
5. Choose the baud rate that matches your sketch and click **Connect**.
6. On each sensor card, choose a video and/or audio file and set the threshold.
7. Move the sensor past the threshold.
## Arduino sketch format
 
Print all sensor values on **one line**, separated by commas, spaces, semicolons or colons, and end the line with `println`:
 
```cpp
void setup() {
  Serial.begin(115200);
}
 
void loop() {
  Serial.print(analogRead(A0));
  Serial.print(",");
  Serial.print(analogRead(A1));
  Serial.print(",");
  Serial.println(analogRead(A2));
  delay(20);
}
```
 
The first value controls sensor 1, the second sensor 2, and so on. A single sensor works the same way, with one number per line. Cards are added automatically when the page sees more values than it has cards. You can also set the number of sensors by hand.
 
## Using the interface
 
### Connection
 
| Setting | What it does |
| --- | --- |
| Baud rate | Must match `Serial.begin()` in your sketch |
| Number of sensors | How many sensor cards to show (1 to 8) |
| Reconnect automatically | Reopens the last used port on page load and after the cable is unplugged and replugged |
 
### Per-sensor settings
 
| Setting | What it does |
| --- | --- |
| Threshold | The value that triggers playback |
| Range min / max | The expected value range, which sets the slider, meter and graph scale (default 0 to 1023) |
| Trigger when | Above or below the threshold |
| Play | Video, audio or both |
| Video file / Audio file | The media this sensor triggers |
| Cooldown | Minimum time in milliseconds before the same sensor can trigger again |
| Ignore while this sensor's media plays | Prevents restarting while its media is still playing |
| Loop | Loops the media |
| Stop when the value leaves the zone | Stops playback when the value goes back across the threshold |
| Simulate a value | Fakes a sensor value for testing without hardware |
 
Playback starts once when the value crosses the threshold, not continuously while it stays beyond it.
 
### Sharing one screen
 
- **Audio** from different sensors can play at the same time.
- **Video** uses one shared screen. You choose whether a new trigger switches to the new video or is ignored while another video plays.
- You also choose whether the screen goes black between videos or keeps the final frame of the last video.
### Exhibition mode
 
Click **Start exhibition mode** to take the video fullscreen. The Arduino connection and all triggers keep working.
 
| Key | Action |
| --- | --- |
| `Esc` | Leave exhibition mode |
| `D` | Show or hide a small overlay with all values and thresholds |
| `1` to `8` | Select a sensor |
| `↑` / `↓` | Raise or lower the selected sensor's threshold (hold `Shift` for bigger steps) |
 
The mouse cursor hides after two seconds, and the page requests a screen wake lock so the display doesn't sleep.
 
## Supported formats
 
A browser can play a file only if it supports the **codec** inside it. The file extension doesn't matter. A `.mov` or `.mkv` file may play fine or not at all, depending on what is inside.
 
Safe choices that work in Chrome and Edge:
 
| Type | Recommended |
| --- | --- |
| Video | `.mp4` with H.264 video and AAC audio (also `.webm`) |
| Audio | `.mp3`, `.m4a` (AAC), `.wav`, `.ogg`, `.flac` |
 
If a file can't be played, the page shows an error and a conversion command. You need [ffmpeg](https://ffmpeg.org) installed.
 
Convert video to a universally playable `.mp4`:
 
```bash
ffmpeg -i input.mov -c:v libx264 -pix_fmt yuv420p -crf 20 -c:a aac -movflags +faststart output.mp4
```
 
Convert audio to AAC:
 
```bash
ffmpeg -i input.aif -c:a aac -b:a 192k output.m4a
```
 
The "What can this browser play?" section in the interface lists which formats your browser supports.
 
## Troubleshooting
 
| Problem | Likely cause and fix |
| --- | --- |
| "Web Serial isn't available" banner | Use Chrome or Edge on a desktop computer |
| The port can't be opened | Close the Arduino Serial Monitor, Serial Plotter and any other program using the port |
| Garbled text in the serial log | The baud rate in the page doesn't match `Serial.begin()` |
| Nothing is triggered | Check the threshold and range, and whether you chose "above" or "below". Use the simulate slider to test the page without hardware |
| "Video could not start: NotAllowedError" | The browser blocks autoplay until you interact with the page. Click anywhere on the page once and try again |
| Video error 4, or audio but no picture | The codec isn't supported. Convert the file as described above |
| Media files are gone after a reload | Browsers don't allow re-opening local files automatically. Choose the files again. Thresholds and other settings are kept |
 
## Project structure
 
```
sensor-trigger.html   the whole application (HTML, CSS and JavaScript)
README.md             this file
LICENSE               MIT license
```
 
## Browser support
 
| Browser | Supported |
| --- | --- |
| Chrome (desktop) | Yes |
| Edge (desktop) | Yes |
| Opera (desktop) | Should work (Chromium-based, untested) |
| Firefox | No (no Web Serial API) |
| Safari | No (no Web Serial API) |
| Mobile browsers | No |
 
## Contributing
 
Issues and pull requests are welcome. The project is intentionally a single file with no build step, so please keep it that way. It should stay easy for students to open, read and change.
 
## License
 
This project is released under the [MIT License](LICENSE). You are free to use, copy, modify and distribute it, including for teaching and commercial work, as long as the copyright notice is kept.
 
