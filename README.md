# sound_lib
High-level cross-platform pythonic interface to the Bass audio library and its add-ons.

## Description
Originally found at http://hg.q-continuum.net/sound_lib, this work is my attempt to update the library, and (more importantly) add documentation.

# Usage
## Before we begin
To use the library, it is necessary to first instantiate either an input or an output, depending on what you are doing. Most often this will be an output, which can be used to play audio files from disk, or URLs for example.

```
from sound_lib.input import Input
from sound_lib.output import Output

input = Input()
output = Output()
```

It is worth noting that you do not need both of these objects.

## Playing files
We use the `sound_lib.stream` module for this purpose.

Once we have instantiated an instance of `sound_lib.stream.Stream`, there are several common attributes we can use:

* volume - The volume of the stream, from 0.0 to 1.0.
* pan - The panning of the stream, from -1.0 to 1.0
* frequency - The frequency of the stream, from 100.0 to 200000.0.
* position - The current position within the stream.

These are all Python properties, so for every attribute, there are matching `get_attribute` and `set_attribute` methods you can use.

There are also several useful methods:

* play - Play the stream.
* pause - Pause the stream.
* stop - Seems to do the same as pause.

### File streams
```
from sound_lib.stream import FileStream
stream = FileStream(file='sound.mp3')
```

### URL streams
```
from sound_lib.stream import URLStream
stream = URLStream('http://example.com/stream.ogg')
```

### Push stream
```
from sound_lib.stream import PushStream
stream = PushStream()
stream.push(data)
```

Note the `push` attribute. This can be used to push data from a microphone or other source to the stream.

## Recording
When recording we must always make an instance of `sound_lib.input.Input`:

```
from sound_lib.input import Input
input = Input()
```

We record with either `sound_lib.recording.Recording`, or `sound_lib.recording.WaveRecording`.

The difference is that with `sound_lib.recording.Recording, you have to provide your own callback as the proc keyword argumement to the constructor, and with `sound_lib.recording.WaveRecording`, you pass a filename as the filename keyword argument.

```
from sound_lib.recording import Recording
recording = Recording(proc = lambda *args: return True)
recording.play()
```

The above code will silently record, passing the gathered data to a lambda which does nothing except return True, which signifies that recording should continue.

A real world callback could look like this:

```
from ctypes import string_at
def cb(handle, buffer, length, user):
    """Turn the data into a string and print it."""
    buf = string_at(buffer, length)
    print(buf)
    return True
```

Alternatively, you could push it to an instance of `sound_lib.stream.PushStream` for live monitoring:

```
from ctypes import string_at
from sound_lib.output import Output
from sound_lib.input import Input
from sound_lib.recording import Recording
from sound_lib.stream import PushStream

output = Output()
input = Input()
stream = PushStream(chans=1)  # Mono stream.
stream.play()  # Play audio whenever it is received.

def cb(handle, buffer, length, user):
    """Push the received data to stream."""
    buf = string_at(buffer, length)
    stream.push(buf)
    return True

recording = Recording(channels=1, proc=cb)
recording.play()  # Start streaming.
```

An instance of `sound_lib.recording.WaveRecording is easier to use:

```
from sound_lib.input import Input
from sound_lib.recording import WaveRecording

input = Input()
recording = WaveRecording(filename='test.wav')
recording.play()
```

## Inputs and outputs
### Properties
* device - The numerical index of the device in use. Use `get_device_names() to get human-readable representations.
* volume - The volume of the device (from 0.0 to 100.0).
* frequency (output only) - The frequency of the output. Ranges from 100 to 44100. Use 0 to reset to 44100. Seems buggy.

### Methods
* free - Used to free up the sound device (necessary before creating a new input or output and resetting its device - you cannnot do that in real time).
* find_default_device - Used to find the numerical index of the default sound device.
* get_device_names - Used to get human-readable names for devices.
* stop (output only) - Used to stop all playing streams.
* start (output only) - Must be called after `stop` 
* pause (output only) - Seemingly the same as `stop`.
