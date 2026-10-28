Encoded clips go here. tools/encode-all.sh writes them for you:

  bash tools/encode-all.sh ~/Downloads/APP_demo/3x

Each clip is three files:
  NAME.mp4         the video
  NAME.jpg         its first frame (shown until the video starts)
  NAME-thumb.jpg   the thumbnail in the strip

NAME must match a "file" entry in CLIPS near the bottom of index.html:
  two-mice  open-field  three-chamber  novel-object  y-maze
  tail-suspension  mac-head-direction  mac-plus-maze
