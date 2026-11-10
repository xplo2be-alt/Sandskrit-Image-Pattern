# SandSkrit

An image and text to sand-table pattern GUI and Python library, based on SandSkrit by UnintelligibleMaker. Converts designs into continuous SVG and THR paths; released under the MIT License.


![SandSkritIntro](images/SandSkritIntro.jpg)
Note: My sand is a little piled up near the center of the table still in this photo.  

![SandSkritCharacters](images/SandSkritCharacters.jpg)



## Features

- **Continuous Path Generation**: Ensures that the resulting polyline is a single, unbroken line, suitable for sand drawing robots.
- **Parametric Shapes**: Built-in support for Archimedean spirals (inward and outward), lines, and circular loops.
- **Advanced Text Rendering**: Convert strings or multiple lines of text into continuous paths with automatic boundary checking.
- **Format Support**: Export to SVG for visualization or THR for direct use with sand table controllers.
- **Extensible Character Set**: Easily add or modify characters via the `characters_data.py` module.

## Installation

SandSkrit requires Python 3.6 or later. No additional dependencies are required for the core functionality.

1. Clone the repository:
   ```bash
   git clone https://github.com/UnintelligibleMaker/Sandskrit.git
   cd Sandskrit
   ```

## Usage

The following example demonstrates how to create a simple pattern with a spiral, some text, and export it to THR and SVG.

```python
from sandskrit import SandSkrit

# Initialize at the origin
sandskrit = SandSkrit(0, 0)

# Add a spiral starting from the center
sandskrit.add_spiral_out(radius=98, turns=5, ending_angle=90)

# Add some text
sandskrit.add_string("HELLO", scale=15)

# Add a circular loop at the current radius
sandskrit.add_outer_loop(turns=1, ending_angle=180)

# Save the output
sandskrit.save_svg("output.svg")
sandskrit.save_thr("output.thr")
```

### Multiple Lines of Text

SandSkrit can also center and fit multiple lines of text within the current boundary. This is useful for creating complex labels or messages.

```python
from sandskrit import SandSkrit

sandskrit = SandSkrit(0, 0)
# Create a boundary
sandskrit.add_spiral_out(radius=100, turns=0, ending_angle=90)

multi_lines = [
    {'text': 'THIS IS WHAT', 'scale': 10},
    {'text': 'SANDSKRIT', 'scale': 20},
    {'text': 'CAN DO.', 'scale': 15},
]
sandskrit.add_lines_of_text(multi_lines)
```

See `Example.py` for a more complex example of how to use the library.

### Browser GUI: image to pattern

The browser GUI runs locally and does not upload images or require Python, a web server, or additional packages.

1. Download or clone this repository.
2. Keep `image_pattern_gui.html` and `characters_data.js` together in the same folder.
3. Open `image_pattern_gui.html` in a current desktop browser.
4. Choose an image, enter optional text, adjust the settings, and download an SVG preview or a THR path.

Use the language selector to switch between English, Dutch, German, and French. The selection is saved in the browser and restored the next time the GUI is opened. Image processing takes place in the browser. The image is scaled to fit the selected sampling resolution, then pixels darker than the brightness threshold are selected. Small connected dark regions can be removed with the minimum-component-size setting.

Image modes:

- **Hatched fill** draws scan lines through the selected image regions. Fill spacing controls the distance between scan lines. The shading-level control maps pixel brightness to different lane densities: darker regions get more lines, and lighter regions get fewer.
- **Contours only** traces the boundaries of the selected dark regions.
- **Trace outline after fill** adds those contours after the hatch fill, useful for emphasizing logos and other shapes. This option is disabled in contours-only mode.

Sampling resolution controls image detail and processing time; a higher value can produce a larger, slower-to-generate path. The defaults are 768-pixel sampling, 1-pixel fill spacing, a brightness threshold of 245, 5 shading levels, and a 20-pixel minimum component size. These are starting points, not universal image settings: adjust them for each image. The fill encodes brightness as grayscale line density, not color; lower shading levels create fewer, more distinct tones, while higher levels can show smoother tone changes. Lower the brightness threshold to exclude lighter areas. Text uses the SandSkrit stroke glyphs rather than a system font. When image and text are both used, text is placed below the image.

