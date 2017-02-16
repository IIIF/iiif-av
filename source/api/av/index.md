
This document describes a set of changes to the [IIIF Presentation API](http://iiif.io/api/presentation/2.1/) to support A/V resources, and also proposes set of services for working with A/V bitstreams and integrating [IIIF Authentication API](http://iiif.io/api/auth/1.0/). Changes related to migration from the Open Annotation Data Model to the Web Annotation Data Model are also listed, as many of the [fixtures](https://github.com/IIIF/iiif-av/tree/master/source/api/av/examples) that were created as part of this process use properties from the Web Annotation JSON-LD context instead of the IIIF Presentation API context.

## IIIF Presentation API 3.0 (Choicey McChoiceface) Changes

### 3. Resource Properties

#### 3.3. Technical Properties

  * Rename `@id` to `id` [#590](https://github.com/IIIF/iiif.io/issues/590)
  * Rename `@type` to `type` [#590](https://github.com/IIIF/iiif.io/issues/590)
  * Add `duration` to support time. `duration` is a non-negative floating point number measured in seconds. Optional for Canvas and Content Resources. [#1069](https://github.com/IIIF/iiif.io/issues/1069)
  * A canvas _MUST_ have exactly one `width` and one `height`, or exactly one `duration`. It may have `width`, `height` and `duration`.
  * Add new property `timeMode`, for Annotations that applies to all resources in the Body of the Annotation that could be considered to have a duration [#1075](https://github.com/IIIF/iiif.io/issues/1075). Possible values:
     * `trim` (default): If the content resource has a longer duration than the duration of portion of the canvas it is associated with, then at the end of the canvas's duration, the playback of the content resource _MUST_ also end. If the content resource has a shorter duration than the duration of the portion of the canvas it is associated with, then, for video resources, the last frame _SHOULD_ persist on-screen until the end of the canvas portion's duration. For example, a video of 120 seconds annotated to a canvas with a duration of 100 seconds would play only the first 60 seconds at normal speed.
     * `scale`: Fit the duration of content resource to the duration of the portion of the canvas it is associated with by scaling. For example, a video of 120 seconds annotated to a canvas with a duration of 60 seconds would be played at double-speed.
     * `loop`: If the content resource is shorter than the `duration` of the canvas, it _MUST_ be repeated to fill the entire duration. Resources longer than the `duration` _MUST_ be trimmed as described above. For example, if a 20 second duration audio stream is annotated onto a canvas with duration 30 seconds, it will be played one and a half times. 
     * Note that the association of `timeMode` with the Annotation means that different resources in the body cannot have different values.
    
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

### 4. JSON-LD Considerations

#### 4.3. Language of Property Values

 * Consider JSON-LD Language Maps [#755](https://github.com/IIIF/iiif.io/issues/755).

#### 4.5. Linked Data Context and Extensions

 * Use and explain the import of the Web Annotation context (changes several key names)

### 5. Resource Structure

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


### 6. Advanced Association Features

  * Remove section and export to a new document


## AV Description in the Presentation API

The modifications and additions to the [IIIF Presentation API](http://iiif.io/api/presentation/2.1/) described above facilitate use of static (level0) audio and video resources via the IIIF Presentation API, without the need to implement any additional API. The following examples demonstrate a number of different uses:

  * Captions via Annotations

Example: https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/10.json


## AV with Authentication


Example: https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/14.json

```javascript
  "body" : {  // or item in Choice
    "id": "http://example.org/foo.mp4",
    "type": "Video",
    "format": "video/mpeg4;codecs='...'",
    "service": {
      "id": "http://example.org/whatever-for-foo/services.json",
      "profile": "http://iiif.io/api/svc/0/profile.json",  
      "description": "Auth services live in this external service description JSON" 
    }
  }
```

## Bitstream APIs

A number of [use cases (labeled AV-bitstream-API)](https://github.com/IIIF/iiif-av/issues?q=is%3Aissue+is%3Aopen+label%3AAV-bitstream-API) require the generation of derivative video, audio and image resources from source video or audio resources. We imagine that each of these APIs will follow the style of the [Image API][image-api], being a resource-oriented, ordered set of parameters, expressed as path fragments. The base URIs of these APIs will be described, along with any authentication information for the AV resource, in a services description document.

```javascript
{ 
  "@context" : "http://iiif.io/api/svc/0/context.json",
  "id": "http://example.org/whatever/services.json",
  "profile": "http://iiif.io/api/svc/0/profile.json",  
  "for": "http://example.org/video/foo.mp4",
  "service": [ {
       // ... frame-extraction service here ...
       "id": "http:/example.org/frame-extract/foo",
       "profile": "http://iiif.io/api/av/0/v2i"

    },{
       // ... auth in here as above...
    }
  ]
}
```

The following sections describe the APIs for Audio and Video that might be described in the services document.

### Video Bitstream API

Possible URI pattern for requesting image information and content resources, where `av-api-id` is specified in the services description document:

```
{av-api-id}/info.json

{av-api-id}/{timeRegion}/{spaceRegion}/{timeSize}/{spaceSize}/{quality}.{fmt}
```

As with the [Image API][image-api] the order of the parameters in the URI corresponds with the order of transformation operations.

Extract:
iiif/opera-act-1/0,30/0,0,100,100/full/full/default.mp4

Refer:
iiif/opera-act-1.mp4#t=0,30&xywh=0,0,100,100

Use cases:

  * Keep rights from source in derivative [#35](https://github.com/IIIF/iiif-av/issues/35)
  * Video zoom: https://github.com/IIIF/iiif-av/issues/25
  * Segmented for download: https://github.com/IIIF/iiif-av/issues/7
  * Segmentation by time https://github.com/IIIF/iiif-av/issues/4
  * https://github.com/IIIF/iiif-av/issues/47
  * https://github.com/IIIF/iiif-av/issues/39

### Audio Bitstream API

Possible URI pattern for requesting image information and content resources, where `audio-api-id` is specified in the services description documen:

```
{audio-api-id}/info.json

{audio-api-id}/{timeRegion}/{timeSize}/{quality}.{fmt}
```

  * `timeRegion` is comma-separated pair of floating point number of seconds from 0, as per Media Fragments.  E.g. 5,15
  * `timeSize` is a floating point number of seconds, as a duration. E.g. 10
  * ??? track selection and manipulation

As with the [Image API][image-api] the order of the parameters in the URI corresponds with the order of transformation operations.

Use cases:

  * Keep rights from source in derivative: https://github.com/IIIF/iiif-av/issues/35
  * Segmentation by time https://github.com/IIIF/iiif-av/issues/4
  * https://github.com/IIIF/iiif-av/issues/39

#### Audio info.json

```javascript
{
  "id": "",
  "profile": "",
  "duration": 25.3,
  
}
```

### Video to Image API

Described in use case [#5](https://github.com/IIIF/iiif-av/issues/5). Expectation is that this interface would be able to present a Image API at a given timepoint in the video. Possible URI patterns, where `v2i-api-id` is specified in the services description document:

```
{v2i-api-id}/info.json

{v2i-api-id}/{timePoint}/info.json -- An Image API Image Information document

{v2i-api-id}/{timePoint}/{region}/{size}/{rotation}/{quality}.{fmt} -- Image API request at timePoint
```

`timePoint` is specified as the floating point number of seconds after the beginning of the video stream.
The remaining parameters are interpreted exactly as the Image API, and a Image API `info.json` should be provided at `{v2i-api-id}/timePoint/info.json`

#### Video to Image info.json format

```javascript
{
  "id": "http://example.org/prefix/v2i-api-id",
  "profile": "http://iiif.io/api/av/v2i/0/level0.json",
  "duration": 25.3,
  "timePoints": [ 0, 5.01, 10.04, 15.12, 20.18 ]
}
```

 * level0 - only the `timePoints` specified (cf sizes)
 * level1 - can also give other `timePoints`, and the given ones are preferred


### Video to Audio transformation

We assume that video to audio transformation can be subsumed in an Audio API service associated with a Video rather than an Audio source. Use case: [#30](https://github.com/IIIF/iiif-av/issues/30). 
