---
title: Very Simple Framer
---
## Writeup: Very Simple Framer (Misc)

This challenge involved extracting a hidden message from a 1-pixel border of an image. The provided script, `chal.py`, revealed a custom steganography method where a message was converted to binary and mapped to the perimeter pixels of the file.

---

### 1. Challenge Analysis

The script `chal.py` performs the following operations:

* **Binary Conversion**: It converts a plaintext message into an 8-bit binary string (e.g., 'A' becomes `01000001`).
* **Border Mapping**: It generates coordinates for the outermost border of the image, starting from the top-left  and moving clockwise.
* **Pixel Coloring**:
* A `0` bit is represented by a **black** pixel `(0, 0, 0)`.
* A `1` bit is represented by a **white** pixel `(255, 255, 255)`.


* **Message Looping**: The message repeats until the entire 1-pixel border is filled.

### 2. Extraction Strategy

To retrieve the flag, we need to reverse the process by reading the border pixels in the same order they were written. Although `output.jpg` is a lossy format, the script uses high-contrast colors (black and white), which allows for bit recovery through thresholding.

**The script to extract the flag:**

1. Open `output.jpg` and get its dimensions.
2. Iterate through the border coordinates using the same logic as `generate_border_coordinates`.
3. Check the Red channel of each pixel: if it is closer to 255, the bit is `1`; if closer to 0, the bit is `0`.
4. Group the bits into 8-bit bytes and convert them back to ASCII.

### 3. Execution and Result

Running the extraction script against the provided image yields a repeating string of the hidden flag.

**Flag:**
`pascalCTF{Wh41t_wh0_4r3_7h0s3_9uy5???}`

---