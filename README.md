# Köməkçi: voice assistant "Ali" (MVP)

A voice assistant in Azerbaijani for elderly people. With voice, Ali sends WhatsApp messages, makes calls, reads medicine boxes with the camera, and warns about scam SMS messages.

Plan, architecture and demo script: [MVP_PLAN.md](MVP_PLAN.md).

```
NVATON/
├── backend/   Node.js + TypeScript + Express. Gemini (or Claude) and Azure keys are stored here
└── android/   Kotlin + Jetpack Compose app
```

## Requirements

- Node.js 20.12+ (for the backend)
- Android Studio (or JDK 17/21 + Android SDK 36)
- An Android 8.0+ phone. WhatsApp must be installed on the phone for automatic "Send"
- A [Gemini API key](https://aistudio.google.com/apikey) (Google AI Studio, has a free plan). Alternative: a [Claude API key](https://platform.claude.com/)
- Optional: an [Azure Speech](https://azure.microsoft.com/products/ai-services/speech-services) key (the free F0 plan is enough). This provides Ali's natural Azerbaijani voice, and it can also make speech recognition more accurate than Gemini

## 1. Starting the backend

```bash
cd backend
npm install
cp .env.example .env
```

In the `.env` file, put your key after `GEMINI_API_KEY=`, then run:

```bash
npm run dev
```

`npm run dev` restarts the server automatically when code changes. After changing `.env`, restart the server manually: press **Ctrl+C** in the terminal, then run `npm run dev` again.

Keep this terminal open, because the server stays running there. Run the next commands in a **separate terminal tab** (the **+** button in the terminal panel of VS Code). Commands typed into the running server's window are not executed.

You should see `AI (gemini): ready` and `Speech recognition: Gemini`. If you add an Azure key, speech recognition becomes `Azure (az-AZ)` and `Azure TTS: ready` also appears. The server also runs without any keys: the app understands simple commands (messages, calls, opening the camera, scam rules) on the phone itself, and only the parts that need AI do not work.

| Variable | Purpose |
|---|---|
| `GEMINI_API_KEY` | Gemini: complex commands, reading images, scam checks, and Azerbaijani speech recognition |
| `GEMINI_MODEL`, `GEMINI_VISION_MODEL` | By default, `gemini-3.5-flash-lite` for commands, scam checks and speech (measured: ~1–3 s), and `gemini-3.6-flash` for the camera (~6–9 s). `gemini-3.8-flash` took 75 seconds on the free plan, which is too slow for a voice assistant. `-latest` names such as `gemini-flash-latest` do not exist in the `v1` API this SDK uses |
| `ANTHROPIC_API_KEY`, `AI_PROVIDER=claude` | To use Claude instead of Gemini (default model `claude-opus-5-5`) |
| `AZURE_SPEECH_KEY`, `AZURE_SPEECH_REGION` | Ali's natural Azerbaijani voice (`az-AZ-BanuNeural`). When set, speech recognition also uses Azure's `az-AZ` model instead of Gemini. If left empty, the phone's own voice is used |
| `HOST`, `APP_TOKEN` | For connecting the phone over Wi-Fi instead of USB (see below) |

> **Important note on speech recognition.** Google's built-in phone speech recognizer has no Azerbaijani model (we checked on the emulator: `does not support locale: az-AZ`). That is why, when a Gemini or Azure key is set on the server, the app records speech, sends it to the server, and recognizes it there in Azerbaijani. If there is no server key, the phone's recognizer is tried. If that fails too, the "Type a command" field always works.

## 2. Connecting the device and installing the app

**a) Add `adb` and `emulator` to your PATH** (macOS, once). Otherwise you get the error `zsh: command not found: adb`:

```bash
echo 'export PATH="$HOME/Library/Android/sdk/platform-tools:$HOME/Library/Android/sdk/emulator:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

**b) Connect a device.** Without a device, the installation stops with `No connected devices!`.

- **Real phone** (needed for the demo, because WhatsApp and the microphone work there): Settings → About phone → tap "Build number" 7 times → Developer options → enable **USB debugging**. Connect the phone with USB and tap **Allow** when the phone asks "Allow USB debugging".
- **Emulator:** this script starts the emulator using your Mac's camera (as the phone camera) and your Mac's microphone:

```bash
scripts/emulator.sh
```

macOS may ask for camera and microphone permission the first time. Tap **Allow**. If you start the emulator from Android Studio → Device Manager → ▶, the Mac microphone is turned off. Turn it on from the emulator's ⋯ → Microphone → "Virtual microphone uses host audio input" switch. Check that the device is visible:

```bash
adb devices
```

One line should show `device`.

**c) Download the offline wake-word model** ("Ali") once, after cloning the project (~37 MB). Without the model, the app still works, but it cannot start by itself when it hears the name:

```bash
scripts/get-wake-model.sh
```

**d) Install the app.** In Android Studio: File → Open → the `android/` folder → ▶ Run. Or from the terminal:

```bash
cd /Users/farid/NVATON/android && JAVA_HOME=$(/usr/libexec/java_home -v 21) ./gradlew installDebug
```

## 3. Connecting the phone to the backend

If the device is connected with USB (this also works with the emulator):

```bash
adb reverse tcp:8787 tcp:8787
```

Run this command again every time you reconnect the device, restart the emulator, or run `adb kill-server`.

If both a phone and an emulator are connected, plain `adb reverse` gives a "more than one device" error. Choose the device: `-d` for the USB phone, `-e` for the emulator:

```bash
adb -d reverse tcp:8787 tcp:8787
```

By default, the app connects to `http://127.0.0.1:8787`. This command forwards that address to the server on your laptop.

**Connecting over Wi-Fi:** in `.env`, set `HOST=0.0.0.0` and `APP_TOKEN=secret-word`, then restart the server. In the app, go to ⚙️ → Server and enter `http://<your-laptop-IP>:8787` and the same token. Without a token, anyone on the same Wi-Fi can use your Gemini key.

## 4. Phone setup (once)

In the app, go to ⚙️ → **Setup** and tap **Open** next to each ❌ line:

1. **Microphone, camera, calls, notifications:** allow them.
2. **Scam protection:** in the "Notification access" list, enable **Köməkçi: scam protection**.
3. **Automatic "Send" in WhatsApp and screen reading:** Accessibility → enable **Köməkçi**. The same permission is used for both tasks.

> **If the setting is greyed out (Android 13+):** Settings → Apps → Köməkçi → ⋮ in the top right → **Allow restricted settings**. Then repeat steps 2 and 3.
>
> After reinstalling the app, Android may turn off the Accessibility service. Check ⚙️ → Setup after every installation.

> **Xiaomi / Redmi (HyperOS, MIUI):** for USB debugging, go to About phone → tap **HyperOS version** 7 times, then Additional settings → Developer options → **USB debugging**. When you plug in the cable, set the USB mode to **File transfer**. So that Ali can hear you in the background, go to Apps → Köməkçi → turn on **Autostart**, and set **Battery saver** to "No restrictions".

4. **Family contacts:** the demo contact numbers are fake (`+994 00 ...`). Delete them and enter your team members' real numbers. In the "How it's called" field, enter words such as `nəvəm` (my grandson) or `qızım` (my daughter).

## 5. Demo commands

You can say **"Əli"** without pressing a button (see the section below). The button also works.

| Say | What happens |
|---|---|
| *"Əli"* … *"nəvəmə yaz: salam"* (*"Ali… text my grandson: hello"*) | *"Bəli?"* ("Yes?") → *"Kimə yazım: Murada, yoxsa Aysuya?"* ("Who should I write to: Murad or Aysu?") → *"Murada"* → *"Murada «Salam» yazım?"* ("Send «Hello» to Murad?") → *"Hə"* ("Yes") → sent on WhatsApp |
| *"Əli, nəvəmə yaz: salam"* (all in one breath) | Skips "Bəli?" and goes straight to *"Kimə yazım…"* |
| *"Qızıma de ki axşam gəlsin"* (*"Tell my daughter to come in the evening"*) | *"Leylaya «Axşam gəlsin» yazım?"* ("Send «Come in the evening» to Leyla?") |
| *"Əli, qızıma WhatsApp-dan salam yaz və göndər"* (*"Ali, text my daughter hello on WhatsApp and send it"*) | *"Leylaya «Salam» yazım?"*: the words "və göndər" and "WhatsApp-dan" are not included in the message |
| *"Oğluma zəng et"* (*"Call my son"*) | Makes the call |
| *"Əli, WhatsApp-dan Kamrana yaz ki sabah görüşək"* (*"Ali, text Kamran on WhatsApp that we'll meet tomorrow"*) | Writes to any contact not in the family list: first it looks in the phone's contacts, then searches WhatsApp. *"Ailə qrupuna yaz: axşam gəlin"* ("Write to the family group: come in the evening") works for groups too. It asks for confirmation before sending |
| *"Əli, Rauf adlı yeni kontakt saxla, nömrəsi sıfır əlli, yüz iyirmi üç, qırx beş, altmış yeddi"* (*"Ali, save a new contact named Rauf, number 050 123 45 67"*) | Reads the number back (*"Rauf, 050 123 45 67. Yadda saxlayım?"* — "Shall I save it?"), then saves it after "hə" (yes). If the phone integration is on, it also saves to the phone's contacts. If the name or number is missing, it asks for it |
| The number already exists under another name | *"Bu nömrə artıq «Murad» adı ilə var. Köhnəni saxlayım, əvəz edim, yoxsa adları birləşdirim?"* ("This number is already saved as Murad. Keep the old one, replace it, or merge the names?"): **Keep old / Replace / Merge** |
| *"Əli, telefon kontaktlarımı əlavə et"* (*"Ali, add my phone contacts"*) | Opens the phone's contacts screen: search and select. If one number is saved under several names on the phone, it asks which name to keep; the other names stay as nicknames |
| *"Əli, Kamranla söhbəti aç"* (*"Ali, open the chat with Kamran"*) | Opens the WhatsApp chat. Then you can say *"Əli, ekranı oxu"* ("Ali, read the screen") to have it read. In the read chat, *"ona yaz ki gəlirəm"* ("tell him I'm coming") replies to that person |
| *"Əli, mesajlarımı oxu"*, *"kim yazıb?"* (*"Ali, read my messages"*, *"who wrote?"*) | Reads the latest incoming messages (who sent them, from which app, and when). Follow-up questions such as *"nə vaxt gələcək?"* ("when will they come?") are answered from those messages |
| *"Əli, sabah səhər 7-də oyat"*, *"hər gün axşam 9-da dərmanımı xatırlat"*, *"10 dəqiqədən sonra zəng çal"* (*"Ali, wake me at 7 tomorrow morning"*, *"remind me to take my medicine at 9 every evening"*, *"ring in 10 minutes"*) | Sets an alarm. When the time comes, it appears on the screen, says what it is for, and rings: **Turn off** / **Snooze 5 min** |
| *"Zəngli saatlarım hansılardır?"*, *"axşamkı xatırlatmanı sil"*, *"bütün zəngli saatları sil"* (*"What are my alarms?"*, *"delete the evening reminder"*, *"delete all alarms"*) | Reads the list, then deletes one or all of them. The list is also shown under ⚙️ → **⏰ Alarms** |
| *"Əli, dili ingilis dilinə dəyiş"*, *"switch to Azerbaijani"* | Changes the language by voice |
| In any app: *"Əli, ekranı oxu"*, *"Əli, ekranda kim yazıb?"* (*"Ali, read the screen"*, *"who wrote on the screen?"*) | Reads the screen and explains it: who wrote what, amounts, dates. Translates Russian and English text, and warns you if it sees a request for a code or money |
| *"Bu dərman nə üçündür?"*, *"Bu nədir?"* (*"What is this medicine for?"*, *"What is this?"*) | Opens the camera, says *"Kameranı tutun, şəkil çəkirəm"* ("Hold the camera, I'm taking a photo"), takes a photo itself, and answers **that question** |
| On the camera screen: *"Bəs gündə neçə dəfə içim?"*, *"Son istifadə tarixi keçib?"* (*"And how many times a day should I take it?"*, *"Has it expired?"*) | Takes a new photo. It remembers earlier answers, so short follow-up questions are understood |
| *"Kameranı aç"* (*"Open the camera"*) | Opens the camera and waits for your question (**🎤 Ask** or **📷 Read**) |
| On the camera screen: *"Bağla"* (*"Close"*) | Returns to the home screen |
| *"Nə edə bilirsən?"* (*"What can you do?"*) | Lists what it can do |
| Any other question | The AI gives a short answer |

- For yes/no questions, large **✅ Yes / ❌ No** buttons also appear on the screen.
- In the camera: tap the screen to focus, and pinch with two fingers to zoom. If the phone has a flashlight, the **🔦 Light** button appears.
- **Incoming notifications** (WhatsApp, SMS, Telegram, Viber messages, missed calls, reminders) are read aloud as they arrive: *"New message, WhatsApp. Murad: …"*. Ali stays silent while it is speaking, while you are speaking, during a call, and in "Do not disturb" mode. You can turn this off with ⚙️ → "Read incoming messages and notifications aloud".
- **In a WhatsApp chat**, when you say *"Əli, ekranı oxu"*, each message is read with its sender, its time and your own messages separately. Then you can ask follow-up questions.
- In a noisy place, use the **⌨️ Type a command** field. Azerbaijani diacritics are not required: `neveme yaz salam` works too.

## Design

The app's look was built with the [taste skill](https://github.com/Leonxlnx/taste-skill). It uses the "Frost and slate" palette chosen for software (light background, dark blue-grey text, one turquoise accent), the Chivo font (Google Fonts, OFL, `assets/licenses/Chivo-OFL.txt`), and the same card, button pair and corner size throughout. The icons are Material Symbols Rounded (Apache 2.0) and are stored as path data in the code (`ui/AppIcons.kt`). Large text, high contrast and touch targets of at least 56 dp are kept for older users.

## Language: Azerbaijani / English

Under ⚙️ → **🌐 Dil / Language**, select **English**. Screen text, Ali's voice (the phone's English voice), speech recognition and the AI's answers all switch to English. Azerbaijani stays the default, and the choice is remembered.

