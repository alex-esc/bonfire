# About the BONFIRE project

## About the sequencer

BONFIRE is a cross-platform drum machine designed to work best on a live and acoustic performance. Ideal to play around the campfire with a voice and a guitar. BONFIRE version alpha-1a was developed over a span of 6 months by alex-esc.

## About the author

Hello, I am alex-esc, the maker of the BONFIRE drum machine. I am a young(-sh) music production and audio engineer student from Mexico, a hobbyist music programmer and an overall music and technology fan.

I am more than happy to make a custom drum machine or electronic instrument for you or your band.

I love helping musicians realize their ideas trough technology. I offer mixing services, music production services and one-on-one production workshops, I can build you a custom instrument for a flat fee or offer music programming tutoring or a music programming workshop and or conference. If you're interested in any of these services please reach out via my email: alexesc at disroot dot org.


## The inner workings of BONFIRE

BONFIRE is build on pure data and MobMuPLat, a visual programming environment for music and smartphone music apps. The project's design goes as follows:

```
Metronome -> 32 step Matrix -> kit selector -> Drum  synth -> MobMuPlat GUI -> Phone speakers
```

For some features to work, the button presses from the Phone are feed back into the matrix or drum synth like so:

```
Metro ->  Matrix -> kit selector -> Drum  synth ->  GUI -> speakers
            A          A                A            V
            |          |                |            |
            |          |                |            |
            +----<-----+-------<--------+------<-----+

("A" represents an arrow pointing up)
```

The BMP knobs signal flow works in a very similar fashion:

```
Metro ->  Matrix -> kit selector -> Drum  synth ->  GUI (BPM knobs) -> speakers
  A                                                  V
  |                                                  |
  |                                                  |
  +---------<----<-----<-------<--------<------<-----+

```

The pattern selector also works by taking input from the MObMuPLat GUI and feeding it back to a pattern selector with hard coded patterns, internally the selector works like this:

```
up down buttons -> internal counter -> pattern table -> pattern applier -> 32 step matrix --+
  A                                                                                         |
  |                                                                                         |
  +---------<----<-----<-------<--------<------<-----+                                      |
                                                     |                                      |
                                                     |                                      V
                                                     A                                      |
Metro ->  Matrix -> kit selector -> Drum  synth ->  GUI (BPM knobs) -> speakers             |
              A                                                                             |
              |                                                                             |
              +----------<------------------<-----------------------------<-----------------+
																							
```

The pattern table consists of a sub patch with 32 binary outputs that are fed into the 32 step matrix, the matrix itself has 96 inputs, 32 per step per drum kit (Kick, snare and hi hat). The patterns are stored by a hardwired connection between the steps and the pattern output. Therefore certain presets action it's given position on the step matrix.

The patterns included internally are not the patterns themselves but a combination of generic building blocks. One sub pattern is the "erase everything on the 32 steps", this sets all the steps to zero. Another sub pattern is the "Kick on the up beat", other examples include "snare or 3" or "hi hats on odd" and "hats on even". These sub patterns are connected to first trigger the "erase everything on the 32 steps" sub pattern before triggering the desired kick and snare sub pattern.

```
pattern table input -> sub patterns -> full patterns -> pattern applier
      A                                    V
      |                                    |
      +----<Erase previous pattern<--------+
```

The pattern sector also outputs text that represents the name of the currently selected full pattern. This is taken as the input of the phone GUI:

```
up down buttons -> internal counter -> pattern table -> pattern applier -> 32 step matrix --+
                                            V                                               |
                                            |                                               |
                                            +--string---+                                   |
                                                        |                                   |
                                                        |                                   V
                                                        V                                   |
Metro ->  Matrix -> kit selector -> Drum  synth ->  GUI (Pattern display)                   |
              A                                                                             |
              |                                                                             |
              +----------<------------------<-----------------------------<-----------------+
																							
```

Finally the drum synth, the section that makes the sounds when they are triggered by the metronome, works by applying and amplitude envelope to an always running synthesizer and applying some modulation or noise. This general process applies to all the drum synth engines, the 3 currently implemented synths are basic, modern and complex.

More specifically each synths works as follows:

**SIMPLE KICK**

