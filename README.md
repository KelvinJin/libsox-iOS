SoX is a very powerful tool to convert audio files into other formats and it can also apply
various effects to these files.

In order to use SoX on iOS, you'll have to import the libsox.a library and the sox.h header file.
You can either build it from the source code by yourself or you can use a prebuilt one.

### Build by yourself

+ First, download the source code using git.
```
git clone http://git.code.sf.net/p/sox/code sox-code
```
+ Download the build_ios script from this repo and put it into the sox-code folder you just downloaded.
+ Install automake, autoconf as well as libtool. (recommend using `homebrew`)
```
brew install automake
brew install autoconf
brew install libtool
```
+ Run it. (You might need to change the minimum iOS version in the script.)
```
# Run this to generate the configure bash script.
autoreconf -i

# And then run this to build the static library.
./build_ios
```
+ The libsox.a will be ready in the /build folder together with the sox.h header file.

### Download the prebuilt version.

You can download the version that supports iOS 9.0+ with bitcode support [here](https://drive.google.com/open?id=0B2ZE77-A2FLGX2pBTElJRjA3elE).

### How to use it

Drag both libsox.a and sox.h to your Xcode project. And add them to your target.

Import `sox.h` header. For Swift, you'll need to import `sox.h` in the bridge header.

<details>
<summary>Sample Code - Objective-C - Convert RAW PCM into WAV</summary>


```objective-c
BOOL rawPCMReencode(NSURL *srcURL, NSURL *dstURL) {
  @try {
      sox_format_t *in, *out; /* input and output files */
      sox_effects_chain_t * chain;
      sox_effect_t * e;
      char *args[10];

      // Since the input pcm file is actually raw data, we need to
      // provide the input encoding. Here we assume it's 32 bit floating number PCM
      sox_encodinginfo_t inputEncoding = {
          SOX_ENCODING_FLOAT, // Data type, here we say it's float
          32,                 // Number of bits per data point. Should be 32 in our case.
          0,
          sox_option_default,
          sox_option_default,
          sox_option_default,
          sox_false
      };

      // The output file should have the same encoding as the input.
      sox_encodinginfo_t outputEncoding = inputEncoding;

      sox_signalinfo_t inputSignal = {
          44100, // Sample rate, in our case 44100Hz.
          2,     // Channel number, in our case 2 channels.
          0,     // Precision, bit per sample.
          0,     // Sample number in the file.
          NULL
      };

      // Again, the output signal should be the same as input one.
      sox_signalinfo_t outputSignal = inputSignal;

      /* All libSoX applications must start by initialising the SoX library */
      assert(sox_init() == SOX_SUCCESS);

      /* Open the input file (with default parameters), the file type is raw in our case */
      assert(in = sox_open_read(srcURL.fileSystemRepresentation, &inputSignal, &inputEncoding, "raw"));

      /* Open the output file; Now we can specify that the file is in wav format */
      assert(out = sox_open_write(dstURL.fileSystemRepresentation, &outputSignal, &outputEncoding, "wav", NULL, NULL));

      /* Create an effects chain; some effects need to know about the input
       * or output file encoding so we provide that information here */
      chain = sox_create_effects_chain(&in->encoding, &out->encoding);

      /* The first effect in the effect chain must be something that can source
       * samples; in this case, we use the built-in handler that inputs
       * data from an audio file */
      e = sox_create_effect(sox_find_effect("input"));
      args[0] = (char *)in, assert(sox_effect_options(e, 1, args) == SOX_SUCCESS);
      /* This becomes the first `effect' in the chain */
      assert(sox_add_effect(chain, e, &in->signal, &in->signal) == SOX_SUCCESS);

      /* The last effect in the effect chain must be something that only consumes
       * samples; in this case, we use the built-in handler that outputs
       * data to an audio file */
      e = sox_create_effect(sox_find_effect("output"));
      args[0] = (char *)out, assert(sox_effect_options(e, 1, args) == SOX_SUCCESS);
      assert(sox_add_effect(chain, e, &out->signal, &out->signal) == SOX_SUCCESS);

      /* Flow samples through the effects processing chain until EOF is reached */
      sox_flow_effects(chain, NULL, NULL);

      /* All done; tidy up: */
      sox_delete_effects_chain(chain);
      sox_close(out);
      sox_close(in);
      sox_quit();
  } @catch (NSException *exception) {
      NSLog(@"Error when reencode PCM: %@", [exception description]);
      return NO;
  } @finally {
      NSLog(@"New PCM Generation done: %@", dstURL.path);
      return YES;
  }
}
```

</details>

### Any Questions

Please file an issue!
