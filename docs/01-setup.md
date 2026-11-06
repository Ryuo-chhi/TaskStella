# 01 Setup

## Requirements

| Tool | Notes |
|---|---|
| Flutter SDK (stable channel) | Both members use the same version. Record it below once installed. |
| Android Studio | For the Android SDK and an emulator. |
| VS Code or Android Studio | With the Flutter and Dart extensions. |
| Git | Every member commits from their own account. |

Agreed Flutter version: `_____` (fill in after running `flutter --version`).

## First run

1. Install Flutter and add it to your PATH.
2. Check the setup:

```bash
flutter doctor
```

3. Fix every item `flutter doctor` reports, then create or clone the project.
4. Fetch packages:

```bash
flutter pub get
```

5. Start an emulator (a 360 dp wide device and a 390 dp wide device, see [05-testing.md](05-testing.md#manual-ui-checks)) and run:

```bash
flutter run
```

## Font: Nunito

The design uses Nunito everywhere. In Figma it is built in. The app needs the font files bundled, because the app must work offline (no runtime font downloads).

1. Download Nunito from Google Fonts (it is free under the SIL Open Font License).
2. Copy only the weights the design uses into `assets/fonts/`: Regular (400), SemiBold (600), Bold (700).
3. Register the family and each weight under the `fonts:` section of `pubspec.yaml`.
4. Set Nunito as the app font family in the theme (see [07-design-tokens.md](07-design-tokens.md)).

Do not use a package that downloads fonts at runtime. It breaks the offline rule.

## Everyday commands

Run the tests:

```bash
flutter test
```

Check for analyzer warnings (must be zero before a commit):

```bash
flutter analyze
```

Format the code:

```bash
dart format .
```