```
Drum synth kick input -> Sound oscillator -> Amp and pitch envelope -> Sound output

```

**SIMPLE SNARE**

```
Drum synth snare input -> Noise oscillator -> high + low pass filters -> Amp envelope -> Sound output

```

**SIMPLE, MODERN AND COMPLEX HI-HATS**

All the drum synths trigger the same hi-hat engine for now.

```
Drum synth hi-hat input -> Noise oscillator -> narrow band pass filter -> Amp envelope -> Sound output
```

**MODERN KICK**

```
Drum synth kick input + knob settings -> low freq phasor oscillator -> Amp envelope -> amplitude modulated and band-passed noise synth -> pitch modulation -> Sound output
```

The knob settings values are fed into the oscillators and envelopes to modify the timbre of the MODERN kick. This same architecture is used for all synths from here onward:

**MODERN SNARE**

The snare has a similar architecture to the kick  however the internal values are set differently to focus the noise of the output instead of the pitch modulated thump.

```
Drum synth snare input + knob settings -> hi freq sound oscillator -> Amp envelope -> amplitude modulated and band-passed noise synth -> pitch modulation -> Sound output
```
This time the sound oscillator is lowered in volume and creates a hi pitched sound to emulate the pitch of a tuned snare drum. The pitch is way lower in volume than the noise oscillator.

The amp and pitch modulation applied on the noise part of the snare is not only higher in volume but it has a longer decay to emulate gated reverb.

**COMPLEX DRUM SYTHS**

The theory behind the complex drum synths are beyond the scope of this document. Please refer to the original myMembrene patch or the author of the synth engine: [Mike Moreno](https://mikemorenodsp.github.io/about/).

**BLEND MODE**

The blend mode simply takes the input from the 32 step matrix and uses it to trigger all engines at once, but feeds them into a volume control section:

```
                      +--> Simple engine----> Independent volume knob --+
                      |                                                 |
Drum is triggered ----+--> Modern engine----> Independent volume knob --+---> Master Volume--> Sound output
                      |                                                 |
                      +--> Complex engine---> Independent volume knob --+
```

The drum kit selector functions like the blend mode, but instead of sending audio signals to drum synths, it sends triggers to the engines that are gated off depending on the status of what kit is selected, this process occurs in parallel:

```

Drum kit selector -> Number between 1 and 4 -> close all gates, then open Nth gate



                        +--> Gate 1---> Simple engine---+
                        |                               |                    
Drums are triggered ----+--> Gate 2---> Modern engine---+--> Sound output
                        |                               |                 
                        +--> Gate 3---> Complex engine--+
                        |                               | 
                        +--> Gate 4---> Blend engine----+
```

Future additions to BONFIRE like a sampler of audio effects will be implemented in parallel of the current implementation.

For example, the sampler will simply be another "drum synth" parallel to all other synths. The audio effects will run from the GUI to a module just before the sound output.

**SAMPLER**

```

Drum kit selector -> Number between 1 and 5 -> close all gates, then open Nth gate



                        +--> Gate 1---> Simple engine---+
                        |                               |                    
Drums are triggered ----+--> Gate 2---> Modern engine---+--> Sound output
                        |                               |                 
                        +--> Gate 3---> Complex engine--+
                        |                               | 
                        +--> Gate 4---> Blend engine----+
                        |                               | 
                        +--> Gate 5---> Sampler---------+
```

**AUDIO EFFECTS**

```
Metronome -> 32 step Matrix -> kit selector -> Drum  synth -> Audio FX module -> MobMuPlat GUI -> Phone speakers
                                                                  A                    V
                                                                  |                    |
                                                                  +---------<----------+
```
Of course the parameters being set on the GUI are fed back int the audio module.


## Credits, thank you's and acknowledgments

I'd like to thank my friends, family and every person that provided me with care and company during the development of BONFIRE, since it took place as the 2020 pandemic lock-down started. These times were very strange and hard hitting, thank you for helping me getting through, I'm talking about..... well you know who you are, thanks buds :)

This project would also not been possible without the technical inspiration and direct works from several people, including:

