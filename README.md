# Köməkçi: səsli köməkçi "Əli" (MVP)

Yaşlı insanlar üçün Azərbaycan dilində səsli köməkçi. Əli səslə WhatsApp mesajı göndərir, zəng edir, kamera ilə dərman qutusunu oxuyur və fırıldaq SMS-lərindən xəbərdar edir.

Plan, arxitektura və demo ssenarisi: [MVP_PLAN.md](MVP_PLAN.md).

```
NVATON/
├── backend/   Node.js + TypeScript + Express. Gemini (və ya Claude) və Azure açarları burada saxlanılır
└── android/   Kotlin + Jetpack Compose tətbiqi
```

## Tələblər

- Node.js 20.12+ (backend üçün)
- Android Studio (və ya JDK 17/21 + Android SDK 36)
- Android 8.0+ telefon. Avtomatik "Göndər" üçün telefonda WhatsApp olmalıdır
- [Gemini API açarı](https://aistudio.google.com/apikey) (Google AI Studio, pulsuz planı var). Alternativ: [Claude API açarı](https://platform.claude.com/)
- İstəyə bağlı: [Azure Speech](https://azure.microsoft.com/products/ai-services/speech-services) açarı (pulsuz F0 planı kifayətdir). Əlinin təbii Azərbaycan səsi bundan gəlir, nitq tanımanı da Gemini-dən daha dəqiq edə bilər

## 1. Backend-i işə salmaq

```bash
cd backend
npm install
cp .env.example .env
```

`.env` faylında `GEMINI_API_KEY=` sətrinə öz açarınızı yazın, sonra:

```bash
npm run dev
```

`npm run dev` kod dəyişəndə serveri özü yenidən başladır. `.env`-i dəyişəndən sonra isə serveri əl ilə yenidən başladın: terminalda **Ctrl+C**, sonra yenə `npm run dev`.

Bu terminalı açıq saxlayın: server işlədikcə orada qalır. Növbəti əmrləri **ayrıca terminal tabında** işlədin (VS Code-da terminal panelindəki **+** düyməsi). İşləyən serverin pəncərəsinə yazılan əmr icra olunmur.

`AI (gemini): ready` və `Speech recognition: Gemini` yazıları görünməlidir (Azure açarı varsa, nitq tanıma `Azure (az-AZ)` olur və `Azure TTS: ready` də çıxır). Açar olmasa da server işləyir: tətbiq sadə əmrləri (mesaj, zəng, kamera açmaq, fırıldaq qaydaları) telefonda özü başa düşür, yalnız süni intellekt tələb edən hissələr işləmir.

| Dəyişən | Nə üçün |
|---|---|
| `GEMINI_API_KEY` | Gemini: mürəkkəb əmrlər, şəkil oxuma, fırıldaq yoxlaması və Azərbaycan dilində nitq tanıma |
| `GEMINI_MODEL`, `GEMINI_VISION_MODEL` | Standart olaraq əmrlər, fırıldaq və nitq üçün `gemini-3.5-flash-lite` (ölçdük: ~1–3 san), kamera üçün `gemini-3.6-flash` (~6–9 san). `gemini-3.8-flash` pulsuz planda 75 saniyə çəkdi, səsli köməkçi üçün yaramır. `gemini-flash-latest` kimi `-latest` adları bu SDK-nın istifadə etdiyi `v1` API-də yoxdur |
| `ANTHROPIC_API_KEY`, `AI_PROVIDER=claude` | Gemini əvəzinə Claude istifadə etmək üçün (standart model `claude-opus-5-5`) |
| `AZURE_SPEECH_KEY`, `AZURE_SPEECH_REGION` | Əlinin təbii Azərbaycan səsi (`az-AZ-BanuNeural`). Qoşulanda nitq tanıma da Gemini yerinə Azure-un `az-AZ` modeli ilə gedir. Boş qalsa, telefonun öz səsi işləyir |
| `HOST`, `APP_TOKEN` | Telefonu USB yerinə Wi-Fi ilə qoşmaq üçün (aşağıya baxın) |

> **Nitq tanıma barədə vacib qeyd.** Google-un telefon daxilindəki nitq tanıyıcısında Azərbaycan dili modeli yoxdur (emulyatorda yoxladıq: `does not support locale: az-AZ`). Ona görə serverdə Gemini (və ya Azure) açarı olanda tətbiq səsi yazıb serverə göndərir və səs orada Azərbaycan dilində tanınır. Server açarı olmasa, telefonun tanıyıcısı sınanır. Alınmasa, «Yazaraq əmr ver» sahəsi həmişə işləyir.

## 2. Cihazı qoşmaq və tətbiqi yükləmək

**a) `adb` və `emulator` əmrlərini PATH-ə əlavə edin** (macOS, bir dəfə). Yoxsa `zsh: command not found: adb` xətası çıxır:

```bash
echo 'export PATH="$HOME/Library/Android/sdk/platform-tools:$HOME/Library/Android/sdk/emulator:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

**b) Cihazı qoşun.** Cihaz olmasa, quraşdırma `No connected devices!` xətası ilə dayanır.

- **Real telefon** (demo üçün bu lazımdır, çünki WhatsApp və mikrofon orada işləyir): Parametrlər → Telefon haqqında → «Build number» sətrinə 7 dəfə toxunun → Developer options → **USB debugging**-i aktiv edin. Telefonu USB ilə qoşun və telefonda «Allow USB debugging» sualına **Allow** deyin.
- **Emulyator:** bu skript emulyatoru Mac-in kamerası (telefonun kamerası kimi) və Mac-in mikrofonu ilə açır:

```bash
scripts/emulator.sh
```

İlk dəfə macOS kamera və mikrofon icazəsi soruşa bilər, **Allow** deyin. Android Studio → Device Manager → ▶ ilə açsanız, Mac mikrofonu söndürülmüş olur. Onu emulyatorun ⋯ → Microphone → «Virtual microphone uses host audio input» açarı ilə yandırın. Cihazın görünüb-görünmədiyini yoxlayın:

```bash
adb devices
```

Siyahıda `device` yazılmış bir sətir olmalıdır.

**c) «Əli» sözünü tanıyan oflayn modeli endirin** (layihəni yükləyəndən sonra bir dəfə, ~37 MB). Model olmasa, tətbiq yenə işləyir, sadəcə adı eşidəndə özü işə düşmür:

```bash
scripts/get-wake-model.sh
```

**d) Tətbiqi yükləyin.** Android Studio ilə: File → Open → `android/` qovluğu → ▶ Run. Ya da terminalla:

```bash
cd /Users/farid/NVATON/android && JAVA_HOME=$(/usr/libexec/java_home -v 21) ./gradlew installDebug
```

## 3. Telefonu backend-ə qoşmaq

Cihaz USB ilə qoşuludursa (emulyatorda da işləyir):

```bash
adb reverse tcp:8787 tcp:8787
```

Bu əmri cihazı hər dəfə yenidən qoşanda, emulyatoru yenidən açanda və `adb kill-server`-dən sonra təkrarlayın.

Həm telefon, həm emulyator qoşuludursa, adi `adb reverse` "more than one device" xətası verir. Cihazı göstərin: `-d` USB telefon, `-e` emulyator:

```bash
adb -d reverse tcp:8787 tcp:8787
```

Tətbiq standart olaraq `http://127.0.0.1:8787` ünvanına müraciət edir, bu əmr həmin ünvanı noutbukdakı serverə yönləndirir.

**Wi-Fi ilə qoşmaq:** `.env`-də `HOST=0.0.0.0` və `APP_TOKEN=gizli-söz` yazın, serveri yenidən başladın. Sonra tətbiqdə ⚙️ → Server bölməsində `http://<noutbukun-IP>:8787` ünvanını və həmin tokeni yazın. Token olmasa, eyni Wi-Fi-dakı hər kəs sizin Gemini açarınızdan istifadə edə bilər.

## 4. Telefonda quraşdırma (bir dəfə)

Tətbiqdə ⚙️ → **Quraşdırma** bölməsində hər ❌ sətrin yanındakı **Aç** düyməsini basın:

1. **Mikrofon, kamera, zəng, bildirişlər:** icazə verin.
2. **Fırıldaq qoruması:** "Bildirişlərə giriş" siyahısında **Köməkçi: fırıldaq qoruması**nı aktiv edin.
3. **WhatsApp-da avtomatik «Göndər» və ekranı oxumaq:** Əlçatımlılıq → **Köməkçi**ni aktiv edin. Eyni icazə hər iki iş üçündür.

> **Açar bozdursa (Android 13+):** Parametrlər → Tətbiqlər → Köməkçi → sağ yuxarıda ⋮ → **Məhdud parametrlərə icazə ver**. Sonra 2-ci və 3-cü addımı təkrarlayın.
>
> Tətbiqi yenidən yükləyəndə Android Əlçatımlılıq xidmətini söndürə bilər. Hər quraşdırmadan sonra ⚙️ → Quraşdırma bölməsinə baxın.

> **Xiaomi / Redmi (HyperOS, MIUI):** USB debugging üçün Telefon haqqında → **HyperOS versiyası**na 7 dəfə toxunun, sonra Əlavə parametrlər → Developer options → **USB debugging**. Kabeli taxanda USB rejimini **Fayl ötürmə** edin. «Əli»ni arxa planda eşitməsi üçün Tətbiqlər → Köməkçi → **Avtomatik başlama** açıq, **Batareyaya qənaət** «Məhdudiyyət yoxdur» olsun.

4. **Ailə kontaktları:** demo kontaktların nömrələri saxtadır (`+994 00 ...`). Onları silib komanda üzvlərinin real nömrələrini yazın. "Necə çağırılır" sahəsinə `nəvəm`, `qızım` kimi sözlər yazın.

## 5. Demo əmrləri

Düyməni basmadan **"Əli"** deyə bilərsiniz (aşağıdakı bölməyə baxın). Düymə ilə də işləyir.

| Deyin | Nə olur |
|---|---|
| *"Əli"* … *"nəvəmə yaz: salam"* | *"Bəli?"* → *"Kimə yazım: Murada, yoxsa Aysuya?"* → *"Murada"* → *"Murada «Salam» yazım?"* → *"Hə"* → WhatsApp-da göndərilir |
| *"Əli, nəvəmə yaz: salam"* (bir nəfəsdə) | "Bəli?" demədən dərhal *"Kimə yazım…"* |
| *"Qızıma de ki axşam gəlsin"* | *"Leylaya «Axşam gəlsin» yazım?"* |
| *"Əli, qızıma WhatsApp-dan salam yaz və göndər"* | *"Leylaya «Salam» yazım?"*: «və göndər» və «WhatsApp-dan» mesaja düşmür |
| *"Oğluma zəng et"* | Zəng edir |
| *"Əli, WhatsApp-dan Kamrana yaz ki sabah görüşək"* | Ailə siyahısında olmayan istənilən adama yazır: əvvəl telefonun kontaktlarında, sonra WhatsApp axtarışında tapır. *"Ailə qrupuna yaz: axşam gəlin"* qruplar üçün də işləyir. Göndərməzdən əvvəl yenə soruşur |
| *"Əli, Rauf adlı yeni kontakt saxla, nömrəsi sıfır əlli, yüz iyirmi üç, qırx beş, altmış yeddi"* | Nömrəni geri oxuyur (*"Rauf, 050 123 45 67. Yadda saxlayım?"*), «hə»dən sonra saxlayır. İnteqrasiya açıqdırsa, telefonun kontaktlarına da yazır. Ad və ya nömrə çatışmırsa, soruşur |
| Həmin nömrə başqa adla varsa | *"Bu nömrə artıq «Murad» adı ilə var. Köhnəni saxlayım, əvəz edim, yoxsa adları birləşdirim?"*: **Köhnəni saxla / Əvəz et / Birləşdir** |
| *"Əli, telefon kontaktlarımı əlavə et"* | Telefon kontaktları ekranı açılır: axtarış, seçim. Bir nömrə telefonda bir neçə adla yazılıbsa, hansı adla saxlanacağını soruşur, digər adlar ləqəb kimi qalır |
| *"Əli, Kamranla söhbəti aç"* | WhatsApp söhbətini açır. Sonra *"Əli, ekranı oxu"* ilə oxumaq olar. Oxunan söhbətdə *"ona yaz ki gəlirəm"* həmin adama cavab yazır |
| *"Əli, mesajlarımı oxu"*, *"kim yazıb?"* | Son gələn mesajları oxuyur (kimdən, hansı tətbiqdən, nə vaxt). Sonra *"nə vaxt gələcək?"* kimi davam suallarına həmin mesajlardan cavab verir |
| *"Əli, sabah səhər 7-də oyat"*, *"hər gün axşam 9-da dərmanımı xatırlat"*, *"10 dəqiqədən sonra zəng çal"* | Zəngli saat qurur. Vaxt gələndə ekrana çıxır, nə üçün olduğunu deyir və çalır: **Söndür** / **5 dəqiqə sonra** |
| *"Zəngli saatlarım hansılardır?"*, *"axşamkı xatırlatmanı sil"*, *"bütün zəngli saatları sil"* | Siyahını oxuyur, birini və ya hamısını silir. Siyahı ⚙️ → **⏰ Zəngli saatlar** bölməsində də görünür |
| *"Əli, dili ingilis dilinə dəyiş"*, *"switch to Azerbaijani"* | Dili səslə dəyişir |
| İstənilən tətbiqdə: *"Əli, ekranı oxu"*, *"Əli, ekranda kim yazıb?"* | Ekranı oxuyur və izah edir: kim nə yazıb, məbləğ, tarix. Rus və ingilis mətnini tərcümə edir, kod və ya pul istəyi görsə, xəbərdarlıq edir |
| *"Bu dərman nə üçündür?"*, *"Bu nədir?"* | Kamera açılır, *"Kameranı tutun, şəkil çəkirəm"* deyir, özü şəkil çəkir və **həmin suala** cavab verir |
| Kamera ekranında: *"Bəs gündə neçə dəfə içim?"*, *"Son istifadə tarixi keçib?"* | Yeni şəkil çəkir. Əvvəlki cavabları yadda saxlayır, ona görə qısa davam sualları da başa düşülür |
| *"Kameranı aç"* | Kamera açılır və sualınızı gözləyir (**🎤 Soruş** və ya **📷 Oxu**) |
| Kamera ekranında: *"Bağla"* | Ana ekrana qayıdır |
| *"Nə edə bilirsən?"* | Bacardıqlarını sadalayır |
| İstənilən başqa sual | Süni intellekt qısa cavab verir |

- Təsdiq suallarında ekranda iri **✅ Hə / ❌ Yox** düymələri də çıxır.
- Kamerada: fokus üçün ekrana toxunun, yaxınlaşdırmaq üçün iki barmaqla böyüdün. Telefonda fənər varsa, **🔦 İşıq** düyməsi çıxır.
- **Gələn bildirişlər** (WhatsApp, SMS, Telegram, Viber mesajları, buraxılmış zənglər, xatırlatmalar) gələn kimi səsli oxunur: *"Yeni mesaj, WhatsApp. Murad: …"*. Əli danışanda, siz danışanda, zəng zamanı və «Narahat etməyin» rejimində susur. ⚙️ → «Gələn mesajları və bildirişləri səsli oxu» ilə söndürülür.
- **WhatsApp söhbətində** *"Əli, ekranı oxu"* deyəndə hər mesajın kimdən olduğu, saatı və sizin yazdıqlarınız ayrıca oxunur, sonra davam sualları vermək olar.
- Səs-küylü zalda **⌨️ Yazaraq əmr ver** sahəsindən istifadə edin. Diakritik işarələr məcburi deyil: `neveme yaz salam` da işləyir.

## Dizayn

Tətbiqin görünüşü [taste skill](https://github.com/Leonxlnx/taste-skill) ilə qurulub: proqram təminatı üçün seçilmiş «Frost and slate» palitrası (açıq fon, tünd boz-mavi mətn, bir firuzəyi rəng), Chivo şrifti (Google Fonts, OFL, `assets/licenses/Chivo-OFL.txt`) və hər yerdə eyni kart, eyni düymə cütü, eyni künc ölçüsü. İkonlar Material Symbols Rounded-dır (Apache 2.0), kodda yol məlumatı kimi saxlanılır (`ui/AppIcons.kt`). Yaşlılar üçün böyük şrift, yüksək kontrast və ən azı 56 dp toxunma sahəsi saxlanılıb.

## Dil: Azərbaycanca / English

⚙️ → **🌐 Dil / Language** bölməsində **English** seçin. Ekran yazıları, Əlinin səsi (telefonun ingilis səsi), nitq tanıma və süni intellektin cavabları ingiliscəyə keçir. Azərbaycan dili standart olaraq qalır, seçim yadda saxlanılır.

| English | What happens |
|---|---|
| *"Ali"* … *"text my daughter that I'll be late"* | *"Send «I'll be late» to Leyla?"* → *"Yes"* → sent on WhatsApp |
| *"Ali, text my grandson hello"* | *"Who should I write to: Murad or Aysu?"* → *"the second one"* |
| *"Ali, what is this medicine?"* | Camera opens, takes a photo, answers in English |
| *"Ali, read the screen"* | Reads the current screen, translating Azerbaijani or Russian text |
| *"Call my son"*, *"What can you do?"* | Calls, lists what it can do |

- Kontaktlardakı `nəvəm`, `qızım`, `oğlum` kimi sözlər ingiliscə də tanınır: *my grandson*, *my daughter*, *my son*.
- İngiliscə mesaj və zəng əmrlərini süni intellekt başa düşür, ona görə server lazımdır. «Yes/No», kamera, ekran və «help» serversiz də işləyir.
- «Əli» sözünü yenə eyni model tanıyır, ingiliscə «Ali» kimi deyin.
- Azure səsi yalnız Azərbaycan dilindədir. İngiliscə rejimdə həmişə telefonun öz səsi danışır.

## «Əli» deyəndə özü işə düşməsi (Siri kimi)

- Tətbiq bir dəfə açılandan sonra arxa fonda "Əli" sözünü gözləyir. Ekranda **«Əli» deyin** yazısı görünür.
- Telefonun üst panelində bildiriş və yaşıl mikrofon nöqtəsi görünür. Android arxa fonda dinləyən hər tətbiqdən bunu tələb edir.
- Tətbiq bağlı və ya arxa fonda olsa da, "Əli" deyəndə özü ekrana gəlir və *"Bəli?"* deyir.
- Ad **telefonun içində, internetsiz** tanınır: kiçik oflayn model (Vosk). Azərbaycan dili üçün model olmadığından türk modelindən istifadə olunur, o "Əli" sözünü "ali / eli" kimi eşidir. Səs serverə yalnız ad eşidiləndən **sonra** gedir.
- Ad cümlənin əvvəlində olmalıdır: *"Onun əli ağrıyır"* və ya *"Allah razı olsun"* tətbiqi oyatmır.
- Söndürmək üçün: ⚙️ → **«Əli» deyəndə özü işə düşsün** açarı, ya da bildirişdəki **Söndür** düyməsi.
- Köməkçinin adını dəyişsəniz (⚙️ → Köməkçinin adı), yeni adı gözləyir. Adı türk modeli tanımalıdır: Ayla, Leyla, Xanım kimi geniş yayılmış adlar işləyir.
- Danışarkən və sizi dinləyərkən adı gözləmir, ona görə öz səsi ilə "Əli" deyəndə özünü oyatmır.

## Danışmadan yoxlamaq (emulyator)

Əmri iki qat dırnağın içində yazın, yoxsa telefon onu ilk boşluqda kəsir:

```bash
adb shell "am start -n az.nvaton.komekci/.MainActivity --es command 'nəvəmə yaz salam'"
```

Kamera ilə sual (kamera açılır, şəkil çəkir, cavab verir):

```bash
adb shell "am start -n az.nvaton.komekci/.MainActivity --es command 'bu nədir?'"
```

"Əli" eşidilmiş kimi davranmaq:

```bash
adb shell am start -n az.nvaton.komekci/.MainActivity --ez simulate_wake true
```

Dinləməni və danışmanı dayandırmaq:

```bash
adb shell am start -n az.nvaton.komekci/.MainActivity --ez stop true
```

Oyanma sözünü səs faylı ilə yoxlamaq (16 kHz mono WAV, mikrofon bunu eşitmiş kimi):

```bash
adb exec-in run-as az.nvaton.komekci sh -c 'cat > files/ali.wav' < ali.wav
```

```bash
adb shell am start -n az.nvaton.komekci/.MainActivity --es wake_test ali.wav
```

Ekran oxumanı mikrofonsuz yoxlamaq: tətbiq kənara çəkilir, arxadakı ekranı oxuyur, sonra «ekranı oxu» deyilmiş kimi davam edir:

```bash
adb shell am start -n az.nvaton.komekci/.MainActivity --ez screen_test true
```

Modelin nə eşitdiyini görmək:

```bash
adb logcat -s WakeWord
```

Fırıldaq SMS-i göndərmək (yalnız emulyatorda):

```bash
adb emu sms send 5551234 "Hörmətli müştəri, kartınız bloklanıb. Kodu bu nömrəyə göndərin"
```

Real telefonda isə ⚙️ → **Fırıldaq xəbərdarlığını yoxla** düyməsi var.

## Testlər

```bash
cd backend && npm test
```

```bash
cd android && ./gradlew testDebugUnitTest
```

## Problemlər

| Problem | Həll |
|---|---|
| `No connected devices!` | Telefon qoşulmayıb və ya emulyator açıq deyil. 2-ci bölmənin b) addımına baxın, `adb devices` ilə yoxlayın |
| `zsh: command not found: adb` | `adb` PATH-də deyil: 2-ci bölmənin a) addımı |
| `EADDRINUSE: address already in use :::8787` | Server artıq başqa terminalda işləyir. Birini Ctrl+C ilə dayandırın |
| "Serverə qoşula bilmədim" | Server işləyir? `adb reverse tcp:8787 tcp:8787` icra olunub? ⚙️ → Server → "Yadda saxla və yoxla" |
| "Süni intellekt hələ qoşulmayıb" | `backend/.env`-də `GEMINI_API_KEY` yoxdur və ya səhvdir (serverin terminalına baxın) |
| Server logunda `429` / `rate limit` | Gemini-nin pulsuz planının limiti dolub: bir az gözləyin və ya Google AI Studio-da ödənişli plana keçin |
| Nitq tanıma Azərbaycan dilini başa düşmür | `backend/.env`-ə Gemini (və ya Azure) açarını əlavə edin: tətbiq avtomatik serverin nitq tanımasına keçir (⚙️ → «Serverin nitq tanıması» açıq olmalıdır). Ya da yazaraq əmr verin |
| Əli türk ləhcəsi ilə danışır | Telefonda Azərbaycan səsi yoxdur. Backend-ə Azure açarını əlavə edin |
| WhatsApp açılır, amma mesaj öz-özünə getmir | Əlçatımlılıq xidməti aktiv deyil (4-cü bölmə). Nömrə beynəlxalq formatda olmalıdır: `+994...` |
| "Əli" deyəndə işə düşmür | ⚙️ → «Əli» deyəndə özü işə düşsün: "✅ Adınızı gözləyir" yazılmalıdır. "Model tətbiqə daxil edilməyib" yazılıbsa: `scripts/get-wake-model.sh`, sonra tətbiqi yenidən yükləyin. Emulyatorda Mac mikrofonu açıq olmalıdır (`scripts/emulator.sh`). Telefona yaxından, aydın deyin, adı deyib bir anlıq dayanın. `adb logcat -s WakeWord` modelin nə eşitdiyini göstərir |
| Zəngli saat çalanda ekran açılmır, yalnız bildiriş görünür | ⚙️ → Quraşdırma → «⏰ Zəngli saat bütün ekranda» → Aç (Android 14+). Xiaomi-də Tətbiqlər → Köməkçi → **Avtomatik başlama** açıq olsun |
| Telefon kontaktları görünmür | ⚙️ → Ailə kontaktları → «Telefon kontaktları ilə inteqrasiya» açarını yandırın və icazə verin. Sonra «Telefondan əlavə et» |
| "Kamrana yaz" deyəndə adamı tapmır | ⚙️ → Quraşdırma → «Kontaktlar: istənilən adama yazmaq» → Aç. Kontaktda olmayan adlar və qruplar WhatsApp axtarışı ilə tapılır: bunun üçün əlçatımlılıq açıq olmalıdır |
| Gələn mesajları oxumur | ⚙️ → Quraşdırma → «🔔 Bildirişlər» ✅ olmalıdır və «Gələn mesajları… səsli oxu» açıq olmalıdır. «Narahat etməyin» rejimində oxumur |
| "Əli, ekranı oxu" deyəndə "Oxumaq istədiyiniz ekranı açın" deyir | Ekran adı çağırdığınız anda götürülür. Köməkçini əvvəlcə özünüz açsanız, ekranda Köməkçinin özü olur. Lazım olan tətbiqdə «Əli, ekranı oxu» deyin |
| Emulyatorda «Əli» bir müddət sonra eşitmir | Emulyatorun Mac mikrofonu bir-iki dəqiqədən sonra kəsilir: `adb logcat -s WakeWord`-da `peak 0` və ya dəyişməyən `peak 23191` (süni ton) görünür. Emulyatoru `scripts/emulator.sh` ilə yenidən açın. Real telefonda bu yoxdur |
| Emulyatorda kamera otaq şəkli göstərir | Emulyator `scripts/emulator.sh` ilə açılmayıb. Ya da Android Studio → Device Manager → ✏️ → Camera → Back: **Webcam0** |
| Emulyatorda "Speech Recognition and Synthesis from Google keeps stopping" | Google-un telefondakı türk səsi Apple çipli Mac-in emulyatorunda çökür. Tətbiq bunu özü aşkar edir və bir neçə saniyəyə Google-un onlayn səsinə keçir: dialoqu bağlayın. Real telefonda və Azure səsi ilə bu problem yoxdur |
| "Süni intellekt hazırda çox məşğuldur" | Gemini həmin anda yüklənib (503). Server avtomatik digər Gemini modelini sınayır, o da cavab verməsə, bir azdan yenidən soruşun |

