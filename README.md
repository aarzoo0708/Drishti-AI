# Drishti AI

Drishti AI is an accessible visual assistance app for Android, built with Expo and React Native. It helps users inspect a live camera view or a selected image with spoken and on-screen descriptions.

## Problem

People who are blind or have low vision can face barriers when identifying nearby objects, reading visible text, or understanding a scene. Drishti AI provides image-based descriptions and voice controls to support those tasks. AI results can be incomplete or incorrect and should not be used as navigation or road-crossing instructions.

## Features

- Camera preview, rear/front camera switching, capture, review, and retake.
- Image selection from the photo library.
- Gemini image analysis for scene description, objects, and visible text; read-text commands use Gemini Vision, not a separate OCR engine.
- YOLOv8n object detection through the FastAPI backend.
- Android speech recognition for the “Hey Drishti” and “Hi Drishti” activation phrases and voice commands.
- Spoken responses with repeat and stop controls.
- Accessible labels, hints, screen-reader announcements, and system text scaling.

## Voice workflow

Start Voice Assistance and grant microphone permission. While the app is in the foreground, Android's speech service listens for “Hey Drishti” or “Hi Drishti”. Drishti AI acknowledges, then recognizes a command. Camera and image commands capture or open an image and send it to Gemini; object-detection commands call the YOLO backend. Results are displayed and spoken. After a result, the app listens for another command, then returns to wake phrase listening after 15 seconds of silence. Stop Assistance ends the session.

Wake phrase recognition uses Android's SpeechRecognizer through `expo-speech-recognition`; there is no offline keyword model bundled in this repository. Recognition depends on the device speech provider, may require network access, and is not an always-on background service. See [VOICE_ASSISTANCE_ANDROID.md](VOICE_ASSISTANCE_ANDROID.md).

## Technology and architecture

- **Mobile app:** Expo SDK 58, React Native, JavaScript.
- **Camera and media:** `expo-camera`, `expo-image-picker`, `expo-image-manipulator`.
- **Speech:** `expo-speech-recognition` and `expo-speech`.
- **Image analysis:** Gemini API called by the mobile client.
- **Object detection:** FastAPI backend using Ultralytics YOLOv8n and Pillow.

The Expo app contains the dashboard, navigation, permissions, camera/gallery flows, voice session, and service clients under `src/services`. Gemini analysis is sent directly from the client. YOLO image requests are posted to the backend's `/detect` endpoint; `/health` reports backend/model status.

## Requirements

- Node.js and npm compatible with the installed Expo SDK.
- Android Studio/Android SDK and a device or emulator for native camera and speech features.
- Python 3.10 or newer for the YOLO backend.
- A Gemini API key for image analysis.

## Frontend setup

```powershell
npm install
Copy-Item .env.example .env
```

Set `EXPO_PUBLIC_GEMINI_API_KEY` and, when using YOLO, `EXPO_PUBLIC_YOLO_URL` in `.env`. Then start Expo:

```powershell
npx expo start
```

Expo Go supports development flows that do not require unavailable native modules. Voice recognition requires the Android development build described below.

## Backend setup

From the repository root, create and activate a Python virtual environment, then install the backend requirements:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
py -m pip install -r backend\requirements.txt
```

Start the API:

```powershell
cd backend
uvicorn main:app --host 0.0.0.0 --port 8000
```

The backend loads `yolov8n.pt` from its working directory. The repository includes this weights file. The app's default `http://10.0.2.2:8000` address is for Android Emulator. For a physical device, set `EXPO_PUBLIC_YOLO_URL` to the computer's reachable LAN address, such as `http://192.168.1.42:8000`, then restart Expo.

## Gemini configuration

Create a key in [Google AI Studio](https://aistudio.google.com/app/apikey) and set it in the project-root `.env`:

```dotenv
EXPO_PUBLIC_GEMINI_API_KEY=your_gemini_api_key_here
```

Expo embeds `EXPO_PUBLIC_` values in the app bundle. Treat this client-side setup as a development/prototype configuration; use a secured server-side proxy before distributing a production app. `.env` is ignored by Git; never commit it.

## Android development build

Install Android Studio and configure an Android device/emulator, then run from the project root:

```powershell
npm install
npx expo prebuild --platform android
npx expo run:android --device
npx expo start --dev-client
```

The Android application ID is `com.drishtiai.assistant`. Rebuild the development app after changing native configuration. Detailed microphone permission and voice-session steps are in [VOICE_ASSISTANCE_ANDROID.md](VOICE_ASSISTANCE_ANDROID.md).

## Environment variables

| Variable | Purpose | Example |
| --- | --- | --- |
| `EXPO_PUBLIC_GEMINI_API_KEY` | Gemini Vision API key for image analysis | `your_gemini_api_key_here` |
| `EXPO_PUBLIC_YOLO_URL` | Base URL of the FastAPI YOLO service | `http://10.0.2.2:8000` (Android Emulator) |

Use `.env.example` as a template. Do not place real credentials in that file.

## Known limitations

- Android speech recognition is provider-dependent, may need network access, runs only while the app is foregrounded, and is not guaranteed to recognize every activation phrase.
- Voice recognition requires a native Android development build; Expo Go does not include the speech-recognition module.
- Gemini analysis requires internet access, and the client-side API key is extractable from a built app.
- YOLO requires the separate backend to be running and reachable from the device.
- Image descriptions and detections may be wrong; they are not safety-critical navigation guidance.

## Future improvements

- Move Gemini requests behind an authenticated backend so the key is not distributed in the client.
- Evaluate a dedicated offline wake-word engine and device speech-recognition coverage.
- Improve reliability testing and user feedback for uncertain image results.
