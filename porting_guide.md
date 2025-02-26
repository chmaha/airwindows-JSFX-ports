# How to Port Airwindows to JSFX

Use https://github.com/chmaha/airwindows-JSFX-ports/blob/main/porting_template.jsfx as a starter template (adapting name of plugin, author and license as appropriate) and be sure to complete the original airwindows plugin name and source code link (by replacing the ??? placeholders).

## name_of_plugin.cpp (e.g. Dubly3.cpp)

1. Add the correct number of sliders, default values, and labels (found in `getParameterName`).
2. **Double-check if a slider is not in the 0-1 range** (found in `getParameterDisplay`).
   - If so, determine the range, use it for the slider, and reverse it in the `@slider` section so the letter remains in the 0-1 range.
   
   Example:
   ```c
   (B*B*175.0)+25.0 // in getParameterDisplay
   ```
   Becomes:
   ```c
   B = sqrt((slider2-25)/175);
   ```
   Otherwise, simply use the format:
   ```c
   @slider
   
   A = slider1;
   B = slider2; // etc.
   ```
4. Add any other non-zero variables to the `@init` section.

## name_of_plugin.h (e.g. Dubly3.h)

1. Scroll down to the `private:` section.
2. Add any arrays using `freemem`, e.g.:
   ```cpp
   bezA[13];
   bezB[13];
   ```
   becomes:
   ```c
   freemem = 0;
   bezA = freemem; freemem += 13;
   bezB = freemem; freemem += 13; // etc.
   ```
4. Convert any enums to values starting at `0` and add them to `@init`. Use the `xxx_total` final number as the freemem addition (generally no need to set that particular variable unless it it referenced later in xxxProc.cpp).

## name_of_pluginProc.cpp (e.g. Dubly3Proc.cpp)

### `@block` Section
Paste the section `processDoubleReplacing` up until `while (--sampleFrames >= 0)`.

Ignore:
```cpp
 double* in1  =  inputs[0];
 double* in2  =  inputs[1];
 double* out1 = outputs[0];
 double* out2 = outputs[1];
```
Ignore:
```c
if (fabs(inputSampleL)<1.18e-23) inputSampleL = fpdL * 1.18e-17;
if (fabs(inputSampleR)<1.18e-23) inputSampleR = fpdR * 1.18e-17;
```

### `@sample` Section
Paste the section starting `while (--sampleFrames >= 0)`.

Replace:
```cpp
double inputSampleL = *in1;
double inputSampleR = *in2;
```
With:
```c
inputSampleL = spl0;
inputSampleR = spl1;
```

Replace:
```c
fpdL ^= fpdL << 13; fpdL ^= fpdL >> 17; fpdL ^= fpdL << 5;
fpdR ^= fpdR << 13; fpdR ^= fpdR >> 17; fpdR ^= fpdR << 5;
```
With:
```c
fpdL = rand(UINT32_MAX);
fpdR = rand(UINT32_MAX); // Set UINT32_MAX = 4294967295; in @init as necessary
```
But only if `fpdL` and `fpdR` are used to generate other values. If not, these lines can simply be deleted.

Replace:
```cpp
*out1 = inputSampleL;
*out2 = inputSampleR;

in1++;
in2++;
out1++;
out2++;
```
With:
```c
spl0 = inputSampleL;
spl1 = inputSampleR;
```

## Conversions (Find/Replace Where Possible)
- Remove all `double`, `int` references
- `getSampleRate()` → `srate`
- `M_PI` → `$pi`
- `M_PI_2` → `$pi/2`
- `fabs` → `abs`
- `fmax` → `max`
- `fmin` → `min`
- `++` → `+=1`
- `--` → `-=1`
- `e` → `*10^`
- `true` → `1`
- `false` → `0`
- `if...else` → ternary e.g.
```cpp
if (x == 3) y = y * c; else y = -c;
```
becomes:
```c
(x == 3) ? y = y*c : y = -c;
```
- `{}` → `();`
- `for` loops → `while` loops (e.g. `count = 1; while () (code here; count+=1;);`)
- `switch` statements → series of ternary equivalents
- and so on...

## Rare Cases

```cpp
VstInt32 inFramesToProcess = sampleFrames; // in processDoubleReplacing
```
```cpp
double buf = (double)sampleFrames/inFramesToProcess; // in while (--sampleFrames >= 0)
```
Becomes:
```c
@block
sample_index = 0;

@sample
sample_index += 1;
samples_left = samplesblock - sample_index;
buf = samples_left/samplesblock;
```

## Final Checks

- Use the **REAPER JS Development Environment** to check for coding errors, including verifying that all custom variables have `ref value >1`; otherwise, they can be safely removed.
- **JSFX does not distinguish between upper and lower case letters**, so rename variables as necessary.

