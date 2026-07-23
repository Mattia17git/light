# Light

Light is a small scripting language with Luau-like syntax and a built-in 2D graphics and audio engine (backed by raylib).

## Install (Windows)

Run `LightSetup.exe` and follow the prompts. It installs `light.exe` into `%LOCALAPPDATA%\Light\bin` and adds it to your user PATH.

Open a **new** terminal window afterward, then run any script with:

```
light yourfile.light
```

## Language

```lua
local x = 5
local name = "world"

local function greet(who)
    return "hello " .. who
end

print(greet(name))

if x > 3 then
    print("big")
else
    print("small")
end

for i = 1, 5 do
    print(i)
end

local i = 0
while i < 3 do
    i = i + 1
end

local t = {}
t.foo = 1
t["bar"] = 2

local arr = {10, 20, 30}
for k, v in arr do
    print(k, v)
end
```

Supported: `local`, `function`, `if/elseif/else`, `while`, numeric `for i = a,b,step`, `for k,v in table`, `return`, `break`, tables, closures, string concatenation with `..`, `and`/`or`/`not`, comments with `--`.

Built-in libraries: `math` (`random`, `floor`, `ceil`, `abs`, `sqrt`, `sin`, `cos`, `max`, `min`, `pi`), `tostring`, `tonumber`, `type`, `len`.

## 2D / Audio API

```lua
window.open(width, height, title)
window.shouldClose()          -- true when user closes the window
window.close()
window.setTitle(title)

draw.begin()
draw.finish()
draw.clear(r, g, b)
draw.rect(x, y, w, h, r, g, b)
draw.circle(x, y, radius, r, g, b)
draw.line(x1, y1, x2, y2, r, g, b)
draw.text(str, x, y, size, r, g, b)
draw.fps(x, y)

image.load(path)               -- returns handle
image.draw(handle, x, y, scale)
image.width(handle)
image.height(handle)

input.key(name)                 -- held down, e.g. "up","down","left","right","space","enter","escape", letters, digits
input.keyPressed(name)          -- true only on the frame it was pressed
input.mouseDown()
input.mouseX()
input.mouseY()

audio.loadSound(path)           -- returns handle, for short effects
audio.play(handle)
audio.loadMusic(path)           -- returns handle, for streamed music
audio.playMusic(handle)
audio.updateMusic(handle)       -- call every frame while music is playing

time.dt()                       -- seconds since last frame
time.now()                      -- seconds since window.open

draw.screenshot(path)           -- saves the current frame as a PNG
```

## 3D rendering

```lua
d3.camera(x, y, z, targetX, targetY, targetZ, fovy)  -- set up the camera
d3.setPos(x, y, z)               -- move the camera
d3.setTarget(x, y, z)            -- point the camera

d3.begin()                       -- open a 3D block (call between draw.begin/draw.finish)
d3.finish()                      -- close it, back to 2D drawing

d3.cube(x, y, z, w, h, d, r, g, b)
d3.cubeWires(x, y, z, w, h, d, r, g, b)
d3.sphere(x, y, z, radius, r, g, b)
d3.plane(x, y, z, w, d, r, g, b)
d3.line(x1, y1, z1, x2, y2, z2, r, g, b)
d3.grid(slices, spacing)
```

A typical frame looks like:

```lua
draw.begin()
draw.clear(140, 190, 230)
d3.begin()
d3.grid(20, 1.0)
d3.cube(0, 1, 0, 1, 1, 1, 200, 60, 60)
d3.finish()
draw.text("score: 0", 10, 10, 20, 255, 255, 255)  -- 2D HUD drawn after d3.finish()
draw.finish()
```

## Video

raylib has no built-in video decoder, so Light ships its own tiny format: **`.lvid`** — a sequence of RGB24 frames, each compressed with a simple pixel-run RLE (good for flat-shaded/cartoon-style footage, not for noisy real-world video). It's built for streaming playback, not general-purpose video compression.

```lua
video.load(path)          -- returns a handle
video.play(handle)
video.pause(handle)
video.stop(handle)        -- pauses and rewinds to frame 0
video.update(handle, dt)  -- advance playback, call once per frame
video.draw(handle, x, y, w, h)   -- w/h optional, defaults to native size
video.seek(handle, seconds)
video.isFinished(handle)  -- true once a non-looping video reaches the end
video.setLoop(handle, true/false)  -- default true
video.duration(handle)    -- length in seconds
```

```lua
local vid = video.load("assets/clip.lvid")
video.play(vid)

while not window.shouldClose() do
    video.update(vid, time.dt())
    draw.begin()
    draw.clear(0, 0, 0)
    video.draw(vid, 0, 0, 640, 360)
    draw.finish()
end
```

To make your own `.lvid` file, use the Python encoder in `tools/lvid_encode.py`:

```python
from lvid_encode import encode_lvid
from PIL import Image

frames = [Image.open(f"frame_{i:04d}.png") for i in range(total_frames)]
encode_lvid(frames, fps=12, out_path="clip.lvid")
```

Or convert straight from a video file (mp4, mov, webm...) with `tools/mp4_to_lvid.py`, which drives ffmpeg for you — extracts frames, resizes them, packs them into `.lvid`, and pulls out the audio track as a `.wav` you can play separately with `audio.loadMusic`:

```
python3 tools/mp4_to_lvid.py input.mp4 output.lvid --fps 12 --width 320
```

This needs `ffmpeg` on PATH and `pip install Pillow`. Keep `--fps` and `--width` low — `.lvid` is not a real video codec, just pixel-run RLE, so it only stays small on simple/flat-shaded footage; busy real-world video will produce large files.

Keep source footage simple (flat colors, few gradients) — the RLE compresses runs of identical pixels, so busy/noisy video gets large fast. `examples/video_demo` includes a procedurally generated clip as a working sample.


## Examples

- `examples/flappybird/main.light` — Flappy Bird clone: gravity, pipes, collision, score, and sound effects.
- `examples/doom/main.light` — a Doom-style raycasting engine: real first-person 3D-looking walls rendered from a 2D map, built entirely out of `draw.rect` columns, plus footstep audio. Arrow keys move/turn. (NOTE:Doom got removed in the latest update for private reasons.)
- `examples/3d_demo/main.light` — actual 3D rendering: an orbiting camera, a ground grid, a bouncing sphere, and spinning cubes, using the `d3` API. Arrow keys orbit/raise the camera.
- `examples/badapple/main.light` — an original silhouette-dance animation with a looping chiptune track, showing off shape rendering and streamed music (no copyrighted footage is used anywhere — the animation and music are generated from scratch). (NOTE:Bad apple got removed in the latest update for private reasons.)
- `examples/video_demo/main.light` — plays back a procedurally generated `.lvid` clip using the `video.*` API (load/play/pause/stop/seek), showing off Light's built-in video codec.

Run any of them the same way:

```
cd examples/flappybird
light main.light
```

## Building from source

Source lives in `src/light.c`. It's a single C file with no dependencies besides libm for the interpreter itself; add `-DLIGHT_GRAPHICS` and link raylib to enable `window`/`draw`/`image`/`input`/`audio`/`time`.

```
gcc -O2 -o light src/light.c -lm
```

or, for the graphics-enabled build against raylib:

```
gcc -O2 -DLIGHT_GRAPHICS -I<raylib include> -o light src/light.c -L<raylib lib> -lraylib -lopengl32 -lgdi32 -lwinmm
```