| English | What happens |
|---|---|
| *"Ali"* … *"text my daughter that I'll be late"* | *"Send «I'll be late» to Leyla?"* → *"Yes"* → sent on WhatsApp |
| *"Ali, text my grandson hello"* | *"Who should I write to: Murad or Aysu?"* → *"the second one"* |
| *"Ali, what is this medicine?"* | The camera opens, takes a photo, and answers in English |
| *"Ali, read the screen"* | Reads the current screen, translating Azerbaijani or Russian text |
| *"Call my son"*, *"What can you do?"* | Makes the call, lists what it can do |

- Words such as `nəvəm`, `qızım` and `oğlum` in the contacts are also recognized in English: *my grandson*, *my daughter*, *my son*.
- The AI understands English messaging and call commands, so the server is required for them. "Yes/No", the camera, the screen and "help" work without a server.
- The same model recognizes the word "Əli" in English mode too, so say "Ali" in English.
- The Azure voice is Azerbaijani only. In English mode, the phone's own voice always speaks.

## Waking Ali by voice (like Siri)

- After the app is opened once, it listens for the word "Ali" in the background. The screen shows **Say "Əli"**.
- The phone's status bar shows a notification and a green microphone dot. Android requires this for any app that listens in the background.
- Even when the app is closed or in the background, saying "Əli" brings it to the screen, and it says *"Bəli?"* ("Yes?").
- The wake word is recognized **on the phone, without internet**, by a small offline model (Vosk). There is no Azerbaijani model, so a Turkish model is used, which hears "Əli" as "ali" / "eli". Audio is sent to the server only **after** the name is heard.
- The name must be at the start of the sentence. *"Onun əli ağrıyır"* ("His hand hurts") or *"Allah razı olsun"* ("God bless you") will not wake the app.
- To turn it off: ⚙️ → **Wake up by saying "Əli"** switch, or the **Turn off** button in the notification.
- If you change the assistant's name (⚙️ → Assistant name), it listens for the new name. The model must recognize the name: common names such as Ayla, Leyla and Xanım work.
- While Ali is speaking or listening to you, it does not listen for the name, so your own voice saying "Əli" does not wake it.

