# 🐟 Fisheye Camera Undistortion Toolkit
![Calibration Example](https://github.com/L42ARO/Fisheye-Calibration/blob/main/media/Recording2026-05-05182901-ezgif.com-video-to-gif-converter.gif)
![Undistorted Example](https://github.com/L42ARO/Fisheye-Calibration/blob/main/media/Recording2026-05-05072724-ezgif.com-video-to-gif-converter.gif)

I made this because I was trying to get usable undistorted video from a wide-angle Amazon fisheye camera, there are plenty of other solutoins out there but figured to give it a shot at making my own and hey it worked out, and specifically for wide angle cameras so figured to put it out there if it helps anyone.

This was the camera I used, but as long as you perform the calibration well any other wide angled camera should also work.
https://www.amazon.com/dp/B07ZS75KZR

This is the website I used for my checkerboard in case you want to make your own:
https://calib.io/pages/camera-calibration-pattern-generator

---

## What this repo does

- Lists available USB cameras
- Records raw fisheye test videos
- Calibrates a fisheye / wide-angle camera using a ChArUco board
- Undistorts recorded videos
- Re-encodes the result into a web-compatible H.264 MP4

There is also a test video included in `test_videos` if you just want to try the undistortion script without recording your own video.

---

## Setup

This is mainly intended for Windows users.

Run the activation/setup script:

```powershell
.\activate_venv.ps1
```

That should create/activate the Python environment and install the needed packages.


You should also have `ffmpeg` installed and available in your PATH if you want the final undistorted video to be broadly accepted by websites and video players.

---

## 1. List cameras

Use this first if you do not know your camera index:

```powershell
python list-cameras.py
```

---

## 2. Record a test video

```powershell
python record-video.py
```

By default, this records from `DEVICE_ID = 1` at `640x480` and saves the output into `test_videos`. 


---

## 3. Calibrate the camera

Print the included ChArUco board PDF, or generate your own from calib.io.

Then run:

```powershell
python fisheye_calibration.py
```

Optional audio guide:

```powershell
python fisheye_calibration.py --audio
```

You can also choose a camera index:

```powershell
python fisheye_calibration.py --device 1
```

And choose where to save the calibration file:

```powershell
python fisheye_calibration.py --output fisheye_calib_data.npz
```

The script walks through six capture phases: center flat, center tilted, image edges, image corners, close range, and far/mixed range. It aims for 60 captures total, with 10 per phase. 

---

## 4. Undistort a video

```powershell
python fisheye_undistort.py test_videos\your_video.mp4
```

This will create an output file next to the input video, usually named something like:

```txt
your_video_undistorted_h264.mp4
```

You can also choose the output path:

```powershell
python fisheye_undistort.py test_videos\your_video.mp4 -o output\undistorted.mp4
```

Use a specific calibration file:

```powershell
python fisheye_undistort.py test_videos\your_video.mp4 -c fisheye_calib_data.npz
```

Disable the preview window:

```powershell
python fisheye_undistort.py test_videos\your_video.mp4 --no-preview
```

The undistortion script writes a temporary AVI first, then uses `ffmpeg` to re-encode the final video as H.264 with `yuv420p` pixel format and `+faststart`, which makes the MP4 much more compatible with websites and players. 

---

## Useful parameters to change

### In `fisheye_calibration.py`

Camera index:

```python
DEVICE_ID = 1
```

Board settings:

```python
SQUARES_X = 5
SQUARES_Y = 7
SQUARE_LENGTH = 0.030
MARKER_LENGTH = 0.022
```

These must match your printed ChArUco board. The values are the number of chessboard squares, not inner corners. 

Capture settings:

```python
TARGET_CAPTURES = 60
COOLDOWN_TIME = 2.0
MIN_CHARUCO_CORNERS = 15
```

Use more captures if your undistortion looks bad. For fisheye lenses, edge and corner coverage matters a lot.

### In `record-video.py`

```python
DEVICE_ID = 1
WIDTH = 640
HEIGHT = 480
FPS = 30
```

Change these if your camera uses a different index or supports a different resolution/FPS. 

### In `fisheye_undistort.py`

```powershell
--balance 0.0
```

`0.0` gives a more cropped image with fewer black borders.
`1.0` keeps more field of view but may show more black edges.

```powershell
--scale 1.0
```

Adjust this if you want to experiment with the final field of view.

---

## Notes

Calibration quality depends heavily on the poses you capture. Try to follow the guide it tells you how to get very varied shots.

If the undistorted video looks worse, the calibration data is probably weak. Re-run calibration and make sure the board appears in many different parts of the image.

That is basically it. Fisheye camera gets less fishy.

