
## IIIF A/V

**Status of this Document** - _This is a working document. Ideas in this document will be discussed by the [IIIF AV Working Group](http://iiif.io/community/groups/av/) and may or may not be taken forward into any IIIF specifications. Please send feedback through the IIIF AV Working Group, via [AV use cases and issues on github](https://github.com/IIIF/iiif-av/issues), or on [iiif-discuss](https://groups.google.com/forum/#!forum/iiif-discuss)._

### Introduction

This document describes a set of changes to the [IIIF Presentation API](http://iiif.io/api/presentation/2.1/) to support A/V resources, and also proposes a set of services for working with A/V bitstreams and integrating with the [IIIF Authentication API](http://iiif.io/api/auth/1.0/). Changes related to migration from the Open Annotation Data Model to the Web Annotation Data Model are also listed, as many of the [fixtures](https://github.com/IIIF/iiif-av/tree/master/source/api/av/examples) that were created as part of this process use properties from the Web Annotation JSON-LD context instead of the IIIF Presentation API context.

### IIIF Presentation API 3.0 (Choicey McChoiceface) Changes

This section lists proposed changes to the [IIIF Presentation API](http://iiif.io/api/presentation/2.1/) under the appropriate section headings. It includes only changes that related to A/V support, there are also [Presentation API issues](https://github.com/IIIF/iiif.io/issues?q=is%3Aissue+is%3Aopen+label%3Apresentation).

#### 3. Resource Properties

##### 3.3. Technical Properties

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
     * In the absence of a `choiceHint`, the client can use any algorithm or process to make the determination.
  * Add a new `viewingHint` value of "none", that can be associated with an AnnotationCollection, AnnotationPage, Annotation, SpecificResource, or a Choice. If this is provided, then the client should not render the resource by default, but allow the user to turn it on and off.
  * Consider whether or not to rename `viewingHint` to `renderingHint` as "viewing" an audio file is weird. [#1073](https://github.com/IIIF/iiif.io/issues/1073)
  * Add a new `viewingHint` value of "auto-advance", that can be associated with Collection, Manifest, Sequence and Canvas. When the client reaches the end of a Canvas with a duration dimension that has (or is within a resource that has) this `viewingHint`, it _SHOULD_ immediately proceed to the next Canvas and render it.
    * If there is no subsequent Canvas in the current context, then this `viewingHint` should be ignored.
    * When applied to a Collection, the client should treat the first Canvas of the next Manifest as following the last Canvas of the previous Manifest, respecting any `startCanvas` specified.
* Add a new `viewingHint` value of "together", that can be associated with Collections. A client _SHOULD_ present all of the child manifests to the user at once in a separate viewing area with its own controls. Caveats:
    * Clients _SHOULD_ catch attempts to create too many viewing areas and not do that.
    * The `together` value _SHOULD NOT_ be interpreted as applying to the members of children.

#### 4. JSON-LD Considerations

##### 4.3. Language of Property Values

 * Consider JSON-LD Language Maps [#755](https://github.com/IIIF/iiif.io/issues/755).

##### 4.5. Linked Data Context and Extensions

 * Use and explain the import of the Web Annotation context (changes several key names) [#496](https://github.com/IIIF/iiif.io/issues/496).

#### 5. Resource Structure

##### 5.3. Canvas

  * Remove `images` and `otherContent` [#1068](https://github.com/IIIF/iiif.io/issues/1068)
  * Add a new property, `content`, which is an ordered list of `AnnotationPage` resources.  These pages may be either inline or by reference to their URIs.  The position in the overall order of Annotations across pages determines the z-index of where the resource should be painted, with the first annotation's body resource being the bottom-most z-index. [#1068](https://github.com/IIIF/iiif.io/issues/1068)
  * Definition of Canvas changed to include `duration`, no longer just an aspect ratio.
  * `sc:painting` (and all `sc:` motivations)  will be changed to, e.g., `painting`
  * Similarly, `oa:Annotation`, etc. will become `Annotation`, etc.
  * Explain that height and width are always `scale`, whereas handling of duration can be changed with `timeMode`.
  * Explain that an Annotation can relate a Canvas to another Canvas:
      * Including that the body Canvas might be external.
      * Warn that very deep recursion should be avoided.
      * It is an error to create a loop.

##### 5.4. Image Resources

  * Rename Section to "Content Resources".
  * Rework to describe how to associate any content resource with a Canvas via an Annotation, rather than just images.
  * Remove all details about renamed properties, replace with Web Annotation context.
  * As part of 5.4, introduce Choice of Content Resource to describe `Choice`

##### 5.5. Annotation List

  * Remove AnnotationLists in favor of the equivalent structures from the Web Annotation model, `AnnotationPage`.

##### 5.6. Range

  * Is broken. See [#1070](https://github.com/IIIF/iiif.io/issues/1070).

##### 5.7. Layer

  * Remove Layers in favor of the equivalent structure from the Web Annotation model, `AnnotationCollection`.


#### 6. Advanced Association Features

  * Remove section and export to a new document.


### A/V Description in the Presentation API

The modifications and additions to the [IIIF Presentation API](http://iiif.io/api/presentation/2.1/) described above facilitate use of static ("level 0") audio and video resources via the IIIF Presentation API, without the need to implement any additional API. The following examples demonstrate a number of different uses:

  * [Duration on a Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/1.json)
  * [Temporal Fragment of a Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/2a.json)
  * [Temporal Fragment of a Video](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/2b.json)
  * [Associate Audio with temporal only Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/4.json)
  * [Associate Video with Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/5.json)
  * [Associate Videos with times on a Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/6.json)
  * [Associate Videos with different spaces on a Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/7.json)
  * [Associate Videos with overlapping time and space on a Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/8.json)
  * [Associate Video and Audio on single Canvas \[e.g. Director's commentary\]](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/9.json)
  * [Associate Video with text overlaid at particular location on Canvas \[subtitles\]](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/10.json)
  * [Associate Video with subtitle annotations](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/11.json)
  * [Associate Video with WebVTT subtitle file](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/11a.json)
  * [Associate Video with subtitle annotations and WebVTT subtitle file](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/11b.json)
  * [Associate multiple Video (or Audio) representations as Choice](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/12.json)
  * [Associate multiple Video representations as Choice, some HLS/MPEG-DASH](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/13.json)
  * [Associate an authentication service in remote services for video](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/14.json)
  * [Audio annotation demonstrating use if timeMode trim](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/15.json)
  * [One audio file, segments of file associated with different canvases](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/16.json)
  * [Use of Ranges with fragments of Audio tracks](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/17.json)
  * [Single canvas with explicit gaps between audio tracks](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/18.json)
  * [Canvas with sub-canvas of audio with lyrics annotations](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/20.json)
  * [All Beatles Albums](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/21.json)
  * [auto-advance viewingHint](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/22.json)
  * [Score images with audio (or video) performance annotated per region](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/24.json)
  * [Associate Audio and thumbnail with temporal-only Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/29.json)
  * [Associate Audio and thumbnail with temporal only Canvas](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/30.json)


### A/V with Authentication

The [IIIF Authentication API](http://iiif.io/api/auth/) may be used to orchestrate authentication and access control for audio and video resources using the same approach as image resources exposed through the [IIIF Image API](http://iiif.io/api/image/). Information about authentication services is expressed in a services description document analogous to the Image API `info.json`. A services document is required even in the case of static ("level 0") audio and video resources, where no transformation services are provided, because the document is used a probe to determine authentication and authorization status.

The following JSON-LD snippet shows how a `service` description `http://example.org/for-foo/services.json` is associated with a video resource `http://example.org/foo.mp4` in a `painting` Annotation:

```javascript
{
  "id": "http://example.org/iiif/audio/1",
  "type": "Annotation",
  "motivation": "painting",
  "body" : {  // or item in Choice
    "id": "http://example.org/foo.mp4",
    "type": "Video",
    "format": "video/mpeg4;codecs='...'",
    "service": {
      "id": "http://example.org/for-foo/services.json",
      "profile": "http://iiif.io/api/svc/0/profile.json",
      "description": "Auth services live in this external service description JSON"
    }
  }
  "target": "http://example.org/iiif/canvas/1"
}
```

A complete Canvas description with the an authentication service link is shown in [example 14](https://github.com/IIIF/iiif-av/blob/master/source/api/av/examples/14.json).

The format of the service description document is shown below.


### Bitstream APIs

A number of [use cases (labeled AV-bitstream-API)](https://github.com/IIIF/iiif-av/issues?q=is%3Aissue+is%3Aopen+label%3AAV-bitstream-API) require the generation of derivative video, audio and image resources from source video or audio resources. We imagine that each of these APIs will follow the style of the [Image API][http://iiif.io/api/image/], being a resource-oriented, ordered set of parameters, expressed as path fragments. The base URIs of these APIs will be described, along with any authentication information for the AV resource, in a services description document.

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
    },
    {
       // ... auth in here as above...
    }
  ]
}
```

The following sections describe the APIs for Audio and Video that might be described in the services document.

#### Video Bitstream API

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
  * Video zoom [#25](https://github.com/IIIF/iiif-av/issues/25)
  * Segmented for download [#7](https://github.com/IIIF/iiif-av/issues/7)
  * Segmentation by time [#4](https://github.com/IIIF/iiif-av/issues/4)
  * Request a snippet be created for later download  [#47](https://github.com/IIIF/iiif-av/issues/47)
  * Ability to describe a master file that's not available [#39](https://github.com/IIIF/iiif-av/issues/39)

##### Video Bitstream API info.json format

```javascript
{
  "id": "http://example.org/prefix/video-api-id",
  "profile": "http://iiif.io/api/av/video/0/level0.json",
  "width": 800,
  "height": 600,
  "duration": 369.2,
  ...
}
```

#### Audio Bitstream API

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

  * Keep rights from source in derivative [#35](https://github.com/IIIF/iiif-av/issues/35)
  * Segmentation by time [#4](https://github.com/IIIF/iiif-av/issues/4)
  * Ability to describe a master file that's not available [#39](https://github.com/IIIF/iiif-av/issues/39)

##### Audio Bitstream API info.json format

```javascript
{
  "id": "http://example.org/prefix/audio-api-id",
  "profile": "http://iiif.io/api/av/audio/0/level0.json",
  "duration": 25.3,
  ...
}
```

#### Video to Image API

Described in use case [#5](https://github.com/IIIF/iiif-av/issues/5). Expectation is that this interface would be able to present a Image API at a given timepoint in the video. Possible URI patterns, where `v2i-api-id` is specified in the services description document:

```
{v2i-api-id}/info.json

{v2i-api-id}/{timePoint}/info.json -- An Image API Image Information document

{v2i-api-id}/{timePoint}/{region}/{size}/{rotation}/{quality}.{fmt} -- Image API request at timePoint
```

  * `timePoint` is specified as the floating point number of seconds after the beginning of the video stream.
  * The remaining parameters are interpreted exactly as the Image API, and a Image API `info.json` should be provided at `{v2i-api-id}/timePoint/info.json`

##### Video to Image info.json format

```javascript
{
  "id": "http://example.org/prefix/v2i-api-id",
  "profile": "http://iiif.io/api/av/v2i/0/level0.json",
  "duration": 25.3,
  "timePoints": [ 0, 5.01, 10.04, 15.12, 20.18 ]
}
```

  * level0 - only the `timePoints` specified (cf. `sizes` in Image API)
  * level1 - can also give other `timePoints`, and the given ones are preferred

#### Video to Audio transformation

We assume that video to audio transformation can be subsumed in an Audio API service associated with a Video rather than creation of a Video to Audio API that then exposes an Audio API service. Use case: [#30](https://github.com/IIIF/iiif-av/issues/30).