The sand table follows one continuous path. Connecting moves between separate strokes are therefore part of the output and may be visible. The router favors paths near the edges of dark shapes, but cannot guarantee that every connection avoids both the shapes and open areas. High-contrast, relatively simple images generally give the clearest results.

The GUI exports a horizontal-orientation THR file with ten decimal places. Its theta direction is horizontally mirrored to compensate for the orientation reported on the original project setup. THR conventions and table orientation vary between controllers: check your controller's import requirements and test a small file before running a large pattern. The preview and SVG show the readable, unmirrored design.

#### Customizing the GUI

The interface, image-processing logic, translations, and export functions are in `image_pattern_gui.html`. Change the control defaults and ranges in the HTML inputs, adjust the processing and THR-export behavior in the inline JavaScript, or add another locale to the `messages` and `localeNames` objects and the language selector. Change the control's `data-i18n` key alongside its translation entries so all languages stay in sync. Keep the `characters_data.js` file beside the HTML file; it provides the browser with the glyph data from `characters_data.py`. If you change or add glyphs, update both files so the Python library and browser GUI stay consistent. The GUI has no build step.

To host the GUI as a static site, publish the HTML file and `characters_data.js` together (for example, from the repository root using GitHub Pages). Users can also download the two files and open the HTML locally.

## Credits and licensing

SandSkrit and the original Python library were created by **UnintelligibleMaker**. The project is distributed under the MIT License; see [LICENSE](LICENSE) for the terms and copyright notice. The browser GUI and later changes are part of this repository, and their contributors are recorded in GitHub's commit history. The existing project images are retained with the repository. Do not redistribute other people's images, fonts, or generated example files unless you have permission or their license allows it.

## API Overview

### `SandSkrit(x=0, y=0)`
Initializes the path at the specified starting point.

### `add_spiral_out(radius=98.0, turns=8.0, ending_angle=180.0)`
Adds an Archimedean spiral starting from the current point moving outwards.

### `add_spiral_in(turns=8.0)`
Adds an Archimedean spiral moving from the current radius towards the center.

### `add_outer_loop(turns=2.0, ending_angle=0.0)`
Adds circular loops at the current radius.

### `add_string(string, scale=5.0, ...)`
Adds a single line of text.

### `add_lines_of_text(lines_config, line_spacing=1.2)`
Adds multiple lines of text, centered vertically and horizontally within the current radius.

### `save_svg(filename, size=200)`
Saves the generated path as an SVG file.

### `save_thr(filename)`
Saves the generated path as a THR file (Theta-Rho format).

## Character Definitions

Characters are defined in `characters_data.py`. Each character entry in the `CHARACTERS` dictionary contains:

- `path`: A list of `(x, y)` tuples relative to the baseline `(0, 0)`.
- `width`: The total width of the character in the x direction.
- `start_offset`: An X-axis offset applied before drawing the character.

The coordinate system used for characters assumes:
- `y = 0`: Baseline.
- `y < 0`: Above baseline.
- `y > 0`: Below baseline (descenders).

## Output Formats

### SVG
A standard Scalable Vector Graphics file, useful for visualizing the path in any browser or vector editor.

### THR (Theta-Rho)
A format used by many sand tables. It represents points in polar coordinates:
- **Theta ($\theta$):** Cumulative angle in radians (can exceed $2\pi$ for multiple rotations).
- **Rho ($\rho$):** Normalized radius (ranging from 0 at the center to 1 at the edge).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Code of Conduct

We are committed to providing a friendly, safe, and welcoming environment for all. Please see our [Code of Conduct](CODE_OF_CONDUCT.md) for more information.

---
Created by UnintelligibleMaker
