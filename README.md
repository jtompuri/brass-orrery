# Brass orrery

A 3D mechanical brass orrery in one HTML file with Three.js. One hand-cranked drive shaft turns six gear pairs, each pair turns a nested tube, and each tube carries an arm with a planet. Nothing moves unless the drive shaft moves.

**Live page:** https://jtompuri.github.io/brass-orrery/

See also the [Tycho orrery](https://jtompuri.github.io/tycho-orrery/), an orrery of Tycho Brahe's geo-heliocentric system made the same way.

## Controls

- Drag to orbit the view, scroll or pinch to zoom.
- Play/pause, reverse and speed are in the control panel, and the crank turns the mechanism by hand.
- Click a planet to trace its gear train.

## How it was made

The page was written by Inkling 2 Mini, a local language model, from the written specification in [SPEC.md](SPEC.md), using the Pi coding agent. Small fixes were then made in Pi with the same model:

- left and right drag no longer turn the view the wrong way,
- the lighting is less bright,
- zooming in and out is faster.

Three.js r128 is loaded from cdnjs, so the page needs an internet connection.