## Testing without speaking (emulator)

Put the command in double quotes, or the phone cuts it off at the first space:

```bash
adb shell "am start -n az.nvaton.komekci/.MainActivity --es command 'nəvəmə yaz salam'"
```

A camera question (opens the camera, takes a photo, and answers):

```bash
adb shell "am start -n az.nvaton.komekci/.MainActivity --es command 'bu nədir?'"
```

Pretend that "Əli" was heard:

```bash
adb shell am start -n az.nvaton.komekci/.MainActivity --ez simulate_wake true
```

Stop listening and speaking:

```bash
adb shell am start -n az.nvaton.komekci/.MainActivity --ez stop true
```

Test the wake word with an audio file (16 kHz mono WAV, as if the microphone heard it):

```bash
adb exec-in run-as az.nvaton.komekci sh -c 'cat > files/ali.wav' < ali.wav
```

```bash
adb shell am start -n az.nvaton.komekci/.MainActivity --es wake_test ali.wav
```

Test screen reading without a microphone: the app moves aside, reads the screen behind it, then continues as if "read the screen" had been said:

```bash
adb shell am start -n az.nvaton.komekci/.MainActivity --ez screen_test true
```

See what the model hears:

```bash
adb logcat -s WakeWord
```

Send a scam SMS (emulator only):

