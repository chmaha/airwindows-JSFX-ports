# How to Port Airwindows to JSFX

## name_of_plugin.cpp (e.g. Dubly3.cpp)

1. Add the correct number of sliders, default values, and labels (found in `getParameterName`).
2. **Double-check if a slider is not in the 0-1 range** (found in `getParameterDisplay`).
   - If so, determine the range, use it for the slider, and reverse it in the `@slider` section so the letter remains in the 0-1 range.
   
   Example:
   ```cpp
   (B*B*175.0)+25.0 // in getParameterDisplay
   ```
   Becomes:
   ```js
   B = sqrt((slider2-25)/175);
   ```
3. Add any other non-zero variables to the `@init` section.

## name_of_plugin.h (e.g. Dubly3.h)

1. Scroll down to the `private:` section.
2. Add any arrays using `@init freemem`, e.g.:
   ```js
   bez = freemem;
   freemem += 13;
   ```
3. Convert any enums to values starting at `0` and add them to `@init`.

## name_of_pluginProc.cpp (e.g. Dubly3Proc.cpp)

### `@block` Section
Paste the section `processDoubleReplacing` up until `while (--sampleFrames >= 0)`.

Ignore:
```cpp
if (fabs(inputSampleL)<1.18e-23) inputSampleL = fpdL * 1.18e-17;
if (fabs(inputSampleR)<1.18e-23) inputSampleR = fpdR * 1.18e-17;
```

### `@sample` Section
Paste the section `while (--sampleFrames >= 0)`.

Replace:
```cpp
fpdL ^= fpdL << 13; fpdL ^= fpdL >> 17; fpdL ^= fpdL << 5;
fpdR ^= fpdR << 13; fpdR ^= fpdR >> 17; fpdR ^= fpdR << 5;
```
With:
```js
fpdL = rand(UINT32_MAX);
fpdR = rand(UINT32_MAX); // Set UINT32_MAX = 4294967295; in @init as necessary
```
Only if `fpdL` and `fpdR` are used to generate other values. Often, these lines can simply be deleted just above:
```cpp
*out1 = inputSampleL;
*out2 = inputSampleR;
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
- `if...else` → `()? : ()`
- `{}` → `();`
- `for` loops → `while` loops (e.g. `count = 1; while () (code here; count+=1;);`)
- `switch` statements → series of `if/else`

## Rare Cases

```cpp
VstInt32 inFramesToProcess = sampleFrames; // in processDoubleReplacing
```
```cpp
double buf = (double)sampleFrames/inFramesToProcess; // in while (--sampleFrames >= 0)
```
Becomes:
```js
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

