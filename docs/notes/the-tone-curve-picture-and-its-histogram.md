# The tone curve's picture and its pre-curve histogram

**The picture.** `draw_curve` in `panel/curve.rs` painted a 256×256 buffer with
nearest-pixel steps along the curve and the diagonal, and square point markers.
Shown across the panel, the stroke stepped and the markers sat large on the
line.

The buffer is now 768×768 (`SCALE = 3`):
- The curve and the reference diagonal are anti-aliased by perpendicular
  distance to each segment, so steep and flat runs keep the same stroke width.
- The 256 histogram bins are linearly interpolated across the 768 columns with
  fractional top-edge blending.
- Point markers are anti-aliased circles at 65% of the former squares' size
  (`3.9 * s` outer radius when selected, ringed in the accent; `2.6 * s` when
  unselected, with a dark rim).

**The backdrop histogram.** `bins_read` fed the curve editor the same post-curve
bins that the Scope panel and the clipping lamps read from `Renderer::analyze`.
Moving a curve point or a parametric slider changed `EditKey`, ran the analysis
pass with the curve applied, and shifted the histogram behind the curve that
had just moved it.

The curve's horizontal axis is the tone entering the curve, as `finish::pick`
already reads it for the curve dropper. `Renderer::analyze_curve` in
`greycard-viewport` bins a 768-bin RGB histogram from a `View` whose active
curve (global when `target` is `None`, or the chosen mask's when `target` is
`Some(k)`) is replaced with `Curves::default().bake_with(&grading)`. Keyed on
that pre-curve view and `weigh_by`, moving curve points or parametric splits
dispatches no read back for the backdrop and leaves the histogram behind the
curve stationary, while the top Scope panel and the clipping lamps continue to
follow `Renderer::analyze` on the full output.