```bash
adb emu sms send 5551234 "Hörmətli müştəri, kartınız bloklanıb. Kodu bu nömrəyə göndərin"
```

On a real phone, use ⚙️ → **Check for scam warnings**.

## Tests

```bash
cd backend && npm test
```

```bash
cd android && ./gradlew testDebugUnitTest
```

## Troubleshooting

| Problem | Solution |
|---|---|
| `No connected devices!` | The phone is not connected, or the emulator is not running. See step 2b, and check with `adb devices` |
| `zsh: command not found: adb` | `adb` is not in PATH: see step 2a |
| `EADDRINUSE: address already in use :::8787` | The server is already running in another terminal. Stop one of them with Ctrl+C |
| "Could not connect to the server" | Is the server running? Did you run `adb reverse tcp:8787 tcp:8787`? ⚙️ → Server → "Save and check" |
| "The AI is not connected yet" | `GEMINI_API_KEY` is missing or wrong in `backend/.env` (check the server terminal) |
| `429` / `rate limit` in the server log | The Gemini free-plan limit is used up: wait a bit, or upgrade to a paid plan in Google AI Studio |
| Speech recognition does not understand Azerbaijani | Add a Gemini (or Azure) key to `backend/.env`. The app then switches to the server's speech recognition automatically (⚙️ → "Server speech recognition" must be on). Or type the command |
| Ali speaks with a Turkish accent | The phone has no Azerbaijani voice. Add an Azure key to the backend |
| WhatsApp opens, but the message is not sent by itself | Accessibility service is not enabled (section 4). The number must be in international format: `+994...` |
| "Əli" does not wake the app | ⚙️ → "Wake up by saying Əli" must show "✅ Waiting for your name". If it says "Model not included in the app", run `scripts/get-wake-model.sh`, then reinstall the app. On the emulator, the Mac microphone must be on (`scripts/emulator.sh`). Speak close to the phone, clearly, and pause briefly after the name. `adb logcat -s WakeWord` shows what the model hears |
| The alarm rings but the screen does not turn on, only a notification appears | ⚙️ → Setup → "⏰ Full-screen alarm" → Enable (Android 14+). On Xiaomi, make sure Apps → Köməkçi → **Autostart** is on |
| Phone contacts are not shown | ⚙️ → Family contacts → turn on "Phone contacts integration" and allow it. Then "Add from phone" |
| "Text Kamran" does not find the person | ⚙️ → Setup → "Contacts: write to anyone" → Enable. Contacts that are not in the family list and groups are found through WhatsApp search, which needs Accessibility enabled |
| Incoming messages are not read | ⚙️ → Setup → "🔔 Notifications" must be ✅, and "Read incoming messages… aloud" must be on. It does not read during "Do not disturb" mode |
| "Əli, read the screen" replies "Please open the screen you want to read" | The screen name is captured when you say the command. If you opened Köməkçi yourself first, the screen is Köməkçi itself. Open the app you need and say "Əli, read the screen" there |
| On the emulator, "Əli" stops hearing after a while | The emulator's Mac microphone cuts out after a minute or two: `adb logcat -s WakeWord` shows `peak 0`, or an unchanging `peak 23191` (a synthetic tone). Restart the emulator with `scripts/emulator.sh`. This does not happen on a real phone |
| On the emulator, the camera shows a room | The emulator was not started with `scripts/emulator.sh`. Or in Android Studio → Device Manager → ✏️ → Camera → Back: **Webcam0** |
| On the emulator, "Speech Recognition and Synthesis from Google keeps stopping" | Google's Turkish voice crashes on the emulator of an Apple-chip Mac. The app detects this itself and switches to Google's online voice within a few seconds: close the dialog. This problem does not happen on a real phone or with the Azure voice |
| "The AI is very busy right now" | Gemini is overloaded at that moment (503). The server automatically tries another Gemini model. If that also fails, ask again in a moment |

