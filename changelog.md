# Ver. alpha-1a

## changelog

+ New name for the project! BONIFIRE was the chosen name!
	* Used to be called simply *"sequencer"* or *"fireplace sequencer"* previously
+ New dark theme
+ Added tablet and phone variants. Phone layout is now vertical and divided unto multiple pages, tablet version remains one screen only.
+ Added 3 new drum sounds: Basic, Modern, Complex & Blend:
	* Basic emulates a simple roland 808 style drum machine with synthesized sounds
	* Modern adds some modulation to the basic preset, giving it an 80's feel. It has a snare with a longer tail and the kick sounds deeper.
	* Complex uses an advanced synthesis drum engine to emulate an acoustic drum set. The engine behind complex is made by [Mike Moreno DSP](https://mikemorenodsp.github.io/about/).
	* Blend simply combines all previous settings, it adds together basic with modern and complex to achieve an extra punchy sound.
+ Added a preset selector: This feature works great for quickly setting a generic rhythm and playing a song on another instrument. Use the arrows to browse drum pattern presets. The blue square shows the currently selected preset, press it to apply the pattern. Applying the pattern will draw the notes for you. Press the red X to erase the currently drawn pattern.
	- Presets included:
		* Four on the floor
		* 2 Beat
		* Iconic 8ths
		* Simple Rock pattern
		* Generic Punk pattern
		* When the Levee breaks break-beat
		* Impeach the president break-beat
		* The funky drummer
		* Bo-Diddley
		* Boom-Bap
		* Slow tempo shuffle
		* Generic Hip-Hop beat
		* Simple techno groove
		* Boots n' cats disco pattern
		* Son Clave
		* Rumba
		* Bosa Nova
		* Dembow-regaeton
+ Changed BPM knob to 3 separate knobs, one adds in 100-th's to the global BPM, the second knob adds in 10-th's, and the third knob adds to the BMP counter by unit.
	* The BMP selector used to be one knob from values 1 to 999, making it impractical to set reasonable values
+ Fixed bug where all knobs and toggles were set at zero at startup
	* Now all knobs have a default value that reflects normal behavior (BMP=120, steps=16)
+ Renamed all previous versions to this one "pre-release" for a more consistent naming convention