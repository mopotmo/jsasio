# jsASIO

A Java Sound `MixerProvider` that makes ASIO audio drivers available to Java applications on Windows. Written in 2004 for the [GrooveManager MC-909 Sample Editor](https://github.com/mopotmo/groovemanager-mc909se), with a 1.1 bugfix release in 2007.

> **Status: archive.** These are the original sources. The native library is 32-bit Windows and was built with Visual Studio .NET 2003.

## Usage

Put `jsasio.jar` on the classpath and `jsasio_<version>.dll` on `java.library.path`. The jar registers `com.groovemanager.spi.asio.ASIOMixerProvider` through `META-INF/services`, so every installed ASIO driver shows up as a regular `javax.sound.sampled.Mixer`:

```java
for (Mixer.Info info : AudioSystem.getMixerInfo()) System.out.println(info);
```

ASIO's callback model (a double buffer the driver swaps) is mapped onto Java Sound's blocking `read`/`write` lines. `ASIODataLine` implements both `SourceDataLine` and `TargetDataLine`. ASIO channels are mono. A mono line plays or records only on the first channel of the device (the left channel on stereo cards), and a line with more channels uses consecutive channels. Features that have no Java Sound counterpart, such as `ASIOListener`, the clock sources and time code, are available by casting the mixer to `ASIOMixer`. Some ASIO drivers need the `floatconversion` plugin. To test without ASIO hardware, use ASIO4ALL. `ASIOMixerProvider.setFullCheck(false)` (1.1) skips the format and driver checks for realtime use.

Known issue (2004): with small buffers or under high system load, playback crackles. A likely cause is attaching the callback thread to the JVM on every buffer switch.

`com.groovemanager.spi.asio.TestASIO` is a command line test tool:

```
java -Djava.library.path=. -cp jsasio.jar com.groovemanager.spi.asio.TestASIO --help
  --list | --info | --test | --play   -mixer=N -line=N -buffersize=N -samplerate=X -source=<file>
```

## Layout

| Path | Contents |
|---|---|
| `src/com/groovemanager/spi/asio` | Java side. `NativeClass` subclasses wrap native ASIO structs through a `long pointer` field |
| `resources/META-INF/services` | Service registration |
| `native/jsasio` | JNI implementation (`jASIO*.cpp`/`.h`, one pair per Java class) and the VS .NET 2003 project |

## Building the native library

The ASIO SDK files the project needs are included in `native/jsasio/common`, `host` and `host/pc`. They come from the [Steinberg ASIO SDK](https://www.steinberg.net/developers/) 2.3.4: `common/` is under Steinberg's dual license, used here under GPLv3 (see `common/LICENSE.txt`), and `host/` is under the BSD-style license in each file's header. The original 2004/2007 builds used ASIO SDK 2.0, which can't be redistributed. Building against 2.3.4 hasn't been tested yet. The include path for the JDK headers is hard-coded in `jsasio.vcproj` (`C:\Programme\Java\jdk1.6.0\include`). Rename the resulting `jsasio.dll` to `jsasio_<LIB_VERSION>.dll` (see `ASIOMixerProvider.LIB_VERSION`).

## History

- **1.0** (2004): initial version, shipped with the MC-909 Sample Editor
- **1.1** (2007): bugfix release, including the "full check" switch contributed by Jurgen Schmitz

## License

GPLv3, see [LICENSE](LICENSE). ASIO is a trademark and software of Steinberg Media Technologies GmbH.
