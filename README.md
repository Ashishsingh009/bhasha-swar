# Bhasha-Swar

**Bhasha-Swar (भाषा-स्वर)** is an offline voice and visual agricultural advisor for Android. A farmer or a Krishi Vigyan Kendra (KVK) field worker speaks, photographs a plant, or does both. The phone matches that input to a local advice catalog and shows the result with no network.

Package: `com.bhasha.swar`.

## Who it is for

- Farmers in rural areas where connectivity is absent or unreliable.
- KVK extension workers diagnosing crop problems in the field with those farmers.

The interface is built for outdoor use: high contrast, large type, large touch targets, and voice plus camera as the primary inputs.

## How a diagnosis works

Speech, photos, the embedding model, and the advice catalog stay on the device. A diagnosis never depends on a server.

**EmbeddingGemma 2** runs on the phone and maps text, images, and audio into one 768-dimensional space. It is an embedding model, so the advice text comes from a local catalog. The phone embeds the voice and the photo, then retrieves the closest catalog entries (symptoms, diseases, practices) and shows those entries. On Android the model runs through Google AI Edge (LiteRT-LM or the MediaPipe Universal Embedder).

| Footprint | Modalities | Parameters |
| --- | --- | --- |
| Smaller | Text and image | ~440M |
| Spoken input | Text and audio | ~570M |
| Full | Text, image, and audio | ~740M |

Quantized full-multimodal weights need on the order of a few hundred megabytes of RAM. Which footprint ships first is still open.

## Languages

Hindi is the first language for the voice path. The catalog and interface are intended to cover:

- Hindi
- Bengali
- Marathi
- Telugu
- Tamil
- Gujarati
- Kannada
- Malayalam
- Odia

## What is in the repo today

The app is the Jetpack Compose template. It is not yet an advisor.

- Adaptive navigation (`NavigationSuiteScaffold`) with **Home**, **Favorites**, and **Profile**.
- A greeting screen (`Hello Android!`).
- Material 3 theme with dynamic color still enabled.
- Camera, speech, EmbeddingGemma 2, and the crop catalog are not built yet.

## Tech stack

Versions are the set pinned in `gradle/libs.versions.toml` as of October 2026.

| Piece | Version |
| --- | --- |
| Language | Kotlin 2.2.10 |
| UI | Jetpack Compose, Compose BOM `2026.09.00` |
| Design | Material 3, including window size class and the adaptive navigation suite |
| Android Gradle Plugin | 9.4.1 |
| Gradle | 9.6.0 |
| Gradle JVM | 25 |
| App bytecode | Java 11 |
| `minSdk` | 31 |
| `targetSdk` / `compileSdk` | 37 |
| `applicationId` | `com.bhasha.swar` |
| `versionName` | 1.0 |

Screens are Compose. Shared UI uses Material 3: `Scaffold`, `NavigationSuiteScaffold`, buttons, cards, lists, text, top app bars, and icons, plus window size class so the same UI can move from a phone in the field to a larger KVK device.

## Project layout

```
app/src/main/java/com/bhasha/swar/
├── MainActivity.kt          # activity, navigation destinations, greeting
└── ui/theme/                # color, type, BhashaswarTheme
app/src/main/AndroidManifest.xml
app/build.gradle.kts
gradle/libs.versions.toml    # dependency versions
```

New screens stay under `com.bhasha.swar` as `@Composable` functions. Dependencies are declared in `gradle/libs.versions.toml` and referenced from `app/build.gradle.kts`.

## Requirements

- Android Studio with Android SDK 37.
- A device or emulator on API 31 or newer.
- JDK 25 for the Gradle daemon (the wrapper resolves it through the Foojay toolchain).

## Build and run

From the repository root:

```bash
./gradlew :app:assembleDebug
```

Open the project in Android Studio and run the `app` configuration. The debug APK is written under `app/build/outputs/apk/debug/`.

Release builds enable R8 optimization for the `androidx`, `kotlin`, and `kotlinx` packages.

## Tests

```bash
./gradlew :app:test
./gradlew :app:connectedAndroidTest
```

`test` runs host unit tests. `connectedAndroidTest` needs a connected device or emulator and checks that the package name is `com.bhasha.swar`.

A diagnosis change is accepted only when it still works with the network disabled: advice text comes from the local catalog, a weak match offers a retry, and photos, audio, embeddings, and catalog rows remain in app storage after the process is killed.

## Build order

Each step lands only after the one before it can run on a device with no network.

1. **Compose shell.** Replace the greeting with a home action (speak or photograph) and a place to show the last result. Fix a high-contrast light theme and turn dynamic color off. Favorites and Profile stay as shells.
2. **Local catalog.** Store crop problems, visible symptoms, and advice text on the device, and expose lookup by id and by embedding similarity.
3. **Camera capture.** Photograph a plant with CameraX, preview it, and keep the image on the device.
4. **Voice capture.** Record on device, show the utterance back, and support Hindi first.
5. **On-device retrieval.** Load EmbeddingGemma 2, embed the photo, the utterance, or both, and return the top catalog matches with a similarity score.
6. **Advice screen.** Lead with the problem name, the confidence, and the action to take. Offer a second capture when the top match is weak.
7. **Field check.** A farmer and a KVK worker complete one diagnosis with the network disabled.

## Still open

- Which EmbeddingGemma 2 footprint ships first: text and image, or text, image, and audio.
- Source of the local crop catalog, and which crops and problems are in the first version.
- Whether catalog text is shown as stored, or a later on-device model speaks a shorter answer from the retrieved entry.
- Languages beyond Hindi for the voice path.

## License

Apache License 2.0. See [LICENSE](LICENSE).
