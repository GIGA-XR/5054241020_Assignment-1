# Individual Assignment 1: Extended Reality Application Teardown

| **Name** | Fazle Mawla Wahyuhanda |
| --- | --- |
| **NRP** | 5054241020 |
| **Class** | Rekayasa Kecerdasan Artifisial (N) |
| **Batch** | 2024 |

## Subject

| **Item** | **Finding** |
| --- | --- |
| Application | Spatial Lingo: Language Practice |
| Release | 22 January 2026 |
| Reference device | Meta Quest 3 |
| Class | Standalone 6DoF, video-see-through mixed reality HMD |
| Intended task | Speak vocabulary about room objects |
| AI components | YOLOv9/Sentis on device; Llama 4 Maverick through Llama API; Voice SDK through Wit.ai |

> “Jan 22, 2026” — [Meta announcement](https://developers.meta.com/horizon/blog/spatial-lingo-open-source-app-ai-assisted-language-practice/).

**Thesis.** Spatial Lingo makes speech meaningful and turns the headset into a room-aware vocabulary tutor. This fits pronunciation practice, but cloud stages add network, data-transfer, and recognition dependencies.

**Method note.** No Spatial Lingo or VR headset used. Source-and-code teardown; runtime claims come from linked docs/repository, not personal use.

## Figures and evidence status

![Spatial Lingo developer artwork](assets/SpatialLingo.png)

*Figure 1. Developer artwork; contextual evidence, not hands-on evidence.*

![Passthrough scene](assets/gif_pass.gif)

*Figure 2. Developer passthrough capture; not an author session; date not visible.*

![Generated word cloud](assets/gif_wc.gif)

*Figure 3. Developer word-cloud capture; not an author session; date not visible.*

![Voice transcription](assets/VoiceTranscription.gif)

*Figure 4. Developer voice-transcription capture; not an author session; date not visible.*

**Submission gate:** no personal session/measurement. Add one dated author capture or measurement before submission; never claim developer art as your own.

## 1. Device class and continuum placement

Quest 3 is not only “VR.” Meta describes Quest as an all-in-one 6DoF mixed-reality device ([device comparison](https://developers.meta.com/horizon/essentials/compare-devices/)). Spatial Lingo runs as a standalone Android/OpenXR app: outward-facing cameras feed the display, which composites the room with word clouds and Golly Gosh. This is **video see-through mixed reality**, between the physical and virtual ends of the reality–virtuality continuum.

> “Meta Quest devices are wireless all-in-one mixed reality devices delivering the freedom of 6 degrees of freedom” — [Meta device comparison](https://developers.meta.com/horizon/essentials/compare-devices/).

> “On Quest 3 and Quest 3S, passthrough is full color with depth estimation” — [Meta passthrough documentation](https://developers.meta.com/horizon/essentials/horizon-os-passthrough/).

That class forces registration to the physical room, depth-aware placement, and tolerance for camera latency. A word cloud floating through a table breaks the illusion. The view is also mediated, not equivalent to eyesight: Meta warns that HMDs have limited field of view and represent depth and colour less accurately than natural vision ([passthrough safety guidance](https://developers.meta.com/horizon/design/mr-health-passthrough/)). Spatial Lingo gains context but inherits occlusion, lighting, and comfort constraints.

> “HMDs have limited field of view (FoV) and are not as accurate at representing important visual cues like depth and color.” — [Meta passthrough safety guidance](https://developers.meta.com/horizon/design/mr-health-passthrough/).

## 2. Input modality: speech is the core, hands are support

The application combines speech, hand/controller selection, head pose, and environmental sensing. I would keep the combination but rank the channels. **Speech is the core modality**: saying “chair” tests pronunciation and sentence use; pointing tests only selection. A keyboard or controller-only design would turn an oral exercise into visual spelling.

Hands are least disruptive for selecting or squeezing a word cloud. Controllers provide a fallback when tracking loses a fingertip. The sample supports both and implements squeeze from tracked finger bones ([sample repository](https://github.com/oculus-samples/Unity-SpatialLingo)). Looking supplies candidates; walking toward a word cloud activates its lesson.

> “This experience supports both hand tracking and controllers.” — [Spatial Lingo source repository](https://github.com/oculus-samples/Unity-SpatialLingo).

Speech can fail on accents, overlapping talk, or background noise; a wrong transcript can make Llama mark a correct answer wrong, and the network adds waiting time. My replacement is **local speech recognition with a visible transcript plus a text/gesture fallback**. This is a recommendation, not a documented feature. The sample names on-device recognition as an extension “to reduce latency and network dependency” ([technical documentation](https://developers.meta.com/horizon/documentation/unity/unity-sample-spatial-lingo/)).

## 3. AI across the pipeline

| Stage | What runs | Location and cost |
| --- | --- | --- |
| Perception | Passthrough → YOLOv9 → 2D boxes; Environment Depth maps boxes into 3D | YOLO/Sentis runs on Quest. It uses GPU, memory, battery, and camera bandwidth, but need not upload every frame. |
| Content and evaluation | Crop + class → Llama 4 Maverick; word cloud, translation, feedback, dialogue | Hosted Llama API adds network latency, image/token transfer, usage cost, and a cloud data boundary. |
| Speech | Audio → Voice SDK/Wit.ai transcription; text → Wit.ai TTS audio | External service requires internet and adds latency; TTS caching can reduce repeated requests. |
| Rendering/delivery | Unity renders passthrough, text, character, audio, and depth placement | No evidence of a separate generative model; AI outputs feed ordinary headset-side XR rendering. |

> “YOLO object detection runs on-device via Unity Sentis to classify objects into 80 COCO classes” — [Meta technical documentation](https://developers.meta.com/horizon/documentation/unity/unity-sample-spatial-lingo/).

> “That transcript is then sent to Llama 4 Maverick” — [Meta announcement](https://developers.meta.com/horizon/blog/spatial-lingo-open-source-app-ai-assisted-language-practice/).

> “Only if the end user turns on the mic activation will the developer app record the voice command and send it directly to Wit.ai” — [Voice SDK activation documentation](https://developers.meta.com/horizon/documentation/unity/voice-sdk-activation/).

The split is sensible: a small detector runs continuously on Quest; the larger model is called after selection. Camera capture adds documented latency, GPU, and memory overhead ([camera performance](https://developers.meta.com/horizon/documentation/spatial-sdk/spatial-sdk-pca-overview/)). Llama exposes prompt/completion-token counts, so longer prompts and retries increase usage ([Llama API package documentation](https://github.com/oculus-samples/Unity-SpatialLingo/blob/main/Packages/com.meta.utilities.llamaapi/README.md)). I cannot state a per-lesson dollar cost without a bill or request log.

> “Image capture latency: 20-40ms” and “GPU overhead: ~1-2% per streamed camera” — [Passthrough Camera API](https://developers.meta.com/horizon/documentation/spatial-sdk/spatial-sdk-pca-overview/).

> “num_prompt_tokens” and “num_completion_tokens” — [Llama API package documentation](https://github.com/oculus-samples/Unity-SpatialLingo/blob/main/Packages/com.meta.utilities.llamaapi/README.md).

The pipeline also has a security cost. A cropped camera image goes to Llama, and Meta classifies camera images as Device User Data. Production should proxy the key through a backend, rate-limit calls, and minimise or blur content. The upstream package warns that embedding a Llama key in a shipped Quest binary can cause unauthorised usage and unexpected charges ([Llama API security guidance](https://github.com/oculus-samples/Unity-SpatialLingo/blob/main/Packages/com.meta.utilities.llamaapi/README.md)).

> “Once an object is identified with YOLO, the image is cropped and sent to Llama API” — [Meta announcement](https://developers.meta.com/horizon/blog/spatial-lingo-open-source-app-ai-assisted-language-practice/).

> “camera image data is considered Device User Data” — [Passthrough Camera API](https://developers.meta.com/horizon/documentation/spatial-sdk/spatial-sdk-pca-overview/).

> “Quest apps that are shipped to end users should not directly embed Llama API keys” — [Llama API security guidance](https://github.com/oculus-samples/Unity-SpatialLingo/blob/main/Packages/com.meta.utilities.llamaapi/README.md).

## 4. Impact

The intended benefit is concrete: a learner sees a real object, receives attached vocabulary, says the word, and gets feedback. That context may connect to daily life better than a detached list. It remains a design hypothesis, not a measured learning outcome; Meta reports no controlled retention study.

Privacy risk follows directly from the sensors: the camera sees rooms, documents, and bystanders; the microphone captures speech; depth scanning reveals layout. Raw camera access requires permission ([Passthrough Camera API](https://developers.meta.com/horizon/documentation/spatial-sdk/spatial-sdk-pca-overview/)). Users need a capture indicator, lesson-scoped mic permission, and notice that selected crops and voice data leave the headset. Never ship a reusable Llama key in the client.

> “Either permission `android.permission.CAMERA` or `horizonos.permission.HEADSET_CAMERA` is required.” — [Passthrough Camera API](https://developers.meta.com/horizon/documentation/spatial-sdk/spatial-sdk-pca-overview/).

The human-factors issue is accessibility. Speaking fits the pedagogy but excludes users with speech disabilities and users whose accents or noisy rooms reduce recognition accuracy; hand/controller selection does not replace it. Subtitles, adjustable speech rate/volume, text answers, and a replayable transcript would widen access. Meta also warns about long passthrough exposure ([passthrough safety guidance](https://developers.meta.com/horizon/design/mr-health-passthrough/)). Sessions need pauses and should not require tiny labels while moving.

> “Long periods of exposure to full passthrough may result in visual discomfort, motion sickness, disorientation, or negative after-effects.” — [Meta passthrough safety guidance](https://developers.meta.com/horizon/design/mr-health-passthrough/).

## Conclusion

Spatial Lingo is a 2026 standalone Quest mixed-reality application, not generic VR. Its strongest idea is linking a real object to an oral lesson; its weakest dependency is the cloud chain. I would keep speech, add local ASR and a non-speech fallback, and treat learning benefits as unverified until measured.

## AI-assistance disclosure

**What I used and for what.** I used an AI assistant to compare the draft with the rubric, remove unsupported claims, fit 900–1500 words, and suggest examiner questions. I checked primary sources and retained quoted evidence. I did not use AI to invent a measurement, study, price, or figure.

**What I disagreed with my AI assistant about.** It initially treated voice processing as on-device and said the screenshots met the dated-figure rule. I rejected both: Meta says audio is sent to Wit.ai, and no current image visibly contains a date. The report labels speech external and leaves the author-figure requirement as a pre-submission gate.

## References

1. Meta Horizon OS Developers. [Spatial Lingo announcement](https://developers.meta.com/horizon/blog/spatial-lingo-open-source-app-ai-assisted-language-practice/), 22 January 2026.
2. Meta Horizon OS Developers. [Spatial Lingo technical documentation](https://developers.meta.com/horizon/documentation/unity/unity-sample-spatial-lingo/), updated 11 May 2026.
3. Meta Horizon OS Developers. [Meta Quest device comparison](https://developers.meta.com/horizon/essentials/compare-devices/), updated 3 September 2026.
4. Oculus Samples. [Unity-SpatialLingo source repository](https://github.com/oculus-samples/Unity-SpatialLingo).
5. Meta Horizon OS Developers. [Passthrough Camera API overview](https://developers.meta.com/horizon/documentation/spatial-sdk/spatial-sdk-pca-overview/), updated 21 April 2026.
6. Meta Horizon OS Developers. [Voice SDK activation](https://developers.meta.com/horizon/documentation/unity/voice-sdk-activation/), updated 2026.
7. Meta/Oculus Samples. [Llama API package documentation and security guidance](https://github.com/oculus-samples/Unity-SpatialLingo/blob/main/Packages/com.meta.utilities.llamaapi/README.md).
8. Meta Horizon OS Developers. [Passthrough safety guidance](https://developers.meta.com/horizon/design/mr-health-passthrough/), updated 25 July 2025.
