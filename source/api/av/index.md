# IIIF AV

## IIIF Presentation API 3.0 (Choicey McChoiceface) Changes

### 2. Resource Type Overview

#### 2.1. Basic Types

#### 2.2. Additional Types

### 3. Resource Properties

#### 3.1. Descriptive Properties

#### 3.3. Technical Properties

  * Rename `@id` to `id` [#590](https://github.com/IIIF/iiif.io/issues/590)
  * Rename `@type` to `type` [#590](https://github.com/IIIF/iiif.io/issues/590)
  * Add `duration` to support time. `duration` is a non-negative floating point number measured in seconds. Optional for Canvas and Content Resources. [#1069](https://github.com/IIIF/iiif.io/issues/1069)
  * A canvas _MUST_ have exactly one `width` and one `height`, or exactly one `duration`. It may have `width`, `height` and `duration`.
  * Add new property `timeMode`, for Annotations that applies to all resources in the Body of the Annotation that could be considered to have a duration, with possible values:
     * `trim` (default): If the content resource has a longer duration than the duration of portion of the canvas it is associated with, then at the end of the canvas's duration, the playback of the content resource _MUST_ also end. If the content resource has a shorter duration than the duration of the portion of the canvas it is associated with, then, for video resources, the last frame _SHOULD_ persist on-screen until the end of the canvas portion's duration. For example, a video of 120 seconds annotated to a canvas with a duration of 100 seconds would play only the first 60 seconds at normal speed.
     * `scale`: Fit the duration of content resource to the duration of the portion of the canvas it is associated with by scaling. For example, a video of 120 seconds annotated to a canvas with a duration of 60 seconds would be played at double-speed.
     * `loop`: If the content resource is shorter than the `duration` of the canvas, it _MUST_ be repeated to fill the entire duration. Resources longer than the `duration` _MUST_ be trimmed as described above. For example, if a 20 second duration audio stream is annotated onto a canvas with duration 30 seconds, it will be played one and a half times. 

    Note that the association of `timeMode` with the Annotation means that different resources in the body cannot have different values.  [#xxxx]()
    
  * Add new property `choiceHint` that can be associated with a Choice, with the possible values:
     * `client`: The client software is expected to select an appropriate option without user interaction.
     * `user`: The client software is expected to present an interface to allow the user to explicitly select an option.
     * In the absence of `choiceHint`, the client can use any algorithm or process to make the determination
  * Add a new `viewingHint` value of "none", that can be associated with an AnnotationCollection, AnnotationPage, Annotation, SpecificResource, or a Choice. If this is provided, then the client should not render the resource by default, but allow the user to turn it on and off.
  * Consider whether or not to rename `viewingHint` to `renderingHint` as "viewing" an audio file is weird. [#1073](https://github.com/IIIF/iiif.io/issues/1073)
  * Add a new `viewingHint` value of "auto-advance", that can be associated with Collection, Manifest, Sequence and Canvas. When the client reaches the end of a Canvas with a duration dimension that has (or is within a resource that has) this `viewingHint`, it _SHOULD_ immediately proceed to the next Canvas and render it. 
    * If there is no subsequent Canvas in the current context, then this `viewingHint` should be ignored.
    * When applied to a Collection, the client should treat the first Canvas of the next Manifest as following the last Canvas of the previous Manifest, respecting any `startCanvas` specified. 
* Add a new `viewingHint` value of "together", that can be associated with Collections. A client _SHOULD_ present all of the child manifests to the user at once in a separate viewing area with its own controls. Caveats:
    * Clients _SHOULD_ catch attempts to create too many viewing areas and not do that.
    * The `together` value _SHOULD NOT_ be interpreted as applying to the members of children.

#### 3.4. Linking Properties

#### 3.5. Paging Properties

### 4. JSON-LD Considerations

#### 4.1. URI Representation

#### 4.2. Repeated Properties

#### 4.3. Language of Property Values

 * Consider JSON-LD 1.1 Language Maps [#1074](https://github.com/IIIF/iiif.io/issues/1074).

#### 4.4. HTML Markup in Property Values

#### 4.5. Linked Data Context and Extensions

 * Explain the import of the Web Annotation context

### 5. Resource Structure

#### 5.1. Manifest

#### 5.2. Sequence

#### 5.3. Canvas

  * Remove `images` and `otherContent` [#1068](https://github.com/IIIF/iiif.io/issues/1068)
  * Add a new property, `content`, which is an ordered list of `AnnotationPage` resources.  These pages may be either inline or by reference to their URIs.  The position in the overall order of Annotations across pages determines the z-index of where the resource should be painted, with the first annotation's body resource being the bottom-most z-index. [#1068](https://github.com/IIIF/iiif.io/issues/1068)
  * Definition of Canvas changed to include duration, no longer just an aspect ratio.
  * `sc:painting` (and all `sc:` motivations)  will be changed to, e.g., `painting`
  * Similarly, `oa:Annotation`, etc. will become `Annotation`, etc.
  * Explain that height and width are always `scale`, whereas duration can be changed with `timeMode`.
  * Explain that an Annotation can relate a Canvas to another Canvas
      * Including that the body Canvas might be external
      * Warn that very deep recursion should be avoided
      * An error to create a loop

#### 5.4. Image Resources

  * Rename Section to "Content Resources"
  * Rework to describe how to associate any content resource with a Canvas via an Annotation, rather than just images.
  * Remove all details about renamed properties, replace with Web Annotation context.
  * As part of 5.4, introduce Choice of Content Resource to describe `Choice`

#### 5.5. Annotation List

  * Remove AnnotationLists in favor of the equivalent structures from the Web Annotation model, `AnnotationPage`.

#### 5.6. Range

  * Is broken. See [#1070](https://github.com/IIIF/iiif.io/issues/1070).

#### 5.7. Layer

  * Remove Layers in favor of the equivalent structure from the Web Annotation model, `AnnotationCollection`.

#### 5.8. Collection

#### 5.9. Paging

### 6. Advanced Association Features

  * Remove section and export to a new document


## AV level0

### Captions via Annotations

https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/10.json

## AV level0 + auth

https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/14.json


## Bitstream APIs

A number of [use cases (labeled AV-bitstream-API)](https://github.com/IIIF/iiif-av/issues?q=is%3Aissue+is%3Aopen+label%3AAV-bitstream-API) require the generation of derivate video, audio and image resources from source video or audio resources. We imagine APIs that follow the style of the [Image API][image-api], with an information resource `info.json` (as used for level0 with authentication) and parameterized URI patterns for content resources. Work on these is deferred pending additional experience but the following sections offer suggested patterns for experimentation.

### Video Bitsream API

Possible URI pattern for requesting image information and content resources:

```
{scheme}://{server}{/prefix}/{identifier}
  /info.json

{scheme}://{server}{/prefix}/{identifier}
  /{timeRegion}/{spaceRegion}/{timeSize}/{spaceSize}/{quality}.{fmt}
```

Question - should there be a path element between `identifier` and `timeRegion` that can then be used to cleanly indivate whether we are accessing the `video`, `frame` or `audio` extraction API from the identifier video.

As with the [Image API][image-api] the order of the parameters in the URI corresponds with the order of transformation operations.

Extract:
iiif/opera-act-1/0,30/0,0,100,100/full/full/default.mp4

Refer:
iiif/opera-act-1.mp4#t=0,30&xywh=0,0,100,100

Use cases:

  * https://github.com/IIIF/iiif-av/issues/47
  * https://github.com/IIIF/iiif-av/issues/39
  * https://github.com/IIIF/iiif-av/issues/35 -- keep rights from source in derivative
  * https://github.com/IIIF/iiif-av/issues/25 -- video zoom
  * https://github.com/IIIF/iiif-av/issues/7 -- segmented for download
  * https://github.com/IIIF/iiif-av/issues/4 -- segmentation by time

### Audio Bitstream API

Possible URI pattern for requesting image information and content resources:

```
{scheme}://{server}{/prefix}/{identifier}
  /info.json

{scheme}://{server}{/prefix}/{identifier}
  /{timeRegion}/{timeSize}/{quality}.{fmt}
```

As with the [Image API][image-api] the order of the parameters in the URI corresponds with the order of transformation operations.

Use cases:

  * https://github.com/IIIF/iiif-av/issues/39
  * https://github.com/IIIF/iiif-av/issues/35 -- keep rights from source in derivative
  * https://github.com/IIIF/iiif-av/issues/4 -- segmentation by time


### Video Still Frame Extraction API

Described in use case [#5](https://github.com/IIIF/iiif-av/issues/5). Expectation is that this interface would be able to present a Image API at a given timepoint (frame) in the video. Possible URI patterns:

```
{scheme}://{server}{/prefix}/{identifier}
  /info.json  -- indicate in AV JSON that still frame extraction available?
  
  /timePoint/region/size/rotation/quality.fmt

  /timePoint -> 303 -> prefix/identifier/timePoint/info.json (Image API)
  /timePoint/info.json -- Image API Image Information
```

image-identifier = (video-identifier, timePoint)

Given understanding that different transcodings of a video will have different lengths, which Choice of the full video should be used to generate timePoint parameter for still-frame extraction?

### Video to Audio Extraction API

Present Audio Bitstream API for track from video? Use cases:

  * https://github.com/IIIF/iiif-av/issues/30


```
{scheme}://{server}{/prefix}/{identifier}
  /info.json  -- indicate in AV JSON that still frame extraction available?
  
  /track=??/{timeRegion}/{timeSize}/{quality}.{fmt}

  /track=?? -> 303 -> prefix/identifier/track/info.json (Image API)
  /track/info.json -- Audio API Image Information
```