## Security

- API keys are only in `backend/.env`, not in the APK.
- The Accessibility service only presses the button in WhatsApp, and only sends a message that was confirmed by voice, within 20 seconds. If the text in the input box differs from the confirmed text, it presses nothing.
- Incoming notifications are only read aloud on the phone. Recent messages are kept in memory only, for up to 3 hours (never on disk), for the "read my messages" feature. When you ask a follow-up question (for example, "when will they come?"), those messages or the screen text are sent to the server to answer it.
- Phone contacts integration is only turned on after the user agrees (in Settings or the import screen), and can be turned off at any time.
- Phone contacts are searched only on the phone and are not sent to the server. The server only sees the name you said (for example, "Kamran").
- Alarms are stored on the phone. The server sees only the alarm times and labels, so it can understand the command.
- The language choice is sent to the server with each request as `language: "az" | "en"`. Nothing else changes.
- The screen is only read when "Ali" is said, and is kept in memory for 3 minutes. Password fields are not read. Text is sent to the server (Gemini) only when you say "read the screen".
- The scam check runs on the phone first. Only suspicious messages are sent to the server.
- When server speech recognition is working, audio is recorded only after the microphone button is pressed or "Ali" is said, until silence, and is sent to the server (Gemini or Azure) for recognition. It is not stored anywhere.
- While waiting for the word "Ali", audio is processed only inside the phone and is never sent anywhere.
- The camera image is taken only when you ask a question or press **Read**, and is sent to the server for the answer.
- On Gemini's free plan, Google may use requests to improve its products. Use a paid plan for real users' messages.
- When Claude is selected (`AI_PROVIDER=claude`), the server-side `fallbacks: "default"` option is on: if the safety filter rejects a request, the API automatically retries it on the recommended model.