## Təhlükəsizlik

- API açarları yalnız `backend/.env`-dədir, APK-da yoxdur.
- Əlçatımlılıq xidməti yalnız WhatsApp-da düymə basır və yalnız səslə təsdiq olunan mesajı 20 saniyə ərzində göndərir. Mətn qutusundakı yazı təsdiq olunan mətnlə eyni deyilsə, heç nə basmır.
- Gələn bildirişlər yalnız telefonda səsləndirilir. Son mesajlar «mesajlarımı oxu» üçün 3 saata qədər yalnız yaddaşda saxlanılır (diskdə yox). Davam sualı verəndə (məs. «nə vaxt gələcək?») həmin mesajlar və ya ekran mətni cavab üçün serverə gedir.
- Telefon kontaktları ilə inteqrasiya yalnız istifadəçi razılıq verəndən sonra açılır (Tənzimləmələr və ya idxal ekranı), istənilən vaxt söndürülür.
- Telefonun kontaktları yalnız telefonda axtarılır, serverə getmir. Server yalnız dediyiniz adı görür (məs. «Kamran»).
- Zəngli saatlar telefonda saxlanılır. Server yalnız əmri başa düşmək üçün siyahının saatlarını və etiketlərini görür.
- Dil seçimi serverə hər sorğu ilə `language: "az" | "en"` kimi gedir, başqa heç nə dəyişmir.
- Ekran yalnız «Əli» deyiləndə götürülür, 3 dəqiqə yalnız yaddaşda qalır, şifrə sahələri oxunmur. Mətn serverə (Gemini) yalnız «ekranı oxu» deyəndə gedir.
- Fırıldaq yoxlaması əvvəl telefonda gedir. Serverə yalnız şübhəli mesajlar göndərilir.
- Serverin nitq tanıması işləyəndə səs yalnız mikrofon düyməsi basılandan və ya "Əli" deyiləndən sonra, sükut olana qədər yazılır və tanınmaq üçün serverə (Gemini və ya Azure) göndərilir. Heç yerdə saxlanılmır.
- "Əli" sözünü gözləyərkən səs yalnız telefonun içində emal olunur və heç yerə göndərilmir.
- Kameranın şəkli yalnız sual veriləndə və ya **Oxu** basılanda çəkilir və cavab üçün serverə göndərilir.
- Gemini-nin pulsuz planında Google sorğuları öz məhsullarını yaxşılaşdırmaq üçün istifadə edə bilər. Real istifadəçilərin mesajları üçün ödənişli plana keçin.
- Claude seçiləndə (`AI_PROVIDER=claude`) server tərəfli `fallbacks: "default"` aktivdir: təhlükəsizlik filtri sorğunu rədd etsə, API onu avtomatik olaraq tövsiyə olunan modeldə yenidən icra edir.
