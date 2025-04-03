# continous-capture

## App Initialization
- The host OS's corresponding ffmpeg binary (which is embeded in the application) is extracted and written to a folder to be called by the application. Skip this step if the executable has already been extracted.
- By default, an attempt is made to register hotkeys alt+`num_key`, where `num_key` are keys `0` to `9`. If at any point a hotkey that was registered is pressed: take a screenshot, store the hotkey pressed, store the timestamp of screenshot (might already be a part of the image file)

## Primary Screengrab Loop
1. Start continuous screen capture of entire screen in chunks of `segment_time` seconds. Lossless encoding (preset ultrafast) to minimize processing power. If this command fails, continuously try to restart it (for example when computer goes to sleep). Default framerate is 4.
    - ./ffmpeg -f gdigrab -framerate 4 -i desktop -reset_timestamps 1 -sc_threshold 0 -g 10 -force_key_frames "expr:gte(t, n_forced * 10)" -segment_time 10 -f segment -crf 0 -preset ultrafast -y output_lossless_%d.mp4
2. Crop the screen grab chunks for the display bounds by querying display bounds. For example, with the following bounds, (0,0)-(1920,1200), (1920,58)-(3840,1138), the following commands would crop correctly:
    - ./ffmpeg -i output_lossless_0.mp4 -vf "crop=1920:1200:0:0" -preset ultrafast -y monitor_0_output_lossy_0.mp4
    - ./ffmpeg -i output_lossless_0.mp4 -vf "crop=1920:1080:1920:58" -preset ultrafast -y monitor_1_output_lossy_0.mp4
3. Delete the continuous screen grab chunks produced in step `1`
4. In a separate thread, check display bounds every 5 seconds. If display bounds have changed and stayed consistent, re-run the continuous screen capture command from step `1`

## MVP
- [ ] App initialization
- [ ] Primary screengrab loop
- GUI
    - [ ] View screengrab videos 
    - [ ] Scrub screengrab videos with arrow keys
    - [ ] Jump to when hotkeys were pressed
    - [ ] Filter for when only certain hotkeys were pressed
- User Settings
    - [ ] Hotkeys for screen capture 
    - [ ] How much video to keep (Data and Time)
    - [ ] Archival of old videos (default lossy compression)
    - [ ] Set framerate of continuous screengrab (default 4)
    - [ ] Auto start on boot (default yes)

## Future
- [ ] Feed cropped display grab chunks into OCR (maybe tesseract)
- [ ] Text/fuzzy search with OCR
- [ ] Remove duplicate frames in screen recording
- [ ] Mark the times where windows are opened/closed
- [ ] Investigate using event listener instead of checking display bounds every second. For example, the [Microsoft display change event](https://learn.microsoft.com/en-us/uwp/api/windows.devices.display.core.displaymanager.changed?view=winrt-26100).

