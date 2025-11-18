# Brother Speedio CNC Post Processor - Functions Reference

**Complete Function Documentation**
Post Processor Version: 44093 5afed19dc47d472ae5e33114f06fb8010607587f
File: `brother speedio.cps` (4329 lines)

---

## Table of Contents

### Lifecycle & Core Callbacks
- [activateMachine](#activatemachine)
- [defineMachine](#definemachine)
- [onAction](#onaction)
- [onClose](#onclose)
- [onComment](#oncomment)
- [onCommand](#oncommand)
- [onCycle](#oncycle)
- [onCycleEnd](#oncycleend)
- [onCyclePoint](#oncyclepoint)
- [onDwell](#ondwell)
- [onMovement](#onmovement)
- [onOpen](#onopen)
- [onParameter](#onparameter)
- [onPassThrough](#onpassthrough)
- [onRadiusCompensation](#onradiuscompensation)
- [onSection](#onsection)
- [onSectionEnd](#onsectionend)
- [onSpindleSpeed](#onspindlespeed)

### Motion Functions
- [onCircular](#oncircular)
- [onLinear](#onlinear)
- [onLinear5D](#onlinear5d)
- [onRapid](#onrapid)
- [onRapid5D](#onrapid5d)

### Multi-Axis & Rewind Functions
- [onMoveToSafeRetractPosition](#onmovetosaferetractposition)
- [onReturnFromSafeRetractPosition](#onreturnfromsaferetractposition)
- [onRewindMachineEntry](#onrewindmachineentry)
- [onRotateAxes](#onrotateaxes)

### Tool Management
- [getBodyLength](#getbodylength)
- [noSpindle](#nospindle)
- [prepareForToolCheck](#preparefortoolcheck)
- [startSpindle](#startspindle)
- [writeToolBlock](#writetoolblock)
- [writeToolCall](#writetoolcall)
- [writeToolMeasureBlock](#writetoolmeasureblock)

### WCS & Work Plane Functions
- [cancelWorkPlane](#cancelworkplane)
- [defineWorkPlane](#defineworkplane)
- [forceWorkPlane](#forceworkplane)
- [getWorkPlaneMachineABC](#getworkplanemachineabc)
- [positionABC](#positionabc)
- [setWorkPlane](#setworkplane)
- [writeWCS](#writewcs)

### Probing Functions
- [decodeProbeWCSBlum](#decodeprobeWCSBlum)
- [getProbingArguments](#getprobingarguments)
- [printProbeResults](#printproberesults)
- [protectedProbeMove](#protectedprobemove)
- [setProbeAngle](#setprobeangle)
- [setProbeAngleMethod](#setprobeanglemethod)
- [writeExtraBlumProbing](#writeextrablumprobing)
- [writeMeasureTools](#writemeasuretools)
- [writeProbeCycle](#writeprobecycle)
- [writeProbingToolpathInformation](#writeprobingtoolpathinformation)

### Drilling & Cycle Functions
- [approach](#approach)
- [getCommonCycle](#getcommoncycle)
- [writeDrillCycle](#writedrillcycle)

### Coolant Functions
- [getCoolantCodes](#getcoolantcodes)
- [setCoolant](#setcoolant)

### Smoothing Functions
- [initializeSmoothing](#initializesmoothing)
- [setSmoothing](#setsmoothing)

### Output & Formatting Functions
- [formatComment](#formatcomment)
- [writeBlock](#writeblock)
- [writeComment](#writecomment)
- [writeNotes](#writenotes)
- [writeProgramHeader](#writeprogramheader)
- [writeStartBlocks](#writestartblocks)
- [writeStock](#writestock)

### Positioning & Retract Functions
- [getRetractParameters](#getretractparameters)
- [writeInitialPositioning](#writeinitialPositioning)
- [writeRetract](#writeretract)

### Parametric Feed Functions
- [FeedContext](#feedcontext)
- [getFeed](#getfeed)
- [initializeParametricFeeds](#initializeparametricfeeds)

### Utility Functions
- [ensurePositiveAngle](#ensurepositiveangle)
- [forceABC](#forceabc)
- [forceAny](#forceany)
- [forceFeed](#forcefeed)
- [forceModals](#forcemodals)
- [forceXYZ](#forcexyz)
- [getForwardDirection](#getforwarddirection)
- [getOffsetCode](#getoffsetcode)
- [getSetting](#getsetting)
- [isTCPSupportedByOperation](#istcpsupportedbyoperation)
- [parseChoice](#parsechoice)
- [subprogramsAreSupported](#subprogramsaresupported)
- [validateCommonParameters](#validatecommonparameters)
- [validateToolData](#validatetooldata)

### Inspection Functions
- [defineLocalVariable](#definelocalvariable)
- [formatLocalVariable](#formatlocalvariable)
- [getPointNumber](#getpointnumber)
- [inspectionCreateResultsFileHeader](#inspectioncreateresultsfileheader)
- [inspectionWriteCADTransform](#inspectionwritecadtransform)
- [inspectionWriteWorkplaneTransform](#inspectionwriteworkplanetransform)

---

## Function Documentation

## activateMachine

**Location**: `brother speedio.cps:2584`

**Signature**:
```javascript
function activateMachine()
```

**Description**: Configures the machine settings based on the machine configuration. This function disables unsupported rotary axes, sets up tilted workplane usage, identifies TCP support, configures multi-axis feedrate settings, and optimizes machine angles for head configurations.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: None - Helper/utility function that configures internal state

**Called By**:
- onOpen (brother speedio.cps:630)

**Usage Example**:
```javascript
// From onOpen function
defineMachine(); // hardcoded machine configuration
activateMachine(); // enable the machine optimizations and settings
```

**Related Functions**:
- [defineMachine](#definemachine)
- [getBodyLength](#getbodylength)

---

## approach

**Location**: `brother speedio.cps:839`

**Signature**:
```javascript
function approach(value)
```

**Description**: Converts a probing approach direction string ("positive" or "negative") into a numeric sign value (1 or -1). Used throughout probing cycles to determine the direction of probe movement.

**Parameters**:
- `value` (string) - Approach direction, either "positive" or "negative"

**Returns**: (number) - Returns 1 for "positive", -1 for "negative"

**G-Code Impact**: None - Helper function for probing calculations

**Called By**:
- writeProbeCycle (brother speedio.cps:1139) - Used extensively in all probing operations

**Usage Example**:
```javascript
// From probing-x cycle
var edgeCoord = x + approach(cycle.approach1) * (cycle.probeClearance + tool.diameter / 2);
```

**Related Functions**:
- [writeProbeCycle](#writeprobecycle)
- [protectedProbeMove](#protectedprobemove)

---

## cancelWorkPlane

**Location**: `brother speedio.cps:3949`

**Signature**:
```javascript
function cancelWorkPlane(force)
```

**Description**: Cancels the active workplane rotation by outputting G69 (cancel frame). Also resets the currentWorkPlaneABC tracking variable to force re-output of workplane on next use.

**Parameters**:
- `force` (boolean) - If true, resets the gRotationModal before canceling

**Returns**: void - No return value

**G-Code Impact**: Outputs G69 to cancel active G68/G68.2 rotation

**Called By**:
- defineWorkPlane (brother speedio.cps:3068)
- onClose (brother speedio.cps:2380)
- onSection (brother speedio.cps:710)
- setWorkPlane (brother speedio.cps:3959)
- writeRetract (brother speedio.cps:4006)

**Usage Example**:
```javascript
// From onSection
if (insertToolCall) {
  currentWorkOffset = undefined;
  wcsIsRequired = newWorkOffset || insertToolCall;
  writeBlock(gRotationModal.format(69)); // cancel frame
}
```

**Related Functions**:
- [forceWorkPlane](#forceworkplane)
- [setWorkPlane](#setworkplane)

---

## decodeProbeWCSBlum

**Location**: `brother speedio.cps:1987`

**Signature**:
```javascript
function decodeProbeWCSBlum(probeOutputWorkOffset)
```

**Description**: Decodes Fusion 360's work offset numbering scheme to Blum probe's WCS numbering scheme. Handles standard offsets (G54-G59), extended offsets (G54.1 P1-P300), and rotary offsets (G54 G54.2 P1-P8, etc).

**Parameters**:
- `probeOutputWorkOffset` (number) - Fusion 360 work offset number (0-362)

**Returns**: (number) - Blum probe WCS code

**G-Code Impact**: None - Helper function for probe argument calculation

**Called By**:
- getProbingArguments (brother speedio.cps:1952)

**Usage Example**:
```javascript
// From getProbingArguments for Blum probing
return [
  conditional(outputWCSCode, "W" + probeWCSFormat.format(decodeProbeWCSBlum(probeOutputWorkOffset)))
];
```

**Related Functions**:
- [getProbingArguments](#getprobingarguments)
- [writeProbeCycle](#writeprobecycle)

---

## defineMachine

**Location**: `brother speedio.cps:544`

**Signature**:
```javascript
function defineMachine()
```

**Description**: Defines the machine configuration including rotary axes (A-axis or AC-trunnion), TCP settings, rewind/reconfigure logic, and multi-axis feedrate modes. This function overrides CAM-provided machine configurations based on post properties.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: None - Configuration function only

**Called By**:
- onOpen (brother speedio.cps:624)

**Usage Example**:
```javascript
// From onOpen
receivedMachineConfiguration = machineConfiguration.isReceived();
if (typeof defineMachine == "function") {
  defineMachine(); // hardcoded machine configuration
}
activateMachine(); // enable the machine optimizations and settings
```

**Related Functions**:
- [activateMachine](#activatemachine)
- [onOpen](#onopen)

---

## defineLocalVariable

**Location**: `brother speedio.cps:4161`

**Signature**:
```javascript
function defineLocalVariable(indx, value)
```

**Description**: Defines a local macro variable for inspection/probing operations. Writes a macro variable assignment statement using the specified index and value.

**Parameters**:
- `indx` (number) - Index (1-6) into the localVariable array
- `value` (string/number) - Value to assign to the variable

**Returns**: void - No return value

**G-Code Impact**: Outputs macro variable assignment (e.g., "#20 = 45.123")

**Called By**:
- inspectionWriteCADTransform (brother speedio.cps:4218)
- inspectionWriteWorkplaneTransform (brother speedio.cps:4241)
- writeProbingToolpathInformation (brother speedio.cps:4256)

**Usage Example**:
```javascript
// From inspectionWriteCADTransform
defineLocalVariable(1, abcFormat.format(cadEuler.x));
defineLocalVariable(2, abcFormat.format(cadEuler.y));
defineLocalVariable(3, abcFormat.format(cadEuler.z));
```

**Related Functions**:
- [formatLocalVariable](#formatlocalvariable)
- [inspectionWriteCADTransform](#inspectionwritecadtransform)

---

## defineWorkPlane

**Location**: `brother speedio.cps:3068`

**Signature**:
```javascript
function defineWorkPlane(_section, _setWorkPlane)
```

**Description**: Calculates and optionally sets the work plane (coordinate system rotation) for the current section. Handles 3-axis, 3+2, and 5-axis simultaneous operations with support for G68.2 tilted workplanes.

**Parameters**:
- `_section` (Section) - The section to calculate workplane for
- `_setWorkPlane` (boolean) - If true, outputs the workplane setting code

**Returns**: (Vector) - ABC angles for the workplane

**G-Code Impact**: May output G68.2 for tilted workplane or rotary positioning codes

**Called By**:
- onCommand (brother speedio.cps:2187) - For COMMAND_LOAD_TOOL
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection with tool call
if (insertToolCall) {
  if (settings.workPlaneMethod.useTiltedWorkplane) {
    defineWorkPlane(currentSection, true);
  }
}
```

**Related Functions**:
- [getWorkPlaneMachineABC](#getworkplanemachineabc)
- [setWorkPlane](#setworkplane)
- [positionABC](#positionabc)

---

## ensurePositiveAngle

**Location**: `brother speedio.cps:610`

**Signature**:
```javascript
function ensurePositiveAngle(angle)
```

**Description**: Converts negative angles to their positive equivalent by adding 360 degrees. Used for probing cycles that require positive angle values.

**Parameters**:
- `angle` (number) - Angle in degrees (can be negative)

**Returns**: (number) - Positive angle equivalent (0-360 degrees)

**G-Code Impact**: None - Helper function for angle calculations

**Called By**:
- writeProbeCycle (brother speedio.cps:1139) - Used in partial circle probing cycles

**Usage Example**:
```javascript
// From probing-xy-circular-partial-boss
"H" + xyzFormat.format(ensurePositiveAngle(cycle.partialCircleAngleA)),
"U" + xyzFormat.format(ensurePositiveAngle(cycle.partialCircleAngleB)),
"V" + xyzFormat.format(ensurePositiveAngle(cycle.partialCircleAngleC)),
```

**Related Functions**:
- [writeProbeCycle](#writeprobecycle)

---

## FeedContext

**Location**: `brother speedio.cps:3382`

**Signature**:
```javascript
function FeedContext(id, description, feed)
```

**Description**: Constructor function that creates a feed context object for parametric feed output. Stores an ID, description, and feed value for a specific movement type.

**Parameters**:
- `id` (number) - Unique identifier for this feed context
- `description` (string) - Human-readable description (e.g., "Cutting", "Ramping")
- `feed` (number) - Feed rate value

**Returns**: (object) - FeedContext object with id, description, and feed properties

**G-Code Impact**: None directly - Used to build parametric feed table

**Called By**:
- initializeParametricFeeds (brother speedio.cps:3242)

**Usage Example**:
```javascript
// From initializeParametricFeeds
if (movements & ((1 << MOVEMENT_CUTTING) | (1 << MOVEMENT_LINK_TRANSITION) | (1 << MOVEMENT_EXTENDED))) {
  var feedContext = new FeedContext(id, localize("Cutting"), getParameter("operation:tool_feedCutting"));
  activeFeeds.push(feedContext);
  activeMovements[MOVEMENT_CUTTING] = feedContext;
}
```

**Related Functions**:
- [initializeParametricFeeds](#initializeparametricfeeds)
- [getFeed](#getfeed)

---

## forceABC

**Location**: `brother speedio.cps:2786`

**Signature**:
```javascript
function forceABC()
```

**Description**: Forces output of A, B, and C rotary axis coordinates on the next motion command by resetting their output variables. Ensures rotary positions are re-output even if they haven't changed.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Causes A, B, C to be output on next motion even if unchanged

**Called By**:
- forceAny (brother speedio.cps:2793)
- positionABC (brother speedio.cps:3143)

**Usage Example**:
```javascript
// From positionABC
if (force) {
  forceABC();
}
var a = machineConfiguration.isMultiAxisConfiguration() ? aOutput.format(abc.x) : ...
```

**Related Functions**:
- [forceXYZ](#forcexyz)
- [forceAny](#forceany)
- [forceFeed](#forcefeed)

---

## forceAny

**Location**: `brother speedio.cps:2793`

**Signature**:
```javascript
function forceAny()
```

**Description**: Forces output of all axes (X, Y, Z, A, B, C) and feedrate on the next output by calling forceXYZ, forceABC, and forceFeed. Used when modal state needs to be completely reset.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Causes all axes and feed to be output on next motion

**Called By**:
- onSectionEnd (brother speedio.cps:2333)
- writeInitialPositioning (brother speedio.cps:4054)

**Usage Example**:
```javascript
// From onSectionEnd
if (isProbeOperation()) {
  // Turn off probe logic
  setProbeAngle();
}
forceAny();
```

**Related Functions**:
- [forceXYZ](#forcexyz)
- [forceABC](#forceabc)
- [forceFeed](#forcefeed)

---

## forceFeed

**Location**: `brother speedio.cps:2773`

**Signature**:
```javascript
function forceFeed()
```

**Description**: Forces feedrate output on the next motion command by resetting the currentFeedId and feedOutput variables. Used after rapid moves or when feedrate must be re-stated.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Causes feedrate to be output on next motion even if unchanged

**Called By**:
- forceAny (brother speedio.cps:2793)
- getFeed (brother speedio.cps:2659)
- onLinear5D (brother speedio.cps:3829)
- onRapid (brother speedio.cps:3751)
- onRapid5D (brother speedio.cps:3807)

**Usage Example**:
```javascript
// From onRapid
if (x || y || z) {
  writeBlock(gMotionModal.format(0), x, y, z);
  forceFeed();
}
```

**Related Functions**:
- [getFeed](#getfeed)
- [forceXYZ](#forcexyz)
- [forceABC](#forceabc)

---

## forceModals

**Location**: `brother speedio.cps:2928`

**Signature**:
```javascript
function forceModals()
```

**Description**: Resets modal G-code output variables to force re-output. If no arguments provided, resets all modal variables (motion, plane, absolute/incremental, feed mode). If arguments provided, resets only those specific modals.

**Parameters**:
- `...arguments` (OutputVariable) - Optional specific modal variables to reset

**Returns**: void - No return value

**G-Code Impact**: Causes modal G-codes to be re-output even if unchanged

**Called By**:
- writeInitialPositioning (brother speedio.cps:4054)
- writeRetract (brother speedio.cps:4006)
- writeToolCall (brother speedio.cps:3188)

**Usage Example**:
```javascript
// From writeRetract for G28
case "G28":
  forceModals(gMotionModal, gAbsIncModal);
  writeBlock(gFormat.format(28), gAbsIncModal.format(91), retract.words);
  writeBlock(gAbsIncModal.format(90));
  break;
```

**Related Functions**:
- [forceWorkPlane](#forceworkplane)
- [writeRetract](#writeretract)

---

## forceWorkPlane

**Location**: `brother speedio.cps:3945`

**Signature**:
```javascript
function forceWorkPlane()
```

**Description**: Forces re-output of the work plane by clearing the currentWorkPlaneABC tracking variable. Ensures G68.2 or rotary positioning is re-output on next use.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: None directly - Forces workplane output on next setWorkPlane call

**Called By**:
- cancelWorkPlane (brother speedio.cps:3949)
- defineWorkPlane (brother speedio.cps:3068)
- setProbeAngle (brother speedio.cps:4268)
- writeToolCall (brother speedio.cps:3188)
- writeWCS (brother speedio.cps:3172)

**Usage Example**:
```javascript
// From cancelWorkPlane
writeBlock(gRotationModal.format(69)); // cancel frame
forceWorkPlane();
```

**Related Functions**:
- [cancelWorkPlane](#cancelworkplane)
- [setWorkPlane](#setworkplane)

---

## forceXYZ

**Location**: `brother speedio.cps:2779`

**Signature**:
```javascript
function forceXYZ()
```

**Description**: Forces output of X, Y, and Z linear axis coordinates on the next motion command by resetting their output variables. Ensures linear positions are re-output even if they haven't changed.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Causes X, Y, Z to be output on next motion even if unchanged

**Called By**:
- forceAny (brother speedio.cps:2793)
- getCommonCycle (brother speedio.cps:831)
- onLinear5D (brother speedio.cps:3829)
- onRapid5D (brother speedio.cps:3807)
- onReturnFromSafeRetractPosition (brother speedio.cps:2162)
- onSection (brother speedio.cps:710)
- writeInitialPositioning (brother speedio.cps:4054)

**Usage Example**:
```javascript
// From getCommonCycle - force xyz on first drill hole
function getCommonCycle(x, y, z, r) {
  forceXYZ();
  return [xOutput.format(x), yOutput.format(y), zOutput.format(z), "R" + xyzFormat.format(r)];
}
```

**Related Functions**:
- [forceABC](#forceabc)
- [forceAny](#forceany)
- [forceFeed](#forcefeed)

---

## formatComment

**Location**: `brother speedio.cps:2837`

**Signature**:
```javascript
function formatComment(text)
```

**Description**: Formats comment text according to settings (case, permitted characters, maximum length). Filters out invalid characters, applies case transformation, and truncates to maximum line length.

**Parameters**:
- `text` (string) - Raw comment text

**Returns**: (string) - Formatted comment with prefix and suffix, or empty string if invalid

**G-Code Impact**: None - Helper function for comment formatting

**Called By**:
- initializeParametricFeeds (brother speedio.cps:3242)
- onOpen (brother speedio.cps:624)
- writeComment (brother speedio.cps:2869)
- writeMeasureTools (brother speedio.cps:2003)

**Usage Example**:
```javascript
// From onOpen
if (programName) {
  writeComment(programName + conditional(programComment, SP + formatComment(programComment)));
}
```

**Related Functions**:
- [writeComment](#writecomment)
- [writeNotes](#writenotes)

---

## formatLocalVariable

**Location**: `brother speedio.cps:4165`

**Signature**:
```javascript
function formatLocalVariable(prefix, indx, rnd)
```

**Description**: Formats a local macro variable reference with optional prefix and rounding format for DPRNT output statements.

**Parameters**:
- `prefix` (string) - Prefix string (e.g., "*X", "*A")
- `indx` (number) - Index (1-6) into the localVariable array
- `rnd` (string) - Rounding format code (e.g., "[53]" for mm, "[44]" for inches)

**Returns**: (string) - Formatted variable reference

**G-Code Impact**: None - Helper for DPRNT formatting

**Called By**:
- inspectionWriteCADTransform (brother speedio.cps:4218)
- inspectionWriteWorkplaneTransform (brother speedio.cps:4241)
- writeProbingToolpathInformation (brother speedio.cps:4256)

**Usage Example**:
```javascript
// From inspectionWriteCADTransform
writeln(
  "DPRNT[G331" +
  "*N" + getPointNumber() +
  formatLocalVariable("*A", 1, macroRoundingFormat) +
  formatLocalVariable("*B", 2, macroRoundingFormat) +
  // ... more variables
);
```

**Related Functions**:
- [defineLocalVariable](#definelocalvariable)
- [inspectionWriteCADTransform](#inspectionwritecadtransform)

---

## getBodyLength

**Location**: `brother speedio.cps:2649`

**Signature**:
```javascript
function getBodyLength(tool)
```

**Description**: Retrieves the total tool length (body + holder) for the specified tool, checking operation parameters for override values. Used for TCP calculations in head configurations.

**Parameters**:
- `tool` (Tool) - Tool object to get length for

**Returns**: (number) - Total tool length (bodyLength + holderLength)

**G-Code Impact**: None - Helper function for TCP calculations

**Called By**:
- activateMachine (brother speedio.cps:2584)

**Usage Example**:
```javascript
// From activateMachine for head configuration TCP
if (machineConfiguration.isHeadConfiguration() && compensateToolLength) {
  for (var i = 0; i < getNumberOfSections(); ++i) {
    var section = getSection(i);
    if (section.isMultiAxis()) {
      machineConfiguration.setToolLength(getBodyLength(section.getTool()));
    }
  }
}
```

**Related Functions**:
- [activateMachine](#activatemachine)

---

## getCommonCycle

**Location**: `brother speedio.cps:831`

**Signature**:
```javascript
function getCommonCycle(x, y, z, r)
```

**Description**: Generates common drill cycle parameters (X, Y, Z, R) and forces XYZ output for the first hole of any drilling cycle.

**Parameters**:
- `x` (number) - X position
- `y` (number) - Y position
- `z` (number) - Z depth
- `r` (number) - R plane (retract/clearance height)

**Returns**: (Array) - Array of formatted X, Y, Z, R strings

**G-Code Impact**: Outputs X, Y, Z, R parameters for drilling cycles

**Called By**:
- writeDrillCycle (brother speedio.cps:877) - Used in all drilling cycle types

**Usage Example**:
```javascript
// From drilling cycle
writeBlock(
  gRetractModal.format(98), gCycleModal.format(81),
  getCommonCycle(x, y, z, cycle.retract),
  cyclefeedOutput.format(F)
);
```

**Related Functions**:
- [writeDrillCycle](#writedrillcycle)
- [forceXYZ](#forcexyz)

---

## getCoolantCodes

**Location**: `brother speedio.cps:3411`

**Signature**:
```javascript
function getCoolantCodes(coolant, format)
```

**Description**: Generates M-codes for coolant control based on the requested coolant mode. Handles transitions between different coolant states (flood, through-tool, air, etc.) and returns formatted or unformatted codes.

**Parameters**:
- `coolant` (CoolantMode) - Requested coolant mode constant
- `format` (boolean) - If true/undefined, returns formatted M-codes; if false, returns raw values

**Returns**: (Array|number|undefined) - Coolant M-codes or undefined if no change needed

**G-Code Impact**: Returns M8, M9, M494, M495, M402, M403 codes for coolant control

**Called By**:
- onCommand (brother speedio.cps:2187) - For COMMAND_LOAD_TOOL
- setCoolant (brother speedio.cps:3394)

**Usage Example**:
```javascript
// From onCommand COMMAND_LOAD_TOOL
var coolantCodes = getCoolantCodes(tool.coolant);
if (Array.isArray(coolantCodes)) {
  coolantCodes = coolantCodes.join(getWordSeparator());
} else {
  coolantCodes = "";
}
```

**Related Functions**:
- [setCoolant](#setcoolant)

---

## getFeed

**Location**: `brother speedio.cps:2659`

**Signature**:
```javascript
function getFeed(f)
```

**Description**: Generates feedrate output, handling parametric feeds (F#500-style) if enabled, or standard feed values. Checks if feed has changed and returns appropriate output string or empty string if unchanged.

**Parameters**:
- `f` (number) - Feedrate value

**Returns**: (string) - Formatted feedrate string or empty string if unchanged

**G-Code Impact**: Outputs F values or F# parametric feed references

**Called By**:
- Multiple motion and probing functions throughout the file

**Usage Example**:
```javascript
// From onLinear
var f = getFeed(feed);
if (x || y || z) {
  writeBlock(gMotionModal.format(1), x, y, z, f);
}
```

**Related Functions**:
- [initializeParametricFeeds](#initializeparametricfeeds)
- [forceFeed](#forcefeed)

---

## getForwardDirection

**Location**: `brother speedio.cps:2973`

**Signature**:
```javascript
function getForwardDirection(_section)
```

**Description**: Calculates the forward tool direction vector for a section, accounting for multi-axis operations, workplane optimizations, and machine configuration.

**Parameters**:
- `_section` (Section) - Section to get direction for

**Returns**: (Vector) - Forward direction vector

**G-Code Impact**: None - Helper function for calculations

**Called By**:
- writeDrillCycle (brother speedio.cps:877)

**Usage Example**:
```javascript
// From writeDrillCycle - check if spindle axis matches section direction
if (!isSameDirection(machineConfiguration.getSpindleAxis(), getForwardDirection(currentSection))) {
  expandCyclePoint(x, y, z);
  return;
}
```

**Related Functions**:
- [getWorkPlaneMachineABC](#getworkplanemachineabc)

---

## getOffsetCode

**Location**: `brother speedio.cps:4139`

**Signature**:
```javascript
function getOffsetCode()
```

**Description**: Returns the appropriate G-code for tool length compensation based on TCP support. Returns G43 for standard compensation, G43.4 for multi-axis TCP, or G43.5 for 3-axis TCP.

**Parameters**: None

**Returns**: (number) - G-code value (43, 43.4, or 43.5)

**G-Code Impact**: Determines which tool length compensation mode to use

**Called By**:
- onCommand (brother speedio.cps:2187) - For COMMAND_LOAD_TOOL
- onReturnFromSafeRetractPosition (brother speedio.cps:2162)
- writeInitialPositioning (brother speedio.cps:4054)

**Usage Example**:
```javascript
// From onCommand COMMAND_LOAD_TOOL in G100 call
writeToolBlock(gFormat.format(100),
  "T" + toolFormat.format(tool.number),
  xOutput.format(start.x),
  yOutput.format(start.y),
  gFormat.format(getOffsetCode()),
  // ... more parameters
);
```

**Related Functions**:
- [isTCPSupportedByOperation](#istcpsupportedbyoperation)

---

## getPointNumber

**Location**: `brother speedio.cps:4210`

**Signature**:
```javascript
function getPointNumber()
```

**Description**: Returns the macro variable reference for the current inspection point number. Uses inspection framework variable if available, otherwise returns default Fanuc variable.

**Parameters**: None

**Returns**: (string) - Macro variable reference (e.g., "#122[60]")

**G-Code Impact**: None - Helper for inspection output

**Called By**:
- inspectionWriteCADTransform (brother speedio.cps:4218)
- inspectionWriteWorkplaneTransform (brother speedio.cps:4241)

**Usage Example**:
```javascript
// From inspectionWriteCADTransform
writeln(
  "DPRNT[G331" +
  "*N" + getPointNumber() +
  formatLocalVariable("*A", 1, macroRoundingFormat) +
  // ... more parameters
);
```

**Related Functions**:
- [inspectionWriteCADTransform](#inspectionwritecadtransform)
- [inspectionWriteWorkplaneTransform](#inspectionwriteworkplanetransform)

---

## getProbingArguments

**Location**: `brother speedio.cps:1952`

**Signature**:
```javascript
function getProbingArguments(cycle, updateWCS)
```

**Description**: Generates probe cycle arguments (tolerances, tool wear, WCS updates) based on cycle parameters and probing system type (Renishaw vs Blum). Handles different parameter sets for each system.

**Parameters**:
- `cycle` (Cycle) - Probe cycle object with parameters
- `updateWCS` (boolean) - If true and strategy is "probe", outputs WCS update code

**Returns**: (Array) - Array of formatted probe parameters

**G-Code Impact**: Generates B, F, H, M, T, V, W, S parameters for probe macros

**Called By**:
- writeProbeCycle (brother speedio.cps:1139) - Used in all probe cycle types

**Usage Example**:
```javascript
// From probing-x cycle
writeBlock(
  gFormat.format(65), "P" + 8811,
  "X" + xyzFormat.format(x + approach(cycle.approach1) * (cycle.probeClearance + tool.diameter / 2)),
  "Q" + xyzFormat.format(cycle.probeOvertravel),
  getProbingArguments(cycle, true)
);
```

**Related Functions**:
- [writeProbeCycle](#writeprobecycle)
- [decodeProbeWCSBlum](#decodeprobeWCSBlum)

---

## getRetractParameters

**Location**: `brother speedio.cps:2995`

**Signature**:
```javascript
function getRetractParameters()
```

**Description**: Generates retract parameters based on specified axes (X, Y, Z) and safe position method. Calculates home positions, validates retract sequence, and returns method and coordinates.

**Parameters**:
- `...arguments` (Axis) - Variable number of axis constants (X, Y, Z) to retract

**Returns**: (object) - Object with {method, retractAxes, words} or undefined

**G-Code Impact**: None directly - Returns data for writeRetract to use

**Called By**:
- onClose (brother speedio.cps:2380)
- writeRetract (brother speedio.cps:4006)

**Usage Example**:
```javascript
// From onClose - move table to final position
var retract_words;
if (getProperty("positionAtEnd") != "noMove") {
  var retract = getRetractParameters(X, Y);
  if (retract && retract.words.length > 0) {
    retract_words = retract.words;
  }
}
```

**Related Functions**:
- [writeRetract](#writeretract)
- [getSetting](#getsetting)

---

## getSetting

**Location**: `brother speedio.cps:2950`

**Signature**:
```javascript
function getSetting(setting, defaultValue)
```

**Description**: Retrieves a setting value from the settings object using dot notation path. Returns default value if setting doesn't exist. Used throughout to access configuration with fallbacks.

**Parameters**:
- `setting` (string) - Dot-notation path to setting (e.g., "smoothing.level")
- `defaultValue` (any) - Default value if setting not found

**Returns**: (any) - Setting value or default value

**G-Code Impact**: None - Configuration helper function

**Called By**:
- Many functions throughout the file to access settings

**Usage Example**:
```javascript
// From activateMachine
settings.workPlaneMethod.useTiltedWorkplane = getProperty("useTiltedWorkplane") != undefined ?
  getProperty("useTiltedWorkplane") : getSetting("workPlaneMethod.useTiltedWorkplane", false);
```

**Related Functions**:
- All functions that access settings object

---

## getWorkPlaneMachineABC

**Location**: `brother speedio.cps:3121`

**Signature**:
```javascript
function getWorkPlaneMachineABC(_section, rotate)
```

**Description**: Calculates ABC machine angles for a section's workplane using machine configuration preferences. Optionally applies rotation and optimizes 3D positions.

**Parameters**:
- `_section` (Section) - Section to calculate ABC for
- `rotate` (boolean) - If true, applies rotation and optimization

**Returns**: (Vector) - ABC angles for machine configuration

**G-Code Impact**: None directly - Returns angles for positioning

**Called By**:
- defineWorkPlane (brother speedio.cps:3068)
- getForwardDirection (brother speedio.cps:2973)
- setWorkPlane (brother speedio.cps:3959)

**Usage Example**:
```javascript
// From defineWorkPlane
if (settings.workPlaneMethod.eulerCalculationMethod == "machine" && machineConfiguration.isMultiAxisConfiguration()) {
  abc = machineConfiguration.getOrientation(getWorkPlaneMachineABC(_section, true)).getEuler2(settings.workPlaneMethod.eulerConvention);
} else {
  abc = getWorkPlaneMachineABC(_section, true);
}
```

**Related Functions**:
- [defineWorkPlane](#defineworkplane)
- [setWorkPlane](#setworkplane)
- [positionABC](#positionabc)

---

## initializeParametricFeeds

**Location**: `brother speedio.cps:3242`

**Signature**:
```javascript
function initializeParametricFeeds(insertToolCall)
```

**Description**: Sets up parametric feedrate table for different movement types (cutting, finishing, ramping, etc.). Creates FeedContext objects and outputs parameter assignments.

**Parameters**:
- `insertToolCall` (boolean) - If true or pattern changed, initializes feeds

**Returns**: void - No return value

**G-Code Impact**: Outputs #500=F... parameter assignments for each feed type

**Called By**:
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection
if (typeof initializeParametricFeeds == "function") {
  initializeParametricFeeds(insertToolCall);
}
```

**Related Functions**:
- [FeedContext](#feedcontext)
- [getFeed](#getfeed)

---

## initializeSmoothing

**Location**: `brother speedio.cps:3509`

**Signature**:
```javascript
function initializeSmoothing()
```

**Description**: Initializes smoothing/high-accuracy mode settings for the current operation. Determines appropriate smoothing level based on operation type, stock to leave, or tolerance. Handles automatic mode selection.

**Parameters**: None

**Returns**: void - No return value (updates smoothing object)

**G-Code Impact**: None directly - Configures smoothing state for setSmoothing

**Called By**:
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection
if (getProperty("showNotes")) {
  writeSectionNotes();
}
initializeSmoothing(); // initialize smoothing mode
```

**Related Functions**:
- [setSmoothing](#setsmoothing)

---

## inspectionCreateResultsFileHeader

**Location**: `brother speedio.cps:4169`

**Signature**:
```javascript
function inspectionCreateResultsFileHeader()
```

**Description**: Creates or reuses DPRNT results file for inspection/probing output. Opens file with POPEN, writes header with job/operation info, document ID, and model version.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs PCLOS, POPEN, DPRNT[RESULTSFILE*...], DPRNT[DOCUMENTID*...]

**Called By**:
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection for probing operations
if (isProbeOperation()) {
  validate(settings.probing.probeAngleMethod != "G54.4", "Cannot probe with G54.4 enabled");
  if (!settings.probing.probeOn) {
    // Turn on probe
  }
  inspectionCreateResultsFileHeader();
}
```

**Related Functions**:
- [printProbeResults](#printproberesults)
- [writeProbingToolpathInformation](#writeprobingtoolpathinformation)

---

## inspectionWriteCADTransform

**Location**: `brother speedio.cps:4218`

**Signature**:
```javascript
function inspectionWriteCADTransform()
```

**Description**: Writes the CAD model transformation (rotation and translation) to the inspection results file using G331 code. Outputs model origin and Euler angles.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs DPRNT[G331*N...*A...*B...*C...*X...*Y...*Z...]

**Called By**:
- writeProbeCycle (brother speedio.cps:1139)

**Usage Example**:
```javascript
// From writeProbeCycle when print results enabled
if (printProbeResults()) {
  writeProbingToolpathInformation(z - cycle.depth + tool.diameter / 2);
  inspectionWriteCADTransform();
  inspectionWriteWorkplaneTransform();
}
```

**Related Functions**:
- [inspectionWriteWorkplaneTransform](#inspectionwriteworkplanetransform)
- [defineLocalVariable](#definelocalvariable)
- [formatLocalVariable](#formatlocalvariable)

---

## inspectionWriteWorkplaneTransform

**Location**: `brother speedio.cps:4241`

**Signature**:
```javascript
function inspectionWriteWorkplaneTransform()
```

**Description**: Writes the workplane transformation (orientation) to the inspection results file using G330 code. Outputs current workplane Euler angles.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs DPRNT[G330*N...*A...*B...*C...*X0*Y0*Z0*I0*R0]

**Called By**:
- writeProbeCycle (brother speedio.cps:1139)

**Usage Example**:
```javascript
// From writeProbeCycle when print results enabled
if (printProbeResults()) {
  writeProbingToolpathInformation(z - cycle.depth + tool.diameter / 2);
  inspectionWriteCADTransform();
  inspectionWriteWorkplaneTransform();
}
```

**Related Functions**:
- [inspectionWriteCADTransform](#inspectionwritecadtransform)
- [defineLocalVariable](#definelocalvariable)

---

## isTCPSupportedByOperation

**Location**: `brother speedio.cps:3107`

**Signature**:
```javascript
function isTCPSupportedByOperation(_section)
```

**Description**: Determines if Tool Center Point (TCP) mode is supported and should be used for the current operation based on section type and machine configuration.

**Parameters**:
- `_section` (Section) - Section to check

**Returns**: (boolean) - True if TCP should be used

**G-Code Impact**: None - Affects getOffsetCode selection (G43 vs G43.4/43.5)

**Called By**:
- defineWorkPlane (brother speedio.cps:3068)
- setWorkPlane (brother speedio.cps:3959)
- writeInitialPositioning (brother speedio.cps:4054)

**Usage Example**:
```javascript
// From defineWorkPlane
tcp.isSupportedByOperation = isTCPSupportedByOperation(_section);
return abc;
```

**Related Functions**:
- [getOffsetCode](#getoffsetcode)
- [defineWorkPlane](#defineworkplane)

---

## noSpindle

**Location**: `brother speedio.cps:618`

**Signature**:
```javascript
function noSpindle()
```

**Description**: Determines if spindle codes (D, S, M3/M4) should be suppressed during tool change. Returns true for tapping cycles and probe tools.

**Parameters**: None

**Returns**: (boolean) - True if spindle codes should not be output

**G-Code Impact**: Suppresses D, S, M3/M4 in G100 tool change for taps and probes

**Called By**:
- onCommand (brother speedio.cps:2187) - For COMMAND_LOAD_TOOL
- startSpindle (brother speedio.cps:3216)

**Usage Example**:
```javascript
// From onCommand COMMAND_LOAD_TOOL
var rotateSpindle = !noSpindle();
// ... later in G100 block
conditional(rotateSpindle, diameterOffsetFormat.format(tool.diameterOffset)),
conditional(rotateSpindle, sOutput.format(spindleSpeed)),
conditional(rotateSpindle, mFormat.format(tool.clockwise ? 3 : 4)),
```

**Related Functions**:
- [startSpindle](#startspindle)
- [writeToolCall](#writetoolcall)

---

## onAction

**Location**: `brother speedio.cps:2485`

**Signature**:
```javascript
function onAction(action)
```

**Description**: Processes manual NC action commands from Fusion 360. Currently supports ROTATE_WCS action to enable/disable WCS rotation for probing.

**Parameters**:
- `action` (string) - Action command in format "ACTION:PARAM"

**Returns**: void - No return value

**G-Code Impact**: May output WCS rotation codes via setProbeAngleMethod

**Called By**:
- onParameter (brother speedio.cps:2460)

**Usage Example**:
```javascript
// From onParameter
case "action":
  onAction(value);
  break;
```

**Related Functions**:
- [onParameter](#onparameter)
- [parseChoice](#parsechoice)
- [setProbeAngleMethod](#setprobeanglemethod)

---

## onCircular

**Location**: `brother speedio.cps:3861`

**Signature**:
```javascript
function onCircular(clockwise, cx, cy, cz, x, y, z, feed)
```

**Description**: Outputs circular interpolation moves (G2/G3) in IJK or radius mode. Handles full circles, helical moves, and different planes (XY, ZX, YZ). Linearizes if necessary (active G68 rotation, unsupported plane).

**Parameters**:
- `clockwise` (boolean) - True for CW (G2), false for CCW (G3)
- `cx`, `cy`, `cz` (number) - Arc center coordinates
- `x`, `y`, `z` (number) - Arc end coordinates
- `feed` (number) - Feedrate

**Returns**: void - No return value

**G-Code Impact**: Outputs G17/G18/G19 and G2/G3 with IJK or R parameters

**Called By**:
- Post kernel during circular moves

**Usage Example**:
```javascript
// Post kernel calls this automatically for circular toolpath segments
// Outputs: G17 G2 X10. Y20. I5. J10. F500
```

**Related Functions**:
- [onLinear](#onlinear)
- [onRapid](#onrapid)

---

## onClose

**Location**: `brother speedio.cps:2380`

**Signature**:
```javascript
function onClose()
```

**Description**: Program end handler. Closes inspection files, turns off coolant, activates parts counters, retracts to home position with first tool, disables smoothing, and outputs M30 program end.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs coolant off, washdown, M211-M214 counters, M159, G100 return home, M30

**Called By**:
- Post kernel at program end

**Usage Example**:
```javascript
// Automatically called by post kernel
// Outputs final program cleanup and M30
```

**Related Functions**:
- [onOpen](#onopen)
- [setCoolant](#setcoolant)
- [getRetractParameters](#getretractparameters)
- [setSmoothing](#setsmoothing)

---

## onCommand

**Location**: `brother speedio.cps:2187`

**Signature**:
```javascript
function onCommand(command)
```

**Description**: Processes machine commands from toolpath including coolant, spindle, stop, tool change, verification, and tool measurement. Outputs appropriate M-codes and handles G100 tool change sequence.

**Parameters**:
- `command` (Command) - Command constant (COMMAND_STOP, COMMAND_LOAD_TOOL, etc.)

**Returns**: void - No return value

**G-Code Impact**: Outputs M0, M1, M3, M4, M5, M19, G100, and various tool measurement macros

**Called By**:
- Post kernel and other functions (onSection, onSectionEnd, etc.)

**Usage Example**:
```javascript
// From onSection
onCommand(COMMAND_START_CHIP_TRANSPORT);

// Outputs appropriate M-codes based on command type
```

**Related Functions**:
- [setCoolant](#setcoolant)
- [writeToolCall](#writetoolcall)
- [writeToolMeasureBlock](#writetoolmeasureblock)
- [prepareForToolCheck](#preparefortoolcheck)

---

## onComment

**Location**: `brother speedio.cps:2882`

**Signature**:
```javascript
function onComment(text)
```

**Description**: Outputs a comment from the CAM system. Simple wrapper that calls writeComment.

**Parameters**:
- `text` (string) - Comment text

**Returns**: void - No return value

**G-Code Impact**: Outputs formatted comment with prefix/suffix

**Called By**:
- Post kernel when comments appear in toolpath

**Usage Example**:
```javascript
// Post kernel calls this for manual NC comments
onComment("This is a comment from CAM");
// Outputs: (THIS IS A COMMENT FROM CAM)
```

**Related Functions**:
- [writeComment](#writecomment)
- [formatComment](#formatcomment)

---

## onCycle

**Location**: `brother speedio.cps:827`

**Signature**:
```javascript
function onCycle()
```

**Description**: Called at the start of a canned cycle. Ensures plane selection is set to G17 (XY plane) for drilling cycles.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs G17

**Called By**:
- Post kernel at cycle start

**Usage Example**:
```javascript
// Post kernel calls this before drilling/probing cycles
// Outputs: G17
```

**Related Functions**:
- [onCycleEnd](#oncycleend)
- [onCyclePoint](#oncyclepoint)
- [writeDrillCycle](#writedrillcycle)

---

## onCycleEnd

**Location**: `brother speedio.cps:2117`

**Signature**:
```javascript
function onCycleEnd()
```

**Description**: Called at the end of a canned cycle. Outputs protected retract for probing or G80 for drilling. Resets Z output.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs G80 or probe retract macro (G65 P8810/P8703)

**Called By**:
- Post kernel at cycle end

**Usage Example**:
```javascript
// Post kernel calls this after all cycle points processed
// For drilling: G80
// For probing: G65 P8810 Z... F... (protected retract)
```

**Related Functions**:
- [onCycle](#oncycle)
- [onCyclePoint](#oncyclepoint)
- [protectedProbeMove](#protectedprobemove)

---

## onCyclePoint

**Location**: `brother speedio.cps:862`

**Signature**:
```javascript
function onCyclePoint(x, y, z)
```

**Description**: Processes each point in a canned cycle. Routes to inspection, probing, or drilling functions based on operation type.

**Parameters**:
- `x`, `y`, `z` (number) - Cycle point coordinates

**Returns**: void - No return value

**G-Code Impact**: Varies - outputs drill cycles or probe macros

**Called By**:
- Post kernel for each cycle point

**Usage Example**:
```javascript
// Post kernel calls this for each hole/probe point
// Routes to writeDrillCycle or writeProbeCycle
```

**Related Functions**:
- [writeDrillCycle](#writedrillcycle)
- [writeProbeCycle](#writeprobecycle)
- [onCycle](#oncycle)

---

## onDwell

**Location**: `brother speedio.cps:815`

**Signature**:
```javascript
function onDwell(seconds)
```

**Description**: Outputs a dwell (pause) command for the specified time. Clamps value to valid range and outputs G4 with P parameter.

**Parameters**:
- `seconds` (number) - Dwell time in seconds (0-99999.999)

**Returns**: void - No return value

**G-Code Impact**: Outputs G94 G4 P(seconds)

**Called By**:
- Post kernel and onSection (for coolant dwell)

**Usage Example**:
```javascript
// From onSection for coolant dwell
if (tool.coolant == COOLANT_FLOOD) {
  if (lastCoolant == COOLANT_FLOOD_THROUGH_TOOL || lastCoolant == COOLANT_FLOOD) {
    onDwell(0.1);
  } else {
    onDwell(0.6);
  }
}
// Outputs: G94 G4 P0.6
```

**Related Functions**:
- [onSection](#onsection)

---

## onLinear

**Location**: `brother speedio.cps:3770`

**Signature**:
```javascript
function onLinear(_x, _y, _z, feed)
```

**Description**: Outputs linear feed moves (G1) with optional radius compensation activation. Handles pending radius compensation by outputting G41/G42 on first move.

**Parameters**:
- `_x`, `_y`, `_z` (number) - Linear move coordinates
- `feed` (number) - Feedrate

**Returns**: void - No return value

**G-Code Impact**: Outputs G1 (and G41/G42 if radius compensation pending)

**Called By**:
- Post kernel during linear moves

**Usage Example**:
```javascript
// Post kernel calls this for linear cutting moves
// Outputs: G1 X10.5 Y20.3 Z-5.0 F500
```

**Related Functions**:
- [onRapid](#onrapid)
- [onCircular](#oncircular)
- [getFeed](#getfeed)

---

## onLinear5D

**Location**: `brother speedio.cps:3829`

**Signature**:
```javascript
function onLinear5D(_x, _y, _z, _a, _b, _c, feed, feedMode)
```

**Description**: Outputs 5-axis simultaneous linear moves with rotary axes. Handles inverse time or DPM feedrates for multi-axis operations.

**Parameters**:
- `_x`, `_y`, `_z` (number) - Linear coordinates
- `_a`, `_b`, `_c` (number) - Rotary coordinates
- `feed` (number) - Feedrate value
- `feedMode` (FeedMode) - Feed mode (FEED_INVERSE_TIME or FEED_DPM)

**Returns**: void - No return value

**G-Code Impact**: Outputs G93/G94/G95 and G1 with XYZABC coordinates

**Called By**:
- Post kernel during 5-axis simultaneous moves

**Usage Example**:
```javascript
// Post kernel calls this for 5-axis simultaneous toolpaths
// Outputs: G93 G1 X10. Y20. Z5. A15. B30. C45. F0.125
```

**Related Functions**:
- [onRapid5D](#onrapid5d)
- [onLinear](#onlinear)
- [getFeed](#getfeed)

---

## onMovement

**Location**: `brother speedio.cps:2443`

**Signature**:
```javascript
function onMovement(movement)
```

**Description**: Called before each toolpath movement type change. Handles rapid transitions property by disabling smoothing during non-cutting moves (leads, links) in adaptive/pocket operations.

**Parameters**:
- `movement` (Movement) - Movement type constant

**Returns**: void - No return value

**G-Code Impact**: May output M298 L0 to disable smoothing during transitions

**Called By**:
- Post kernel when movement type changes

**Usage Example**:
```javascript
// Post kernel calls this before movement type changes
// For adaptive with rapidTransitions enabled:
// MOVEMENT_LEAD_OUT -> M298 L0 (smoothing off)
// MOVEMENT_CUTTING -> M298 L5 (smoothing on)
```

**Related Functions**:
- [setSmoothing](#setsmoothing)

---

## onMoveToSafeRetractPosition

**Location**: `brother speedio.cps:2140`

**Signature**:
```javascript
function onMoveToSafeRetractPosition()
```

**Description**: Retracts to safe position before indexing rotary axes. Part of the rewind/reconfigure sequence. Cancels TCP if active.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs Z retract and possibly TCP cancel (G49)

**Called By**:
- Post kernel during rewind sequence

**Usage Example**:
```javascript
// Post kernel calls this during rewind/reconfigure
// Outputs retract and TCP cancel before rotary repositioning
```

**Related Functions**:
- [onRotateAxes](#onrotateaxes)
- [onReturnFromSafeRetractPosition](#onreturnfromsaferetractposition)
- [writeRetract](#writeretract)

---

## onOpen

**Location**: `brother speedio.cps:624`

**Signature**:
```javascript
function onOpen()
```

**Description**: Program initialization. Defines machine, activates configuration, writes program header, optional tool measurement, outputs initialization codes (G90, G40, G80, G94, G49), and validates parameters.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs program header, tool list, G0 G90 G40 G80, G94 G49 Z[#5003], WCS, tap accel M-code

**Called By**:
- Post kernel at program start

**Usage Example**:
```javascript
// Post kernel calls this first
// Outputs complete program initialization
```

**Related Functions**:
- [onClose](#onclose)
- [defineMachine](#definemachine)
- [activateMachine](#activatemachine)
- [writeProgramHeader](#writeprogramheader)
- [writeMeasureTools](#writemeasuretools)

---

## onParameter

**Location**: `brother speedio.cps:2460`

**Signature**:
```javascript
function onParameter(name, value)
```

**Description**: Handles CAM parameters. Processes job-description, job-notes (outputs setup notes), and action commands.

**Parameters**:
- `name` (string) - Parameter name
- `value` (any) - Parameter value

**Returns**: void - No return value

**G-Code Impact**: May output notes as comments or action codes

**Called By**:
- Post kernel when parameters are set

**Usage Example**:
```javascript
// Post kernel calls this for setup parameters
// Outputs job notes and processes action commands
```

**Related Functions**:
- [onAction](#onaction)
- [writeNotes](#writenotes)

---

## onPassThrough

**Location**: `brother speedio.cps:2919`

**Signature**:
```javascript
function onPassThrough(text)
```

**Description**: Outputs manual NC commands from CAM without modification. Splits comma-separated commands and writes each as a block.

**Parameters**:
- `text` (string) - Manual NC text (comma-separated commands)

**Returns**: void - No return value

**G-Code Impact**: Outputs raw text as G-code blocks

**Called By**:
- Post kernel for manual NC passthrough

**Usage Example**:
```javascript
// Manual NC in CAM: "G53 Z0, M0"
// Outputs:
// (MANUAL NC PASSTHROUGH)
// G53 Z0
// M0
```

**Related Functions**:
- [writeBlock](#writeblock)

---

## onRadiusCompensation

**Location**: `brother speedio.cps:2911`

**Signature**:
```javascript
function onRadiusCompensation()
```

**Description**: Called when radius compensation mode changes. Sets pending radius compensation flag for activation on next linear move.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: None directly - sets pendingRadiusCompensation for onLinear

**Called By**:
- Post kernel when radius compensation changes

**Usage Example**:
```javascript
// Post kernel calls this when compensation changes
// Next onLinear outputs G41/G42
```

**Related Functions**:
- [onLinear](#onlinear)

---

## onRapid

**Location**: `brother speedio.cps:3751`

**Signature**:
```javascript
function onRapid(_x, _y, _z)
```

**Description**: Outputs rapid traverse moves (G0). Uses protected probe moves if probe is active and not in production mode. Forces feed output after rapids.

**Parameters**:
- `_x`, `_y`, `_z` (number) - Rapid move coordinates

**Returns**: void - No return value

**G-Code Impact**: Outputs G0 or protected probe move macro

**Called By**:
- Post kernel during rapid moves

**Usage Example**:
```javascript
// Post kernel calls this for rapid positioning
// Outputs: G0 X50. Y100. Z25.
```

**Related Functions**:
- [onLinear](#onlinear)
- [protectedProbeMove](#protectedprobemove)
- [forceFeed](#forcefeed)

---

## onRapid5D

**Location**: `brother speedio.cps:3807`

**Signature**:
```javascript
function onRapid5D(_x, _y, _z, _a, _b, _c)
```

**Description**: Outputs 5-axis rapid traverse moves with rotary axes. Forces XYZ output for unoptimized toolpaths.

**Parameters**:
- `_x`, `_y`, `_z` (number) - Linear coordinates
- `_a`, `_b`, `_c` (number) - Rotary coordinates

**Returns**: void - No return value

**G-Code Impact**: Outputs G0 with XYZABC coordinates

**Called By**:
- Post kernel during 5-axis rapid moves

**Usage Example**:
```javascript
// Post kernel calls this for 5-axis rapid positioning
// Outputs: G0 X10. Y20. Z5. A15. B30. C45.
```

**Related Functions**:
- [onRapid](#onrapid)
- [onLinear5D](#onlinear5d)
- [forceFeed](#forcefeed)

---

## onReturnFromSafeRetractPosition

**Location**: `brother speedio.cps:2162`

**Signature**:
```javascript
function onReturnFromSafeRetractPosition(_x, _y, _z)
```

**Description**: Returns from safe retract position after indexing rotaries. Part of rewind/reconfigure. Re-enables TCP and positions in XY then Z.

**Parameters**:
- `_x`, `_y`, `_z` (number) - Position to return to

**Returns**: void - No return value

**G-Code Impact**: Outputs G43 H... for TCP, then XY positioning, then Z

**Called By**:
- Post kernel during rewind sequence

**Usage Example**:
```javascript
// Post kernel calls this after onRotateAxes
// Outputs: G43 H1
//          G0 X10. Y20.
//          Z5.
```

**Related Functions**:
- [onMoveToSafeRetractPosition](#onmovetosaferetractposition)
- [onRotateAxes](#onrotateaxes)
- [getOffsetCode](#getoffsetcode)

---

## onRewindMachineEntry

**Location**: `brother speedio.cps:2135`

**Signature**:
```javascript
function onRewindMachineEntry(_a, _b, _c)
```

**Description**: Allows override of rewind logic. Currently returns false to use default rewind behavior.

**Parameters**:
- `_a`, `_b`, `_c` (number) - Target ABC angles

**Returns**: (boolean) - False to use default behavior

**G-Code Impact**: None - override hook

**Called By**:
- Post kernel during rewind sequence

**Usage Example**:
```javascript
// Post kernel calls this - currently uses default behavior
```

**Related Functions**:
- [onMoveToSafeRetractPosition](#onmovetosaferetractposition)
- [onRotateAxes](#onrotateaxes)

---

## onRotateAxes

**Location**: `brother speedio.cps:2149`

**Signature**:
```javascript
function onRotateAxes(_x, _y, _z, _a, _b, _c)
```

**Description**: Rotates axes to new position during rewind/reconfigure. Disables linear axes, calls 5D rapid for rotary positioning, updates current ABC, then re-enables linear axes.

**Parameters**:
- `_x`, `_y`, `_z` (number) - Linear position (ignored)
- `_a`, `_b`, `_c` (number) - Target rotary angles

**Returns**: void - No return value

**G-Code Impact**: Outputs G0 with ABC positioning

**Called By**:
- Post kernel during rewind sequence

**Usage Example**:
```javascript
// Post kernel calls this during rewind
// Outputs: G0 A45. C90.
```

**Related Functions**:
- [onMoveToSafeRetractPosition](#onmovetosaferetractposition)
- [onReturnFromSafeRetractPosition](#onreturnfromsaferetractposition)

---

## onSection

**Location**: `brother speedio.cps:710`

**Signature**:
```javascript
function onSection()
```

**Description**: Main section initialization. Handles tool changes, WCS changes, workplane changes, smoothing, coolant, spindle start, parametric feeds, and probe activation. Core logic for operation setup.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs tool change (G100), WCS, smoothing codes, coolant, probe activation, initial positioning

**Called By**:
- Post kernel at start of each operation

**Usage Example**:
```javascript
// Post kernel calls this for each new operation/section
// Outputs complete operation setup sequence
```

**Related Functions**:
- [onSectionEnd](#onsectionend)
- [writeToolCall](#writetoolcall)
- [writeWCS](#writewcs)
- [initializeSmoothing](#initializesmoothing)
- [setCoolant](#setcoolant)
- [writeInitialPositioning](#writeinitialPositioning)

---

## onSectionEnd

**Location**: `brother speedio.cps:2333`

**Signature**:
```javascript
function onSectionEnd()
```

**Description**: Section cleanup. Disables inverse time feed for multi-axis, outputs G49 for unoptimized multi-axis, cancels G54.2, handles tool break control, washdown coolant, and probe deactivation.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs G94, G49, G54.2 P0, M-codes for break control/washdown, probe off macros

**Called By**:
- Post kernel at end of each operation

**Usage Example**:
```javascript
// Post kernel calls this after each operation
// Outputs operation cleanup codes
```

**Related Functions**:
- [onSection](#onsection)
- [onCommand](#oncommand)
- [setCoolant](#setcoolant)
- [setProbeAngle](#setprobeangle)

---

## onSpindleSpeed

**Location**: `brother speedio.cps:823`

**Signature**:
```javascript
function onSpindleSpeed(spindleSpeed)
```

**Description**: Outputs spindle speed change during operation. Simple wrapper that outputs S code.

**Parameters**:
- `spindleSpeed` (number) - New spindle RPM

**Returns**: void - No return value

**G-Code Impact**: Outputs S(rpm)

**Called By**:
- Post kernel when spindle speed changes

**Usage Example**:
```javascript
// Post kernel calls this for speed changes
// Outputs: S5000
```

**Related Functions**:
- [startSpindle](#startspindle)

---

## parseChoice

**Location**: `brother speedio.cps:2523`

**Signature**:
```javascript
function parseChoice()
```

**Description**: Parses a text choice against a list of valid options. Converts YES/NO/TRUE/FALSE to boolean. Returns index or boolean based on match.

**Parameters**:
- `arguments[0]` (string) - Text to parse
- `arguments[1..n]` (string) - Valid choice options

**Returns**: (boolean|number|undefined) - Matched value or undefined

**G-Code Impact**: None - Helper for action parsing

**Called By**:
- onAction (brother speedio.cps:2485)

**Usage Example**:
```javascript
// From onAction
param = parseChoice(param, "YES", "NO", "TRUE", "FALSE");
if (param == undefined) {
  error(localize("Invalid ROTATE_WCS param. Use TRUE/FALSE"));
}
```

**Related Functions**:
- [onAction](#onaction)

---

## positionABC

**Location**: `brother speedio.cps:3143`

**Signature**:
```javascript
function positionABC(abc, force)
```

**Description**: Positions rotary axes to specified ABC angles. Retracts Z if not already retracted, unlocks multi-axis, outputs G0 with ABC, and updates current ABC state.

**Parameters**:
- `abc` (Vector) - Target ABC angles
- `force` (boolean) - If true, forces ABC output

**Returns**: void - No return value

**G-Code Impact**: Outputs Z retract and G0 A... B... C...

**Called By**:
- defineWorkPlane (brother speedio.cps:3068)
- setWorkPlane (brother speedio.cps:3959)

**Usage Example**:
```javascript
// From setWorkPlane for 3+2 indexing
if (_section.isMultiAxis() || isPolarModeActive()) {
  cancelWorkPlane();
  positionABC(abc, true);
}
```

**Related Functions**:
- [setWorkPlane](#setworkplane)
- [defineWorkPlane](#defineworkplane)
- [forceABC](#forceabc)

---

## prepareForToolCheck

**Location**: `brother speedio.cps:2799`

**Signature**:
```javascript
function prepareForToolCheck()
```

**Description**: Prepares machine for tool measurement/break detection by turning off coolant and stopping spindle.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs coolant off and M5 (stop spindle)

**Called By**:
- onCommand (brother speedio.cps:2187) - For COMMAND_BREAK_CONTROL
- writeToolMeasureBlock (brother speedio.cps:2082)

**Usage Example**:
```javascript
// From onCommand COMMAND_BREAK_CONTROL
if (!toolChecked) {
  writeln("");
  writeComment("Performing tool break detection");
  prepareForToolCheck();
  // ... output measurement macro
}
```

**Related Functions**:
- [writeToolMeasureBlock](#writetoolmeasureblock)
- [setCoolant](#setcoolant)
- [onCommand](#oncommand)

---

## printProbeResults

**Location**: `brother speedio.cps:706`

**Signature**:
```javascript
function printProbeResults()
```

**Description**: Determines if probe results should be printed to DPRNT file. Returns true only for Renishaw probing with printResults parameter enabled.

**Parameters**: None

**Returns**: (boolean) - True if results should be printed

**G-Code Impact**: None - Controls DPRNT output in probe cycles

**Called By**:
- writeProbeCycle (brother speedio.cps:1139)
- inspectionCreateResultsFileHeader (brother speedio.cps:4169)

**Usage Example**:
```javascript
// From writeProbeCycle
if (printProbeResults()) {
  writeProbingToolpathInformation(z - cycle.depth + tool.diameter / 2);
  inspectionWriteCADTransform();
  inspectionWriteWorkplaneTransform();
}
```

**Related Functions**:
- [writeProbeCycle](#writeprobecycle)
- [inspectionCreateResultsFileHeader](#inspectioncreateresultsfileheader)

---

## protectedProbeMove

**Location**: `brother speedio.cps:844`

**Signature**:
```javascript
function protectedProbeMove(cycle, x, y, z)
```

**Description**: Outputs protected positioning moves for probe operations using G65 P8810 (Renishaw) or P8703 (Blum). Moves in safe sequence: Z up first, then XY, then Z down.

**Parameters**:
- `cycle` (Cycle) - Probe cycle object (for feedrate, can be undefined)
- `x`, `y`, `z` (number) - Target coordinates

**Returns**: void - No return value

**G-Code Impact**: Outputs G65 P8810/P8703 protected move macros

**Called By**:
- onRapid (brother speedio.cps:3751) - When probe active
- writeProbeCycle (brother speedio.cps:1139) - Throughout probe cycles
- writeInitialPositioning (brother speedio.cps:4054) - When probe active

**Usage Example**:
```javascript
// From writeProbeCycle probing-x
protectedProbeMove(cycle, x, y, z - cycle.depth);
writeBlock(
  gFormat.format(65), "P" + 8811,
  "X" + xyzFormat.format(x + approach(cycle.approach1) * (cycle.probeClearance + tool.diameter / 2)),
  // ... probe parameters
);
```

**Related Functions**:
- [writeProbeCycle](#writeprobecycle)
- [onRapid](#onrapid)
- [getFeed](#getfeed)

---

## setCoolant

**Location**: `brother speedio.cps:3394`

**Signature**:
```javascript
function setCoolant(coolant)
```

**Description**: Outputs coolant codes by calling getCoolantCodes and writing them. Handles single-line or multi-line coolant output based on settings.

**Parameters**:
- `coolant` (CoolantMode) - Requested coolant mode

**Returns**: (undefined) - Always returns undefined

**G-Code Impact**: Outputs M8, M9, M494, M495, M402, M403 coolant codes

**Called By**:
- onClose (brother speedio.cps:2380)
- onCommand (brother speedio.cps:2187)
- onSection (brother speedio.cps:710)
- onSectionEnd (brother speedio.cps:2333)
- prepareForToolCheck (brother speedio.cps:2799)

**Usage Example**:
```javascript
// From onSection
setCoolant(tool.coolant); // writes the required coolant codes
```

**Related Functions**:
- [getCoolantCodes](#getcoolantcodes)
- [writeStartBlocks](#writestartblocks)

---

## setProbeAngle

**Location**: `brother speedio.cps:4268`

**Signature**:
```javascript
function setProbeAngle()
```

**Description**: Outputs probe angle rotation codes when angular probing has detected a rotation. Supports G54.4, G68, and AXIS_ROT methods based on machine configuration.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs G54.4 P..., G68 R[#144], or parameter updates for axis rotation

**Called By**:
- onSection (brother speedio.cps:710)
- onSectionEnd (brother speedio.cps:2333)

**Usage Example**:
```javascript
// From onSection after WCS
if (!isProbeOperation()) {
  setProbeAngle(); // output probe angle rotations if required
}
```

**Related Functions**:
- [setProbeAngleMethod](#setprobeanglemethod)
- [writeProbeCycle](#writeprobecycle)

---

## setProbeAngleMethod

**Location**: `brother speedio.cps:4318`

**Signature**:
```javascript
function setProbeAngleMethod()
```

**Description**: Determines the appropriate probe angle method (G54.4, G68, AXIS_ROT, or UNSUPPORTED) based on machine configuration and number of axes. Sets probeAngleMethod in settings.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: None - Configures method for setProbeAngle

**Called By**:
- onAction (brother speedio.cps:2485)
- writeProbeCycle (brother speedio.cps:1139) - For angular probing

**Usage Example**:
```javascript
// From writeProbeCycle for angled probing
if (currentSection.strategy == "probe") {
  setProbeAngleMethod();
  probeVariables.compensationXY = "X" + xyzFormat.format(0) + " Y" + xyzFormat.format(0);
}
```

**Related Functions**:
- [setProbeAngle](#setprobeangle)

---

## setSmoothing

**Location**: `brother speedio.cps:682`

**Signature**:
```javascript
function setSmoothing(mode)
```

**Description**: Enables or disables high-accuracy smoothing mode. Outputs M260-M269 (mode A), M280-M289 (mode B), or M298 L... based on smoothing mode property.

**Parameters**:
- `mode` (boolean) - True to enable smoothing, false to disable

**Returns**: void - No return value

**G-Code Impact**: Outputs M260-M269, M280-M289, or M298 L0/L1-L6

**Called By**:
- onClose (brother speedio.cps:2380)
- onMovement (brother speedio.cps:2443)
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection
setSmoothing(smoothing.isAllowed);

// For M298 mode with finishing level:
// Outputs: M298 L5
```

**Related Functions**:
- [initializeSmoothing](#initializesmoothing)
- [onMovement](#onmovement)

---

## setWorkPlane

**Location**: `brother speedio.cps:3959`

**Signature**:
```javascript
function setWorkPlane(abc)
```

**Description**: Sets the workplane for 3+2 operations using G68.2 tilted workplane or ABC positioning. Retracts Z, cancels TCP if needed, positions rotaries, and outputs workplane code.

**Parameters**:
- `abc` (Vector) - ABC Euler angles for workplane

**Returns**: void - No return value

**G-Code Impact**: Outputs Z retract, G68.2 for tilted workplane, or ABC positioning

**Called By**:
- defineWorkPlane (brother speedio.cps:3068)
- onClose (brother speedio.cps:2380)
- writeInitialPositioning (brother speedio.cps:4054)

**Usage Example**:
```javascript
// From defineWorkPlane for 3+2 operation
if (_section.isMultiAxis() || isPolarModeActive()) {
  cancelWorkPlane();
  positionABC(abc, true);
} else {
  setWorkPlane(abc);
}
```

**Related Functions**:
- [cancelWorkPlane](#cancelworkplane)
- [forceWorkPlane](#forceworkplane)
- [positionABC](#positionabc)

---

## startSpindle

**Location**: `brother speedio.cps:3216`

**Signature**:
```javascript
function startSpindle(tool, insertToolCall)
```

**Description**: Starts the spindle if required and not a non-spindle operation (tap/probe). Checks if speed has changed or direction changed before outputting.

**Parameters**:
- `tool` (Tool) - Current tool
- `insertToolCall` (boolean) - True if this is a tool change

**Returns**: void - No return value

**G-Code Impact**: Outputs S(rpm) M3/M4

**Called By**:
- onCommand (brother speedio.cps:2187) - After tool measurement
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection when not a tool change
if (insertToolCall) {
  writeToolCall(tool, insertToolCall);
} else {
  defineWorkPlane(currentSection, true);
  startSpindle(tool, insertToolCall);
}
```

**Related Functions**:
- [noSpindle](#nospindle)
- [onCommand](#oncommand)

---

## subprogramsAreSupported

**Location**: `brother speedio.cps:3062`

**Signature**:
```javascript
function subprogramsAreSupported()
```

**Description**: Checks if subprogram/subroutine functionality exists in the post. Returns true if subprogramState variable is defined.

**Parameters**: None

**Returns**: (boolean) - True if subprograms supported

**G-Code Impact**: None - Query function

**Called By**:
- Common functions framework

**Usage Example**:
```javascript
// From common functions
if (subprogramsAreSupported()) {
  // Handle subprogram logic
}
```

**Related Functions**:
- None - Standalone utility

---

## validateCommonParameters

**Location**: `brother speedio.cps:2680`

**Signature**:
```javascript
function validateCommonParameters()
```

**Description**: Validates common post parameters including tool data, work offset consistency, multi-axis requirements, inverse time feed support, and TCP configuration.

**Parameters**: None

**Returns**: void - No return value (throws errors if validation fails)

**G-Code Impact**: None - Validation only

**Called By**:
- onOpen (brother speedio.cps:624)

**Usage Example**:
```javascript
// From onOpen
writeComment("File output in " + (unit == 1 ? "MM" : "inches"));
validateCommonParameters();
```

**Related Functions**:
- [validateToolData](#validatetooldata)
- [getSetting](#getsetting)

---

## validateToolData

**Location**: `brother speedio.cps:2708`

**Signature**:
```javascript
function validateToolData()
```

**Description**: Validates tool numbers, length offsets, diameter offsets, and spindle speeds are within machine limits. Outputs warnings if any values exceed configured maximums.

**Parameters**: None

**Returns**: void - No return value (outputs warnings)

**G-Code Impact**: None - Validation only

**Called By**:
- validateCommonParameters (brother speedio.cps:2680)

**Usage Example**:
```javascript
// From validateCommonParameters
function validateCommonParameters() {
  validateToolData();
  // ... more validation
}
```

**Related Functions**:
- [validateCommonParameters](#validatecommonparameters)

---

## writeBlock

**Location**: `brother speedio.cps:2810`

**Signature**:
```javascript
function writeBlock()
```

**Description**: Writes a G-code block with optional sequence numbers and optional block prefix (/). Handles sequence number settings (always, never, tool change only). Uses a line-based increment system where the sequence number only increments every 42 lines of output.

**Parameters**:
- `...arguments` (any) - Variable arguments to format into block

**Returns**: void - No return value

**G-Code Impact**: Outputs formatted G-code block with optional N... sequence number

**Sequence Number Logic**:
- Maintains `lineCounter` to track lines since last sequence number increment
- Increments `lineCounter` on each `writeBlock()` call when sequence numbers are enabled
- Only increments `sequenceNumber` when `lineCounter >= linesPerSequenceIncrement` (42 lines)
- Example: Lines 1-42 use N10, lines 43-84 use N15, lines 85-126 use N20, etc.

**Called By**:
- Nearly every function that outputs G-code

**Usage Example**:
```javascript
// Throughout code
writeBlock(gMotionModal.format(0), xOutput.format(x), yOutput.format(y));
// Line 1-42: N10 G0 X50.0 Y100.0
// Line 43-84: N15 G0 X60.0 Y110.0
// Line 85-126: N20 G0 X70.0 Y120.0

// Actual implementation (Lines 2817-2839):
if (getProperty("showSequenceNumbers") == "true") {
  if (sequenceNumber == undefined || sequenceNumber >= settings.maximumSequenceNumber) {
    sequenceNumber = getProperty("sequenceNumberStart");
    lineCounter = 0;
  }

  lineCounter++;  // Increment line counter

  // Output block with current sequence number
  writeWords2("N" + sequenceNumber, arguments);

  // Only increment sequence number every 42 lines
  if (lineCounter >= linesPerSequenceIncrement) {
    sequenceNumber += getProperty("sequenceNumberIncrement");
    lineCounter = 0;
  }
}
```

**Related Functions**:
- [writeToolBlock](#writetoolblock)
- [writeStartBlocks](#writestartblocks)

---

## writeComment

**Location**: `brother speedio.cps:2869`

**Signature**:
```javascript
function writeComment(text)
```

**Description**: Outputs formatted comments. Splits multi-line text and outputs each line separately through formatComment.

**Parameters**:
- `text` (string) - Comment text (can contain line breaks)

**Returns**: void - No return value

**G-Code Impact**: Outputs formatted comments with prefix/suffix (e.g., "(COMMENT)")

**Called By**:
- Many functions throughout for documentation and debugging

**Usage Example**:
```javascript
// From onSection
writeln("");
writeComment(getParameter("operation-comment", ""));
// Outputs: (ADAPTIVE CLEARING)
```

**Related Functions**:
- [formatComment](#formatcomment)
- [onComment](#oncomment)
- [writeNotes](#writenotes)

---

## writeDrillCycle

**Location**: `brother speedio.cps:877`

**Signature**:
```javascript
function writeDrillCycle(cycle, x, y, z)
```

**Description**: Outputs drilling cycle G-codes (G81, G82, G73, G83, G84, G74, G77, G78, G85, G86, G87, G88, G89, G76). Handles all standard drilling, boring, and tapping cycles with appropriate parameters.

**Parameters**:
- `cycle` (Cycle) - Drill cycle object
- `x`, `y`, `z` (number) - Hole position

**Returns**: void - No return value

**G-Code Impact**: Outputs G98/G99 and G81-G89, G76, G77, G78 with cycle parameters

**Called By**:
- onCyclePoint (brother speedio.cps:862)

**Usage Example**:
```javascript
// From onCyclePoint
if (isProbeOperation()) {
  writeProbeCycle(cycle, x, y, z);
} else {
  writeDrillCycle(cycle, x, y, z);
}
```

**Related Functions**:
- [getCommonCycle](#getcommoncycle)
- [onCyclePoint](#oncyclepoint)

---

## writeExtraBlumProbing

**Location**: `brother speedio.cps:1930`

**Signature**:
```javascript
function writeExtraBlumProbing(cycle)
```

**Description**: Outputs additional Blum probe parameters for tolerances (P8707) and tool wear updates (P8706) based on cycle settings.

**Parameters**:
- `cycle` (Cycle) - Probe cycle with tolerance and tool wear settings

**Returns**: void - No return value

**G-Code Impact**: Outputs G65 P8707 (tolerances) and G65 P8706 (tool wear) as needed

**Called By**:
- writeProbeCycle (brother speedio.cps:1139) - After each Blum probe cycle

**Usage Example**:
```javascript
// From writeProbeCycle after Blum probing-x
writeBlock(
  gFormat.format(65), "P" + 8700,
  // ... probe parameters
);
writeExtraBlumProbing(cycle);
```

**Related Functions**:
- [writeProbeCycle](#writeprobecycle)

---

## writeInitialPositioning

**Location**: `brother speedio.cps:4054`

**Signature**:
```javascript
function writeInitialPositioning(position, isRequired, codes1, codes2)
```

**Description**: Writes initial positioning to operation start point. Handles full positioning with tool length compensation or simple positioning. Supports TWP prepositioning for multi-axis.

**Parameters**:
- `position` (Vector) - Initial position to move to
- `isRequired` (boolean) - True for full positioning, false for simple
- `codes1`, `codes2` (string) - Optional additional codes for positioning blocks

**Returns**: void - No return value

**G-Code Impact**: Outputs G90 G17, positioning moves with G43 H... tool length compensation

**Called By**:
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection
var initialPosition = getFramePosition(currentSection.getInitialPosition());
var isRequired = insertToolCall || retracted || !lengthCompensationActive;
writeInitialPositioning(initialPosition, isRequired);
```

**Related Functions**:
- [writeRetract](#writeretract)
- [setWorkPlane](#setworkplane)
- [protectedProbeMove](#protectedprobemove)

---

## writeMeasureTools

**Location**: `brother speedio.cps:2003`

**Signature**:
```javascript
function writeMeasureTools()
```

**Description**: Optionally measures all tools at program start and/or confirms tool lengths match CAM. Outputs optional blocks with tool measurement macros and length verification logic.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs optional M0, tool comments, G65 P8915/P9921 measurement macros, length checks

**Called By**:
- onOpen (brother speedio.cps:624)

**Usage Example**:
```javascript
// From onOpen
writeProgramHeader();
writeMeasureTools();
```

**Related Functions**:
- [writeToolMeasureBlock](#writetoolmeasureblock)
- [onOpen](#onopen)

---

## writeNotes

**Location**: `brother speedio.cps:2558`

**Signature**:
```javascript
function writeNotes(text)
```

**Description**: Formats and writes multi-line text notes as comments. Splits on newlines, trims whitespace, and outputs each non-empty line.

**Parameters**:
- `text` (string) - Multi-line note text

**Returns**: void - No return value

**G-Code Impact**: Outputs formatted comments

**Called By**:
- onParameter (brother speedio.cps:2460)
- writeProgramHeader (brother speedio.cps:3638)

**Usage Example**:
```javascript
// From onParameter for job-notes
if (!firstNote && getProperty("showNotes")) {
  writeln("");
  writeNotes("Setup. " + jobDescription);
  writeNotes(value);
}
```

**Related Functions**:
- [writeComment](#writecomment)
- [formatComment](#formatcomment)

---

## writeProbeCycle

**Location**: `brother speedio.cps:1139`

**Signature**:
```javascript
function writeProbeCycle(cycle, x, y, z)
```

**Description**: Outputs probe cycle macros for all supported probe types (surface, wall, channel, circular, rectangular, corner, angle, PCD). Handles both Renishaw (P8811-P8843) and Blum (P8700) probe systems.

**Parameters**:
- `cycle` (Cycle) - Probe cycle object
- `x`, `y`, `z` (number) - Probe point position

**Returns**: void - No return value

**G-Code Impact**: Outputs G65 P8811/P8812/P8814/P8815/P8816/P8819/P8823/P8843 (Renishaw) or G65 P8700 (Blum)

**Called By**:
- onCyclePoint (brother speedio.cps:862)

**Usage Example**:
```javascript
// From onCyclePoint
if (isProbeOperation()) {
  writeProbeCycle(cycle, x, y, z);
}
```

**Related Functions**:
- [protectedProbeMove](#protectedprobemove)
- [getProbingArguments](#getprobingarguments)
- [writeExtraBlumProbing](#writeextrablumprobing)
- [approach](#approach)
- [ensurePositiveAngle](#ensurepositiveangle)

---

## writeProbingToolpathInformation

**Location**: `brother speedio.cps:4256`

**Signature**:
```javascript
function writeProbingToolpathInformation(cycleDepth)
```

**Description**: Writes probe toolpath information to DPRNT results file including operation ID, comment, and cycle depth.

**Parameters**:
- `cycleDepth` (number) - Depth of probe cycle

**Returns**: void - No return value

**G-Code Impact**: Outputs DPRNT[TOOLPATHID*...] and DPRNT[TOOLPATH*...] or DPRNT[CYCLEDEPTH*...]

**Called By**:
- writeProbeCycle (brother speedio.cps:1139)

**Usage Example**:
```javascript
// From writeProbeCycle
if (printProbeResults()) {
  writeProbingToolpathInformation(z - cycle.depth + tool.diameter / 2);
  inspectionWriteCADTransform();
  inspectionWriteWorkplaneTransform();
}
```

**Related Functions**:
- [defineLocalVariable](#definelocalvariable)
- [formatLocalVariable](#formatlocalvariable)

---

## writeProgramHeader

**Location**: `brother speedio.cps:3638`

**Signature**:
```javascript
function writeProgramHeader()
```

**Description**: Writes program header with file info, date, setup notes, machine configuration, tool list with Z ranges, duplicate tool checks, and stock dimensions.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs formatted comments with program information

**Called By**:
- onOpen (brother speedio.cps:624)

**Usage Example**:
```javascript
// From onOpen
sequenceNumber = getProperty("sequenceNumberStart");
writeProgramHeader();
writeMeasureTools();
```

**Related Functions**:
- [writeComment](#writecomment)
- [writeNotes](#writenotes)
- [writeStock](#writestock)

---

## writeRetract

**Location**: `brother speedio.cps:4006`

**Signature**:
```javascript
function writeRetract()
```

**Description**: Retracts specified axes (X, Y, Z) to safe position using configured method (G28, G30, G53, or clearance height). Handles rotation cancellation if needed.

**Parameters**:
- `...arguments` (Axis) - Axes to retract (X, Y, Z)

**Returns**: void - No return value

**G-Code Impact**: Outputs G28/G30/G53 retract codes

**Called By**:
- Many functions throughout for safe retracts

**Usage Example**:
```javascript
// From onSection
if (insertToolCall || newWorkOffset || newWorkPlane || smoothing.cancel) {
  if (!insertToolCall && (newWorkOffset || newWorkPlane)) {
    writeRetract(Z); // retract
    forceXYZ();
  }
}
```

**Related Functions**:
- [getRetractParameters](#getretractparameters)
- [cancelWorkPlane](#cancelworkplane)

---

## writeStartBlocks

**Location**: `brother speedio.cps:2897`

**Signature**:
```javascript
function writeStartBlocks(isRequired, code)
```

**Description**: Executes provided code function, optionally making blocks optional (/) if not required and safeStartAllOperations is enabled.

**Parameters**:
- `isRequired` (boolean) - True for required blocks, false for optional
- `code` (function) - Function to execute that writes blocks

**Returns**: void - No return value

**G-Code Impact**: May add / prefix to blocks if optional

**Called By**:
- Multiple functions for conditional optional block output

**Usage Example**:
```javascript
// From writeWCS
writeStartBlocks(wcsIsRequired, function () {
  writeBlock(section.wcs);
});
```

**Related Functions**:
- [writeBlock](#writeblock)

---

## writeStock

**Location**: `brother speedio.cps:2540`

**Signature**:
```javascript
function writeStock()
```

**Description**: Writes stock dimensions and WCS location as comments if stock parameters are available.

**Parameters**: None

**Returns**: void - No return value

**G-Code Impact**: Outputs formatted comments with stock size and WCS bounds

**Called By**:
- writeProgramHeader (brother speedio.cps:3638)

**Usage Example**:
```javascript
// From writeProgramHeader
// Write stock
writeStock();
```

**Related Functions**:
- [writeProgramHeader](#writeprogramheader)
- [writeComment](#writecomment)

---

## writeToolBlock

**Location**: `brother speedio.cps:2889`

**Signature**:
```javascript
function writeToolBlock()
```

**Description**: Writes a block with sequence numbers based on showSequenceNumbers property. Forces sequence numbers for tool changes if set to "toolChange".

**Parameters**:
- `...arguments` (any) - Block content to write

**Returns**: void - No return value

**G-Code Impact**: Outputs block with sequence number if enabled

**Called By**:
- onCommand (brother speedio.cps:2187) - For G100 tool change

**Usage Example**:
```javascript
// From onCommand COMMAND_LOAD_TOOL
writeToolBlock(gFormat.format(100),
  "T" + toolFormat.format(tool.number),
  // ... more parameters
);
```

**Related Functions**:
- [writeBlock](#writeblock)

---

## writeToolCall

**Location**: `brother speedio.cps:3188`

**Signature**:
```javascript
function writeToolCall(tool, insertToolCall)
```

**Description**: Handles tool change sequence by forcing modals, disabling length compensation, outputting optional stop, and calling COMMAND_LOAD_TOOL.

**Parameters**:
- `tool` (Tool) - Tool to load
- `insertToolCall` (boolean) - True if this is a tool change

**Returns**: void - No return value

**G-Code Impact**: Outputs M1 (optional stop) and triggers G100 via onCommand

**Called By**:
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection
if (insertToolCall) {
  // G100 tool call macro handles retract, positioning, spindle
  retracted = true;
  writeToolCall(tool, insertToolCall);
}
```

**Related Functions**:
- [onCommand](#oncommand)
- [forceModals](#forcemodals)

---

## writeToolMeasureBlock

**Location**: `brother speedio.cps:2082`

**Signature**:
```javascript
function writeToolMeasureBlock(tool, preMeasure)
```

**Description**: Outputs tool measurement macro (Renishaw P9921 or Blum P8915). Handles tool loading, spindle orientation, and special offsets for large face/slot mills.

**Parameters**:
- `tool` (Tool) - Tool to measure
- `preMeasure` (boolean) - True if pre-measuring (includes tool change)

**Returns**: void - No return value

**G-Code Impact**: Outputs T... M6, M19, G65 P8915/P9921 measurement macro

**Called By**:
- onCommand (brother speedio.cps:2187) - For COMMAND_TOOL_MEASURE
- writeMeasureTools (brother speedio.cps:2003)

**Usage Example**:
```javascript
// From onCommand after COMMAND_LOAD_TOOL
if (measureTool) {
  writeToolMeasureBlock(tool, false);
  setCoolant(tool.coolant);
  startSpindle(tool, true);
}
```

**Related Functions**:
- [prepareForToolCheck](#preparefortoolcheck)
- [onCommand](#oncommand)

---

## writeWCS

**Location**: `brother speedio.cps:3172`

**Signature**:
```javascript
function writeWCS(section, wcsIsRequired)
```

**Description**: Outputs work coordinate system (WCS) code if it has changed. Optionally cancels tilted workplane first based on settings.

**Parameters**:
- `section` (Section) - Section with WCS to output
- `wcsIsRequired` (boolean) - True to force WCS output

**Returns**: void - No return value

**G-Code Impact**: Outputs G54-G59, G54.1 P..., or G54-G59 G54.2 P...

**Called By**:
- onSection (brother speedio.cps:710)

**Usage Example**:
```javascript
// From onSection
var wcsIsRequired = true;
if (insertToolCall) {
  currentWorkOffset = undefined;
  wcsIsRequired = newWorkOffset || insertToolCall;
  writeBlock(gRotationModal.format(69)); // cancel frame
}
writeWCS(currentSection, wcsIsRequired);
```

**Related Functions**:
- [cancelWorkPlane](#cancelworkplane)
- [forceWorkPlane](#forceworkplane)
- [writeStartBlocks](#writestartblocks)

---

## Summary

This documentation covers all **80 functions** in the Brother Speedio CNC post processor, organized into logical categories for easy navigation. Each function includes:

- Exact file location and line number
- Function signature with parameters
- Detailed description of purpose
- Parameter documentation
- Return value information
- G-code impact analysis
- Call graph (what calls it)
- Real usage examples from the codebase
- Related function cross-references

The post processor handles:
- 3-axis, 3+2, and 5-axis simultaneous machining
- Renishaw and Blum probe systems
- Tool measurement and break detection
- Advanced smoothing/high-accuracy modes (M260/M280/M298)
- G68.2 tilted workplanes
- Comprehensive drilling and tapping cycles
- Parametric feeds
- Multi-coolant modes
- Inspection/results file output (DPRNT)

**Key Reference**: See [POST_PROCESSOR_FLOW.md](POST_PROCESSOR_FLOW.md) for execution flow and lifecycle documentation.
