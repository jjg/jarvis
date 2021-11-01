# Journal

## 10282021

I started playing with [Mycroft](https://mycroft.ai/) to build a voice assistant into a 1950's speakerphone.  I don't know that this warrants a full-blown repository, but I needed a place to keep notes.

Here's the parts list so far:

* Raspberry Pi 3 Model A
* Adafruit i2s audio codec and 3w amplifier
* Some USB mic I had lying around
* Bell System speakerphone

I burned the [picroft image](https://mycroft-ai.gitbook.io/docs/using-mycroft-ai/get-mycroft/picroft) to a 16GB sd card to get started.

There was a lot of confusion around how the i2s board should be wired-up to the pi, but here's what finally worked:

```
pi

pin 4 5v Power      ->  VIN 
pin 6 Ground        ->  GND
pin 12 (GPIO 18)    ->  BCLK
pin 35 (GPIO 19)    ->  LRC
pin 40 (GPIO 21)    ->  DIN
```

The [Adafruit guide](https://learn.adafruit.com/adafruit-max98357-i2s-class-d-mono-amp/raspberry-pi-wiring) threw me off because it refers to the pins by number but what it *means* is to refer to the GPIO numbers, not the pin numbers themselves.

Since I had the wiring wrong, I did a lot of screwing-around with the configuration that may not have been needed.  The guide provides a script you can run that should get things setup to a point where you can hear some output from speaker-test`.

Now with that working it's time to get Mycroft talking to us.  There is a tool to help with this:

`mycroft-start audiotest`

This records some sound and plays it back.  Recording seems to have worked but when it tries to playback I get this:

```
Playing WAVE '/tmp/test.wav' : Signed 16 bit Little Endian, Rate 16000 Hz, Mono
aplay: set_params:1345: Channels count non available
An error occured while playing back audio (1)
```

This may have something to do with the i2s board being mono.

Did some searching and found [this](https://unix.stackexchange.com/questions/615584/alsa-aplay-mono-file-but-returns-channel-count-non-available)

Let's try this and see what happens...

Added these two lines to the bottom of .asoundrc:

```
defaults.pcm.card 0
defaults.ctl.card 0
```

I honestly have no idea what this does, but it can't hurt to try right?

Didn't work, but I'm going to reboot and try again.  Seems like there is a lot of rebooting involved with this audio stuff...

Still no dice.  This might require more serious tweaking, but now that I have the hardware connected right I'm going to re-run the initial setup just to see if that can fix it.

`microft-setup-wizard`

When it comes time to select an audio output device, none of the choices make sense.  I tried the Google one, and after a lot of stuff happening it rebooted and made a nasty noise for a moment.  Ultimately it didn't work, so I'm going to try setting it up the way I did the first time (tell it to use the headphone jack audio) and then re-run the Adafruit scripts and see where that ends up.

This time the Adafruit script worked and played-back audio at the end as expected.  Now we'll reboot and see what Mycroft says.

Mycroft still says nothing, and kinda doesn't understand the audio from the mic.  Time to work through this tshooting page:  https://mycroft-ai.gitbook.io/docs/using-mycroft-ai/troubleshooting/audio-troubleshooting

After a lot of screwing-around, it turned out that there was uneeded arguments being passed to the playback command in /etc/mycroft/mycroft.conf.  Removing that and now the sound test works.

...now to see if Mycroft will say something

...awww yeeeeah.

So it's working.  There is, however some weird buzz/distortion whenever audio is about to play.  There is some mention of this in the Adafruit docs, so let's see if the hack they propose will fix it...

So I got lazy and just re-ran the Adafruit config script.  After a reboot, I wasn't seeing any signal from the mic in the CLI...

fuuuuuuu....all the audio is broken again!

reboot again, mic is back!  It takes awhile after a reboot for the device to be ready to process commands.

OK, two more reboots, and disabled the anti-click hack.  Back to working but with the annoying click.  I'm just going to live with this for now and hope to find a solution (or if not, replace the i2s board).

At this point I think it's working well enough to try and cram it into the case!  I'm very curious about how well the mic will work inside there.

There's a number of other things I'd like to do, in particular I'd like to add an LED to indicate that it's "thinking" and shine this through the existing translucent volume knob (I'd also like to connect the volume knob an it's switch).  But for today let's not break anything else.


Looks like someone has already figured out the LED part I want: https://community.mycroft.ai/t/testing-and-feeback-wake-word-led-gpio/9839


## 11012021

Spent some time cleaning-up the insides and documenting the wiring.

