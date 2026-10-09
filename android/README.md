# WordCraft for Android

The Gradle project that packages `apps/wordcraft-android` (the Rust app as a `GameActivity`
shell) into an APK/AAB. CI (`.github/workflows/android.yml`) builds it on every push to the
`android` branch and attaches signed builds to a GitHub Release on `android-v*` tags.

## How it fits together

- `apps/wordcraft-android/src/lib.rs`: `android_main`, the platform services (open, save,
  preferences, links) and the JNI bridge to `MainActivity`. It mirrors the web shell
  (`apps/wordcraft-web`): picked files arrive as bytes through `Services::inbox`, saves and
  exports go out through `Services::download`.
- `app/src/main/java/.../MainActivity.kt`: the Storage Access Framework picker, saving into
  `Downloads/WordCraft/`, opening links, and the full-screen window.
- `cargo ndk` drops `libwordcraft_android.so` into `app/src/main/jniLibs/arm64-v8a/` (ignored by
  git); Gradle packages it.

## Building locally

Needs the Android SDK (platform 35, build-tools 35), NDK r27, a stable Rust toolchain with the
`aarch64-linux-android` target, and `cargo-ndk`:

```sh
rustup target add aarch64-linux-android
cargo install cargo-ndk
export ANDROID_NDK_HOME=$ANDROID_SDK_ROOT/ndk/<version>
cargo ndk -t arm64-v8a --platform 30 -o android/app/src/main/jniLibs build --release -p wordcraft-android
cd android && ./gradlew assembleDebug
```

Set `CRAFT_FONTS_DIR` to a checkout of storytold/craft-fonts to embed the shared fonts, as the
release builds do.

## Keyboard

The document canvas asks for the soft keyboard when it has focus; a Bluetooth or USB keyboard
works like on the desktop (Ctrl for the shortcuts in `crates/ui-egui/src/keys.rs`).

## Known limits (first version)

- Save, Save As and Export (PDF/PNG) write to `Downloads/WordCraft/<name>` without a dialog;
  saving the same name again in one session overwrites it. There is no Save back into the
  opened file (SAF write-back), and AutoSave is off because documents have no path.
- Open Recent is empty (picked files are names, not reopenable paths); no file drag and drop;
  no TCP control server; Read Aloud is not available.
- The Word-style ribbon needs a tablet-sized screen; on a phone's cover screen it is cramped.
