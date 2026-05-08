# Module 16 — People Flow Detection
### Object Tracking + Heatmap Visualization

---

## What This Project Does

This project detects and tracks people in a video, counts how many enter (IN) or exit (OUT) a defined area, and generates a heatmap showing where people moved the most.

---

## How to Run

**Step 1 — Install libraries** (run once in Colab or terminal):
```
pip install ultralytics supervision opencv-python numpy matplotlib
```

**Step 2 — Run the script:**
```
python main.py
```

**Output files generated:**
- `output_video.mp4` — video with bounding boxes, IDs, and live IN/OUT counter
- `heatmap.png` — final heatmap image showing movement intensity

---

## Detection Method

**Model used:** YOLOv8 Nano (`yolov8n.pt`)

- Pre-trained on the COCO dataset (80 object classes)
- Class `0` = person — we filter only this class
- Chosen because it is the fastest YOLOv8 variant, good for video processing
- Downloads automatically (~6MB) on first run

---

## Tracking Method

**Tracker used:** ByteTrack (via the `supervision` library)

- Assigns a unique `tracker_id` to each person
- Maintains the same ID across frames even if the person briefly disappears
- We store each person's previous Y position in a dictionary to detect direction

---

## Line Coordinates

Two horizontal lines are drawn across the video frame.

| Line | Color | Y Position | Purpose |
|------|-------|------------|---------|
| Line 1 (IN line)  | Green | 40% of frame height | Crossing DOWN = counted as IN  |
| Line 2 (OUT line) | Red   | 65% of frame height | Crossing UP   = counted as OUT |

Lines are defined as percentages of frame height so they work on any video resolution.

To get exact pixel coordinates for a different video, use:
**https://polygonzone.roboflow.com/**

---

## IN / OUT Logic Explained

OpenCV uses a coordinate system where **Y increases downward** (top of frame = y=0, bottom = y=max).

```
TOP OF FRAME (y = 0)
        |
        |  -------- LINE 1 (y = 40%) --------   ← IN LINE
        |
        |  -------- LINE 2 (y = 65%) --------   ← OUT LINE
        |
BOTTOM OF FRAME (y = max)
```

**IN condition:**
```
previous_y < LINE1_Y  AND  current_y >= LINE1_Y
```
Person was ABOVE line 1 last frame, now BELOW it → moved DOWN → counted as IN

**OUT condition:**
```
previous_y > LINE2_Y  AND  current_y <= LINE2_Y
```
Person was BELOW line 2 last frame, now ABOVE it → moved UP → counted as OUT

**Double-count prevention:**
Each `tracker_id` is added to a set (`already_counted_in` / `already_counted_out`) after being counted. Future frames skip IDs already in these sets.

---

## Heatmap Method

1. A blank numpy array (same size as video frame) starts at zero
2. Every frame, for each detected person, we add `+1` at their bounding box center position
3. After all frames, we apply `GaussianBlur` to turn sharp dots into smooth blobs
4. `cv2.normalize` scales values to 0–255
5. `cv2.applyColorMap(COLORMAP_JET)` colors it: **blue = low traffic, red = high traffic**

---

## File Structure

```
module16/
├── main.py             ← main Python script
├── README.md           ← this file
├── output_video.mp4    ← generated output video
└── heatmap.png         ← generated heatmap image
├── heatmap_display.png  
└── requirements.txt   
```

---

## Video Source

```
https://media.roboflow.com/supervision/video-examples/people-walking.mp4
```
Provided by Roboflow for educational use.
