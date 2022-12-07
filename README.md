
# qoirsk (better compression ratio compared to qoir): a Simple, Lossless Image File Format
(c) 2022 by surya kandau.
qoirsk (pronounced like "choir") is simple, lossless image file format that is
very fast to encode and decode while achieving compression ratios roughly
comparable to PNG.

It was inspired by the [QOI image file format](https://qoiformat.org/),
building on it in a number of ways:

- It uses a RIFF-like file structure, allowing future extensions.
- That structure means that qoirsk files can include metadata like color profiles
  (QOI is limited to two choices, "linear" or "sRGB") and EXIF information.
- It integrates LZ4 compression to [produce smaller
  files].
- It can represent premultiplied alpha (as well as PNG's non-premultiplied
  alpha), which can avoid a post-processing step if your game engine or GUI
  toolkit works with [premultiplied
  alpha](https://iquilezles.org/articles/premultipliedalpha/).
- It has an optional lossy mode.
- It partitions the image into independent tiles, allowing for multi-threaded
  codec implementations.
- As an implementation concern (not a file format concern), it can decode into
  a pre-existing pixel buffer (in a variety of pixel formats), not just always
  returning a newly allocated pixel buffer. This also supports multi-threaded
  decoding, where the entire pixel buffer is allocated first and N separate
  threads then decode into their own separate portion of that pixel buffer.


## Building

[`src/qoirsk.h`](src/qoirsk.h) is a single file C library, so there's no separate
configure or build steps. Just `#define qoirsk_IMPLEMENTATION` before you
`#include` it.


## Benchmarks

The numbers below are the top-level summary of the full benchmarks, normalized
so that qoirsk is 1.000 (the [full benchmarks](doc/full_benchmarks.txt) page has
raw, non-normalized numbers and links to the benchmark suite of images). For
example, **comparing PNG/libpng with qoirsk, libpng compresses a little smaller
(0.960 versus 1.000) but qoirsk encodes 30x faster (0.033 versus 1.000) and
decodes 4.9x faster (0.203 versus 1.000)**.

For example, PNG/fpng encodes faster (1.138x) than qoirsk but produces larger
(1.234x) files and decodes slower (0.536x) than qoirsk.

For example, JPEG-XL Lossless at its default encoder options produces smaller
files (0.613x) but encodes slower (0.003x) and decodes slower (0.017x) than
qoirsk. Inverting those last two numbers give qoirsk encoding 296x faster and
decoding 60x faster than JPEG-XL Lossless (using its "effort = 7" option).

The conclusion isn't that qoirsk is always better or worse than any other format.
It's all trade-offs. However, **qoirsk has the fastest decode speed listed** and
achieves reasonable (roughly comparable to PNG) compression ratios. JXL and
WebP, lossless or lossy, have better compression ratios but also slower encode
and decode speeds.

Even though PNG/stb and QOI are worse than qoirsk in all three columns
(compression ratio, encoding speed and decoding speed), they still have their
own advantages. PNG/stb, like any PNG implementation, has unrivalled
compatibility with other software (and the stb library is easy to integrate,
being a single file C library). QOI is the simplest (easiest to understand and
easiest to customize) format, weighing under 700 lines of C code (qoirsk is
around 3000 lines of C code plus 4000 lines of data tables).

To repeat, it's all trade-offs.


### Lossless Benchmarks

```
CmpRatio     = CompressedSize / DecompressedSize. Lower is better.
EncMPixels/s = Encode MegaPixels per second.     Higher is better.
DecMPixels/s = Decode MegaPixels per second.     Higher is better.

qoirsk_Lossless    1.000 CmpRatio     1.000 EncMPixels/s     1.000 DecMPixels/s  (1)
JXL_Lossless/f   0.860 CmpRatio     0.630 EncMPixels/s     0.120 DecMPixels/s  (2)
JXL_Lossless/l3  0.725 CmpRatio     0.032 EncMPixels/s     0.022 DecMPixels/s
JXL_Lossless/l7  0.613 CmpRatio     0.003 EncMPixels/s     0.017 DecMPixels/s
PNG/fpng         1.234 CmpRatio     1.138 EncMPixels/s     0.536 DecMPixels/s  (1)
PNG/fpnge        1.108 CmpRatio     1.851 EncMPixels/s       n/a DecMPixels/s  (1)
PNG/libpng       0.960 CmpRatio     0.033 EncMPixels/s     0.203 DecMPixels/s
PNG/stb          1.354 CmpRatio     0.045 EncMPixels/s     0.186 DecMPixels/s  (1)
PNG/wuffs        0.946 CmpRatio       n/a EncMPixels/s     0.509 DecMPixels/s  (3)
QOI              1.118 CmpRatio     0.870 EncMPixels/s     0.700 DecMPixels/s  (1)
WebP_Lossless    0.654 CmpRatio     0.015 EncMPixels/s     0.325 DecMPixels/s
```

(1) means that the codec implementation is available as a [single file C
library](https://github.com/nothings/stb/blob/master/docs/stb_howto.txt).

(2) means that the fjxl encoder is a single file C library but there is no fjxl
decoder. There's also the j40 single file JXL decoder, in a separate
repository, but I couldn't get it to work. Passing it something produced by the
cjxl reference encoder produced `Error: Decoding failed (rnge) during
j40_next_frame`.

(3) means that the Wuffs decoder is a single file C library but there is no
PNG/Wuffs encoder. The "compression ratio" numbers simply take the benchmark
suite PNG images "as is" without re-encoding.
   
               qoir                qoirsk
dice.png       337,811             333,374 
fish.png       630,380             614,451
clover.png     2,490,373           2,488,356
grass1.png     2,661,762           2,661,422  
 
qoirskv1:
                       input size    qoirsk     comp_time   decomp_time    
kodim01.png            694,365       694,330    0.063s      0.125s    
kodim02.png            605,719       607,354    0.046s      0.140s  
kodim03.png            482,155       496,117    0.047s      0.141s
kodim04.png            610,087       635,092    0.046s      0.140s
kodim05.png            758,418       759,242    0.047s      0.140s
kodim06.png            591,345       609,184    0.078s      0.125s          
kodim07.png            539,172       556,387    0.046s      0.140s 
kodim08.png            746,389       789,206    0.062s      0.141s
kodim09.png            561,120       552,440    0.047s      0.125s
kodim10.png            567,930       589,767    0.047s      0.141s
kodim11.png            601,411       611,644    0.046s      0.125s
kodim12.png            508,938       518,930    0.047s      0.125s
kodim13.png            803,553       789,293    0.046s      0.141s 
kodim14.png            669,142       677,872    0.047s      0.125s 
kodim15.png            574,072       603,778    0.047s      0.140s
kodim16.png            507,174       535,726    0.047s      0.125s
kodim17.png            578,568       593,599    0.046s      0.140s 
pk01_door01_add.png      3,220         5,979    0.015s      0.031s
pk01_door01_local.png   73,467       105,617    0.031s      0.047s 
pk01_door01_s.png       74,021       126,558    0.031s      0.031s 
pk01_door01a_d.png     219,672       253,462    0.031s      0.078s
pk01_door01b_d.png     235,926       263,265    0.047s      0.078s
pk01_floor01_s.png     183,438       300,241    0.047s      0.079s
pk01_floor01a_d.png    454,077       483,273    0.047s      0.094s
pk01_floor01b_d.png    511,058       576,245    0.062s      0.109s
pk01_floor02_local.png  67,734       133,719    0.032s      0.047s
pk01_floor02_s.png      46,811        84,057    0.016s      0.031s
clover.png           2,582,326     2,488,818    0.110s      0.344s
grass1.png           2,851,919     2,661,488    0.078s      0.343s
IMGP5482_seamless.png 2,212,110    2,018,303    0.094s      0.313s
mod_screen1j.png         4,829         7,165    0.047s      0.015s
dice.png               349,827       334,126    0.047s      0.109s
fish.png               447,419       616,577    0.078s      0.328s
                   
qoirskv2:
                       input size    qoirsk     comp_time   decomp_time    
dice.png                  349,827    333,852    0.062s      0.094s
fish.png                  447,419    615,449    0.094s      0.328s

qoirskv3:
                       input size    qoirsk     comp_time   decomp_time    
kodim01.png            694,365       694,236    0.047s      0.141s    
kodim02.png            605,719       607,329    0.078s      0.125s  
kodim03.png            482,155       496,028    0.078s      0.141s
kodim04.png            610,087       635,030    0.062s      0.141s
kodim05.png            758,418       759,186    0.094s      0.140s
kodim06.png            591,345       609,159    0.062s      0.125s          
kodim07.png            539,172       556,294    0.078s      0.141s 
kodim08.png            746,389       789,144    0.078s      0.140s
kodim09.png            561,120       552,384    0.062s      0.156s
kodim10.png            567,930       589,691    0.062s      0.172s
kodim11.png            601,411       611,538    0.063s      0.125s
kodim12.png            508,938       518,898    0.062s      0.125s
kodim13.png            803,553       789,273    0.094s      0.125s 
kodim14.png            669,142       677,835    0.062s      0.157s 
kodim15.png            574,072       603,725    0.078s      0.140s
kodim16.png            507,174       535,716    0.063s      0.141s
kodim17.png            578,568       593,556    0.047s      0.125s 
pk01_door01_add.png      3,220         5,923    0.032s      0.032s
pk01_door01_local.png   73,467       105,481    0.031s      0.031s 
pk01_door01_s.png       74,021       126,551    0.047s      0.047s 
pk01_door01a_d.png     219,672       253,461    0.047s      0.078s
pk01_door01b_d.png     235,926       263,264    0.046s      0.062s
pk01_floor01_s.png     183,438       300,224    0.062s      0.063s
pk01_floor01a_d.png    454,077       483,273    0.046s      0.094s
pk01_floor01b_d.png    511,058       576,245    0.047s      0.109s
pk01_floor02_local.png  67,734       133,556    0.046s      0.046s
pk01_floor02_s.png      46,811        84,049    0.046s      0.031s
clover.png           2,582,326     2,488,692    0.109s      0.344s
grass1.png           2,851,919     2,661,471    0.125s      0.375s
IMGP5482_seamless.png 2,212,110    2,017,453    0.125s      0.296s
mod_screen1j.png         4,829         7,138    0.031s      0.031s
dice.png               349,827       333,468    0.063s      0.094s
fish.png               447,419       614,998    0.125s      0.328s

   
### Lossy Benchmarks

qoirsk is first and foremost a lossless format (for 24-bit RGB or 32-bit RGBA
images) but it also has a trivial lossy mode (reducing each pixel from 8 to 6
bits per channel). Here are some comparisons to other lossy formats. Once
again, there are trade-offs.

```
qoirsk_Lossy       0.641 CmpRatio     0.903 EncMPixels/s     0.731 DecMPixels/s  (1)
JXL_Lossy/l3     0.440 CmpRatio     0.051 EncMPixels/s     0.091 DecMPixels/s
JXL_Lossy/l7     0.305 CmpRatio     0.013 EncMPixels/s     0.070 DecMPixels/s
WebP_Lossy       0.084 CmpRatio     0.065 EncMPixels/s     0.453 DecMPixels/s
```

Lossy encoders (other than qoirsk) use the respective libraries' default options.
Different size/speed/quality trade-offs may be achievable with other options.


### Multi-Threading

The benchmark numbers above are all single-threaded. Other codec
implementations can be sped up (in terms of wall clock time) by using multiple
threads. qoirsk is no different: multi-threaded decoding can be [over 3x
faster], depending on your input image size.


### Other Libraries

These libraries are only used by the benchmark program. The qoirsk codec
implementation has no dependencies (and brings its own LZ4 implementation).

JXL ([libjxl/libjxl](https://github.com/libjxl/libjxl.git)) is the official
JPEG-XL library. The /l suffix denotes the regular libjxl implementation and
the /f suffix denotes the `experimental/fast_lossless` encoder (also known as
fjxl) in that repository (but still using the regular libjxl decoder). The
final 3 or 7 denotes libjxl's "effort" encoding option, which defaults to 7.

PNG/fpng ([richgel999/fpng](https://github.com/richgel999/fpng.git)) is a fast
PNG encoder and decoder. The encoded output are valid PNG images but the fpng
decoder only accepts fpng-encoded PNG images. It is not a general PNG decoder.
It's also currently only SIMD-optimized for the x86 CPU family, not ARM.

PNG/fpnge ([veluca93/fpnge](https://github.com/veluca93/fpnge.git)) is a very
fast PNG encoder. The repository only contains an encoder, not a decoder. It's
also currently only SIMD-optimized for the x86 CPU family, not ARM.

PNG/libpng is the official libpng library as built on Debian Bullseye.

PNG/stb ([nothings/stb](https://github.com/nothings/stb.git)) is one of the
best known "single file C library" PNG implementations.

PNG/wuffs ([google/wuffs](https://github.com/google/wuffs.git)) is the PNG
decoder from the Wuffs repository. There is no encoder but the decoder is
discussed in ["The Fastest, Safest PNG Decoder in the
World"]. While its reported decode speed here is not as fast as PNG/fpng, PNG/wuffs is a
general PNG decoder and isn't limited to only those PNG images produced by the
PNG/fpng encoder.

QOI ([phoboslab/qoi](https://github.com/phoboslab/qoi.git)) is a recent
(released in 2021), simple image file format that is remarkably competitive.

qoirsk is this repository. Lossless is the default option. Lossy means using the
lossiness=2 encoding option, reducing each pixel from 8 to 6 bits per channel.

WebP is the official libwebp library as built on Debian Bullseye. The WebP file
format cannot represent an image dimension greater than 16383 pixels, such as
the 1313 x 20667 `screenshot_web/phoboslab.org.png` image from the benchmark
suite. For these large images, we use PNG/libpng instead.


### Excluded Libraries

AVIF wasn't measured. I gave up after hitting [Debian bug
976349](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=976349) and the
combination of (1) lossless is the primary focus of qoirsk but (2) the [avif
examples](https://github.com/AOMediaCodec/libavif/tree/b3e0f31/examples) or
avif.h file not obviously showing how to encode *losslessly* from the library
(not the command line tools). Maybe I'll try again later.

JPEG wasn't measured. Around 45% of the [benchmark suite
images](https://qoiformat.org/benchmark/) have an alpha channel, which JPEG
cannot represent.

PNG/libspng and PNG/lodepng weren't measured. They are presumably roughly
comparable, [within a factor of 2], to PNG/libpng and PNG/stb.


## License

Apache 2. See the [LICENSE](LICENSE) file for details.


---

Updated on November 2022.
