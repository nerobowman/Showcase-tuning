# Showcase Tuning

Generate standalone code that exercises the project's **actual rendering code** to produce PNG image files, visually inspect those images, and iteratively fix rendering issues until the output is correct.

The developer specifies **what to focus on** when activating this skill (e.g., "showcase tune the particle system", "showcase the map renderer at night"). The skill provides the process; the developer provides the target.

## When to Use

Activate when the user asks to:
- "showcase tune" or "showcase" a visual component
- Visually verify rendering output
- Generate test images from rendering code
- Tune or debug visual appearance of rendered output

## Inputs

When the user activates this skill, extract or ask for:

1. **Focus area** (required): Which rendering code to exercise (e.g., "the 3D scene renderer", "the tile map", "character sprites", "the particle system")
2. **Conditions** (optional): Specific states, parameters, or variants to render (e.g., "at night", "with fog enabled", "low quality mode", "zoomed in")
3. **Known issues** (optional): What the user suspects is wrong (e.g., "colors look washed out", "sprites clip through walls")

If the user's request is vague (e.g., just "showcase tune"), ask them what rendering component to focus on before proceeding.

## Process

### Phase 1: Understand the Rendering Code

Before writing anything, study the target rendering code:

1. **Identify the renderer**: Find the classes/functions responsible for the focus area. Read them thoroughly.
2. **Trace the output path**: Understand how the rendering code produces visual output (Bitmap, Canvas, BufferedImage, pixel arrays, SVG, HTML Canvas, etc.)
3. **Identify inputs**: What data/state does the renderer need? (models, configuration, assets, seeds, etc.)
4. **Identify dependencies**: What does the renderer import? Are there platform-specific APIs (Android, browser, native) that constrain how a standalone harness can be written?
5. **Check for existing test infrastructure**: Look for existing test directories, test utilities, image comparison tools, or screenshot tests already in the project.

### Phase 2: Write the Showcase Harness

Write a **standalone program or test** that:

1. **Constructs the minimum required input data** for the renderer (mock models, test fixtures, generated data with fixed seeds for reproducibility)
2. **Calls the actual rendering code** — never reimplement or approximate the rendering logic
3. **Captures the output as a PNG** saved to a known path on the local filesystem
4. **Is runnable** with a single command (test runner, script, etc.)

#### Harness Guidelines

- **Place the harness** in the project's existing test directory structure. Follow the project's conventions for test file location and naming.
- **Use fixed seeds / deterministic inputs** so the same harness produces identical output across runs, making before/after comparison possible.
- **Minimize scaffolding**: Only set up what the renderer actually needs. Don't bootstrap the entire application if the renderer can be called directly.
- **Output resolution**: Use a resolution large enough to see detail (at least 480x360) but not so large it's slow. 640x480 is a good default.
- **Output path**: Write PNGs to `Showcase/` directory under the project rool. Use descriptive filenames: `showcase/<component>_<variant>.png`. Add the directory to the gitignore file.
- **Multiple variants**: If the user specified conditions, or if the rendering code has distinct visual modes/states, generate a separate PNG for each.

#### Platform-Specific Guidance

**JVM / Android projects:**
- For Android renderers that use `android.graphics.Bitmap`: Write an Android instrumented test (`androidTest/`) or use Robolectric in unit tests (`test/`).
- For pure JVM renderers: Write a JUnit test or a `main()` function. Use `java.awt.image.BufferedImage` + `ImageIO.write()` for output.
- For Compose `DrawScope` renderers: Bridge to `android.graphics.Canvas` via `CanvasDrawScope` or render the background bitmap directly.

**Web / TypeScript / JavaScript projects:**
- Use Node.js with `canvas` (node-canvas) or `sharp` to rasterize output.
- For browser-only renderers (WebGL, Canvas2D): Write a headless browser script (Puppeteer/Playwright) that loads a minimal HTML page, triggers rendering, and screenshots the canvas.
- For SVG output: Save the SVG directly and also rasterize it to PNG for visual review.

**Python projects:**
- Use `Pillow` (PIL) to capture or compose the output into a PNG.
- For matplotlib/plotly renderers: Use `savefig()` / `write_image()`.
- For pygame/pyglet: Render to an off-screen surface and save.

**Rust / C++ / native projects:**
- Write a test binary that calls the renderer and outputs PNG via `image` crate, `stb_image_write`, or equivalent.

**Other platforms:**
- Adapt the pattern: construct inputs, call the real renderer, capture output to PNG, save to filesystem.

### Phase 3: Run the Harness

1. **Build and run** the harness using the project's build system.
2. If the build or run fails, fix the harness (not the rendering code) and retry.
3. Confirm that the expected PNG files were written to the output path.

### Phase 4: Visual Review

Use the **Read tool** to open and inspect each generated PNG:

```
Read Showcase/<component>_<variant>.png
```

Evaluate the image by asking:

#### General Quality
- Does the image show meaningful rendered content (not blank, not all-black, not corrupted)?
- Does the rendering match what the code intends to produce?
- Are colors, proportions, and layout reasonable?

#### Focus-Area-Specific Quality
Based on what the user asked to showcase, evaluate the relevant aspects:

- **3D scenes**: Perspective correct? Depth sorting correct? Lighting gradual? No z-fighting?
- **2D sprites/tiles**: Aligned to grid? No sub-pixel bleeding? Correct layering order?
- **Text rendering**: Readable? Correct font/size? No clipping?
- **Color/palette**: Expected palette applied? No unexpected full-color leaks? Gradients smooth?
- **Effects (particles, fog, glow, shadows)**: Visible? Correct blending? Not obscuring content?
- **Animation frames**: If generating multiple frames, do they show progression?
- **Edge cases**: Empty state? Extreme zoom? Maximum content density?

#### Assessment

After reviewing, state one of:
- **PASS**: The rendering looks correct for the focus area. Describe what you see.
- **ISSUES FOUND**: List each specific visual problem. For each, identify the likely source in the rendering code.

### Phase 5: Fix and Iterate (if issues found)

For each visual issue:

1. **Trace to source**: Map the visual defect to the specific function/line in the rendering code. Read that code carefully.
2. **Fix the rendering code** — modify the main source, NOT the showcase harness. The harness is just a camera; the subject is the renderer.
3. **Re-run the harness** (Phase 3).
4. **Re-review the image** (Phase 4).
5. **Repeat** until the issue is resolved.

After all issues are resolved, do a final review of all generated images to confirm nothing regressed.

## Output

When the showcase tuning is complete, summarize:

1. **Images generated**: List all PNGs with their paths and what each shows
2. **Assessment**: PASS or list of issues found and fixed
3. **Changes made**: If rendering code was modified, list the files and describe each change
4. **Remaining concerns**: Any visual aspects that look off but are outside the current focus area

## Important Rules

- **Never reimplement rendering logic** in the harness. The whole point is to test the *actual* code.
- **Use deterministic inputs** (fixed seeds, hardcoded test data) so output is reproducible.
- **Fix the renderer, not the harness**, when visual output is wrong. The harness only exists to capture what the renderer produces.
- **Generate before reviewing**. Don't guess what the output looks like — always produce and inspect the actual image.
- **One focus area at a time**. If the user wants to verify multiple unrelated components, run the skill separately for each.