* Miller Puckette, the author of Pure Data, the software on top of the BONFIRE engine runs on. Puckette not only writes great software, but is also responsible of great inspiration for me. Watching his lectures and talks provide not only with great technical know-how but with a philosophy of art and instrument design that is truly unique and inspiring.
* Daniel Iglesia, the developer of MobMuPlat, the app that handles importing pd patches into a mobile environment. Without MobMuPlat BONFIRE would not have been possible.
* Sam Aaron, the developer behind Sonic Pi, a live programming environment for music sequencing and synthesis. The initial prototype for what would later become BONFIRE started on sonic pi. Not only that, but his love for music, programming and pedagogy are outright infectious. I personally can't help reading Sam's posts on the sonic pi forum or listen to a sonic pi performance by him or one of his conferences without stooping the video mid way to rush to sonic pi and get programming. Sam is responsible for making music programming so much more fun, accessible and outright addictive. Thanks for everything you do for the community.
* Mike Moreno, for writing and publishing "pd-mkmr", a pure data library  that contains "myMembrane" - the engine behind the Complex kit sound. Above writing myMenbrane Mike is also source of great inspiration for me. His work with pd is so advanced and yet simple in such a way that just leaves me in awe. When I'm stuck on pure data I just browse Mike's pd patches looking for inspiration and solutions and his work never seems to disappoint.
* Andrew brown from YouTube, Brown's tutorials on pure data are one of the most practical and effective out there. I really enjoy his teaching style and his tutorials were crucially in the making of BONFIRE,e specially the simple drum synth, since it's taken directly from Brown's tutorials.
* Soxsa, from the Sonic pi forum. Finally I'd like to thank soxsa for seeing something of value on my prototype sequencer. Frankly I think I would not have perused this project in particular were not for the initial interest of soxsa. Thank you for expressing your excitement for my sonic pi drum machine since it sparked my hunger to develop my drum machine more and more. Thank you for caring!



## History and development of the project

I became interested in getting a drum machine after going for a road trip and camping with my friends. At the time, around early 2019, I started to learn about music and playing the piano since I just got an upright piano. Before getting into piano and music theory I made some original music just by sequencing and looping on Ableton Live and singing on top of that, but until 2019 I never really knew anything about theory or harmony. On that road trip some of my friends brought their acoustic guitars, and we had a blast playing and singing all night.

One of my friends jokinly asked *"why didn't you bring your upright?"* And I think he was right in a way. It would have been awesome to have another instrument other than guitars to get a more full sound. So I looked around for small and portable instruments that I could always have on my bag and just take it out if someone breaks into song, so I could play along and do a fill here and there. I also researched drum machines for a bit, because the idea of a more electro-acoustic cover with electronic percussion would bring a simple song around a campfire into something even more special and polished. 

I looked into keyboard-like instruments, that would be pocket or bag size. I ended up getting a melodica because it's small-ish and the way that it sounds is distinct enough from a guitar to "cut through" all the guitars. The main reason I settled for a melodica was the price. I would have loved to get a VOLCA beats or something like that but here in Mexico the prices for audio equipment can get super inflated, sometimes you'll be paying double the price and international shipping on top of that. And because I also love to sing playing melodca would keep me from singing because the melodica is a wind instrument with a keyboard. So getting a drum machine was always on the back of my mind because I could "play" drums and sing on top.


The prototype for BONFIRE started as a response to an online community challenge, where electronic musicians were challenged to make an acoustic cover song using a music programming environment called *sonic pi*. I am a big fan of the sonic pi software and it's community so I wanted to join in and make an acoustic cover with sonic pi. My plan was to write a program on sonic pi that sends MIDI data to a sampler I own. Make the program loop and voilà you have a makeshift drum machine! The acoustic cover would be performed by me and my brother, where I live code the drum loop and as soon as I run the program my brother joins in with his acoustic guitar playing smashing pumpkins' *1979* and I would sing and my brother would join in doing harmonies on the chorus. I planed to do the cover with my brother because we had been uploading other acoustic covers a few weeks before. We did Post Malone's *Circles* and the Cure's *Boys don't cry* with my brother on acoustic guitar and me on the piano. So adding an electronic element would really take it to the next level. Secretly I also wanted to do the 1979 cover to make my electro-acoustic song around the campfire come halfway true.

Sadly, one of my bother's guitar strings had broken a day before rehearsal so the acoustic cover of 1979 was left in the dust. However the program I had written in sonic pi was fully working. I even posted the code on the sonic pi forum and it made some discussions come up and one user even took my code and re-wrote it into a generalized 16 step drum machine. [Here's the link to my post about my sonic pi drum machine](https://in-thread.sonic-pi.net/t/sonic-pi-monthly-challenge-2/4565/15) and here's [the generalized re-working by soxsa](https://in-thread.sonic-pi.net/t/general-purpose-drumkit/4687), that would later be developed onto BONFIRE.

My sonic pi drum machine was written on late November 2020 and the code was made public on the 20th of November. A few weeks after that I took an online music programming class on pure data, a visual programming environment for electronic music and synthesis. I have been trying to get into pure data for years but I never really understood it enough to make anything of substance with it. After taking the course I started a few projects to try to solidify my grasp of pure data. I made a simple synthesizer and a guitar effects "pedal" emulator, I also made about a half a synthesizer based on karplus strong synthesis, and finally I started to work on a re-imaging of my sonic pi drum machine on pure data.

The other projects fell apart and remained somewhat inactive because this was the first time that I have worked on synthesis. The only project that I kept working on was the sequencer, mainly because I already knew where I wanted to go and how to achieve it because I had already made it on sonic pi, no creativity required, just execution!

It's important to note that by now the pandemic lock-down was making it seem like everything will permanently change forever from here on out. So I really needed to find a project to keep me company and to have some sort of project solving to occupy my mind outside of pandemic induced anxieties. I just needed to fill up my brain with something else other than panic and sorrow. So that's the reason I enrolled on a pure data course and many other online courses, just to kill time on a more productive manner.

On February 2021 I started working on *"seqer-proto"*, the pure data re-implementation of the sonic pi sequencer. The code was made public on the 19th of February, by then I was somewhat advanced and 5 versions deep. *Sequer-proto* was also heavily inspired by [Andrew Brown's pure data tutorials](https://www.youtube.com/watch?v=wYlOw8YXoBs), but I found his method of storing patterns on arrays to be clunky.

On my implementation the pattern would be stored on the GUI itself, a strip of toggle boxes would be scanned by a metronome and if the toggle was on it would trigger a sound. The sounds themselves were directly ported from Brown's tutorial.

By version 8 the basics of a drum machine were already laid out. You could click on a 32 step matrix to draw in a drum pattern, the pattern would play and loop until you hit the off button on your screen or press space bar, you could sue your keyboard or use on screen buttons to trigger the drum sounds manually, like you would play drums on a pad or MIDI keyboard, you could use faders to mute certain parts of the drum kit, you could set your own BPM and switch between a 16 step mode and a 32 step mode, even mid song, that way you can trigger "drum fills" on time by adding ghost notes on the 17th to 32 step rage and staying on the 16 step mode and switching suddenly to 32 step mode, then going back to the regular loop by switching it to 16 steps again.

The project so far had no name (other than *seqer-proto8.pd*) and only worked on a computer or laptop running pure data. But after some testing on the 4th of March's version 11 of the program I set up the program to fully work with MobMuPlat, an app developed by Daniel Iglesia that allows users to create custom GUIs for pure data patches and run them on a mobile device like an iPhone or Android phone.

The next day I pushed version 5, removed *proto* from the name, this time calling it *sequer ver2* because it had come along way from its first implementation and I felt it cloud officially drop the prototype suffix. Because of this I started to feel less and less exited about the project, since it looked like it was close to done at the time. So I abandoned the project for a few months.

I came back to the project on June 2021 with newly found inspiration. This time I had a sudden realization on how I could implement multiple drum synthesis engines. This would allow the user to choose between two drum sounds. This time the second drum engine was entirely of my creation. I mainly tried to make an 80's style drum sound with a deeper and punchier kick and a snare that resembles a gated reverb effect. The second drum engine was dubbed "Modern" because compared to the first drum sound it sounds more modern. The first drum sounds more like a CR78 or a 808 and the second sounds like a retro-modern-ish 80's kit.

The modern setting was tuned to work as my drum sound to cover Gorilaz's *On melancholy hill*. But the sound can be modified since I made the engine modular from the ground up. You can add more decay on the kick and snare, add noise, and mess with the pitch envelope. Controls to the Modern sound were added on the 31st of March.

On that same version I also implemented a more advanced drum synthesis engine made by [Mike Moreno](https://mikemorenodsp.github.io/about/). Turns out Mike graduated from a university campus I was once enrolled in! Would have loved to met him and maybe learn some music programming tips directly from the master himself. Mike's drum synth "myMembrane" is based on physical modeling synthesis and it has ready made parameters that I implemented into knobs on my sequencer.

By now we're on the revision called *seqencer3.3.pd* where all the features were neatly put together, including a brand new feature I've been working on for a while. So far when you wanted to play a pattern you'd first have to punch it in. For my terrible memory this meant opening up a browser and looking up this or that drum pattern because I tend to forget my patterns a lot.

So I made a way to store patterns by hard coding generic building blocks, for example a kick on the up beat, a snare on 3, hats on on-beats and hats on off-beats, and by suing different combinations I can build more complex patterns like a four on the floor or a hip hop pattern. This feature came in company of a selector sub patch, that scrolls between all the patterns and applies the one you like.

Thankfully I was able to travel during the pandemic to visit an old friend that moved out of the city about a year ago. He is just begging to learn guitar and knows a few songs and is also a very passionate singer. So when I stayed at his new home and he started to play guitar I pulled up my laptop to show him the sequencer. This was the first time I publically showed the sequencer to anyone. And because we had a blast playing I begun to build up confidence on the project. Another cool thing that happened on that trip was that my friends grandmother had just gifted his family with an old broken Yamaha keyboard. They were meant to take it and eventually fix it, but even though the keys were sticky and a few notes didn't worked we were able to completely jam out with my friends guitar and his half broken keyboard and my sequencer. My fiend's mother is a classically trained violinist so she was not only impressed by my playing but by the sequencer. So I was able to geek out about the music and the techniques behind the sequencer with another musician. Her feedback became critical in re-working the interface to make it more straightforward for live performances.

Almost ready for release I worked on the smartphone phone interface with MobMuPlat some more and settled on a name for the sequencer: *Bonfire*. The name was supposed to evoke an image of playing acoustic music besides a campfire, so I wanted a name related to camping or marshmallows or something like that. For a time the name was fireplace, but decided against it due to feedback from other friends. So BONFIRE it is!

Today I am getting ready for my first official release to the general public. I have been thinking about the architecture for about a year and have been working on the development for 6 months. So half a year later, when the lock-down is just beginning to slow down I am getting ready to launch BONFIRE version alpha-1a and hopefully plan out another road trip with my friends and have the project go full circle. And hopefully this time we can top last year's campfire show.

## Future plans and upcoming features

As I mentioned on the history section I already worked on a guitar effects pedal emulator, I plan to in the not so distant future implement it into BONFIRE. And add reverb, delay, chorus-flanger and distortion to BONFIRE. The effects already work, it's just a matter of implementing them.

As of version alpha-1a the 3 different drum sounds are static, meaning you can't change the quality of the sounds - the modern setting will always sound the same - but this will change in the future, since on the back end I already have knobs to modify the drum sounds of the modern, complex and blend kits. I just need to design a nice interface for it and implement it on the smartphone version.

I am also currently working on a sampler, on future versions you can play pre-recorded real drums and sequence them using BONFIRE. I plan to add live sampling and hook up the sampler to the effects pedal to make the weirdest beats out there.

As part of the sampler I also plan to add a looper feature. Meaning you can sing a melody into your smartphone microphone and it will loop forever, you can then hit play on the drum sequencer and have a melody and drum loop on the background. You'll be able to loop multiple layers and bring them in and out at will. This would work great for live performances.

Another feature that is already added but locked in the background is a euclidean sequencer. The aim of this feature is to provide a fast way to make up a drum pattern. Currently the setting is not included on the main software because I need to find a simple user interface for it.

Another thing I would love to one day do is to fully deploy BONFIRE into a standalone app. Right now I have no idea how to go about it, but I know it's at least possible thanks to libpd. If anyone reading this is interested in working with me to port BONFIRE to libpd you can contact me at alexesc at disroot dot org.