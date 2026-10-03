# MP4 to MP3 Converter

A simple, lightweight Windows desktop application for converting MP4 videos to high-quality MP3 audio files.

Built with **C# / .NET 8 WinForms** and powered by **FFmpeg**.

> No command line knowledge required. Just drag and drop your MP4 files, choose an output folder, and click **Convert**.

## ✨ Features

- 🎵 Convert MP4 videos to MP3 audio
- 📁 Drag-and-drop MP4 files
- 📦 Batch conversion
- ⏭️ Automatically skip MP3 files that already exist
- 📊 Overall batch progress indicator
- 📝 Real-time conversion log
- 🔄 Sequential processing for reliable batch conversion
- 💾 Remembers the last output folder
- 🪟 Clean Windows 10/11 desktop interface
- 📦 Portable, self-contained Windows build
- 🚫 No .NET installation required on the target computer
- 🔊 Preserves the original audio pitch and tempo
- ⚡ Uses FFmpeg's high-quality MP3 encoding

## 📸 Screenshots

> Add screenshots of the application here.

```text
Coming soon
```

## 🛠️ Technology Stack

| Component        | Technology                    |
| ---------------- | ----------------------------- |
| Language         | C#                            |
| Framework        | .NET 8                        |
| UI               | Windows Forms                 |
| Audio Conversion | FFmpeg 7.x                    |
| MP3 Encoder      | LAME                          |
| Target Platform  | Windows 10/11 x64             |
| Distribution     | Self-contained portable build |

The project uses standard WinForms controls with a clean, Fluent-inspired interface. No paid UI component library is required.

## 📂 Project Structure

```text
mp4/
├── Mp4ToMp3.sln
├── Mp4ToMp3/
│   ├── Mp4ToMp3.csproj
│   ├── Program.cs
│   ├── NOTICE.txt
│   ├── Forms/
│   │   ├── MainForm.cs
│   │   ├── MainForm.Designer.cs
│   │   └── CompletionDialog.cs
│   ├── Services/
│   │   ├── ConversionOrchestrator.cs
│   │   ├── FFmpegRunner.cs
│   │   └── SettingsStore.cs
│   └── Models/
│       └── ConversionResult.cs
├── ffmpeg-7.1.1-essentials_build/
│   └── bin/
│       └── ffmpeg.exe
└── test/
    └── # Optional sample MP4 files
```

## 🚀 Getting Started

### Requirements

For development, you need:

- Windows 10/11 64-bit
- .NET 8 SDK
- FFmpeg 7.x Essentials Build for Windows x64
- Visual Studio 2022, JetBrains Rider, or another C# IDE

The released portable version does **not** require the .NET runtime to be installed on the target computer.

### Clone the Repository

```bash
git clone https://github.com/<your-username>/mp4-to-mp3-converter.git
cd mp4-to-mp3-converter
```

### Build

```bash
dotnet build Mp4ToMp3/Mp4ToMp3.csproj -c Release
```

### Run

```bash
dotnet run --project Mp4ToMp3/Mp4ToMp3.csproj
```

## 🎯 How to Use

### 1. Add MP4 files

Drag and drop one or more `.mp4` files into the application.

Alternatively, click **Add Files** and select MP4 files.

Only `.mp4` files are accepted.

### 2. Select an output folder

Click **Browse** and choose where you want the MP3 files to be saved.

The application remembers your last selected output folder.

### 3. Start conversion

Click **Convert**.

The application processes files sequentially and displays the current progress in the log panel.

Example:

```text
Processing lesson1.mp4...
Done: lesson1.mp4

Processing lesson2.mp4...
Done: lesson2.mp4
```

### 4. Find your converted files

The generated MP3 files use the same base filename:

```text
lesson1.mp4  →  lesson1.mp3
lecture.mp4  →  lecture.mp3
```

## 🔄 Batch Processing

Files are processed **one at a time** rather than concurrently.

For each input file, the converter can produce one of three outcomes:

| Result    | Description                  |
| --------- | ---------------------------- |
| Succeeded | MP3 was successfully created |
| Skipped   | MP3 already exists           |
| Failed    | Conversion failed            |

If one file fails, the application logs the error and continues processing the remaining files.

## ⏭️ Skip Existing Files

The application never overwrites an existing MP3 file.

For example, if:

```text
output/
└── lecture.mp3
```

already exists when converting:

```text
lecture.mp4
```

the converter skips it and logs:

```text
Skipped (already exists): lecture.mp4
```

This makes it safe to run the same batch multiple times.

## 🎧 Audio Conversion

The application uses FFmpeg with the following conversion command:

```bash
ffmpeg.exe -i "input.mp4" -vn -c:a libmp3lame -q:a 0 "output.mp3"
```

The options mean:

- `-i` — input MP4 file
- `-vn` — remove the video stream
- `-c:a libmp3lame` — encode audio using the LAME MP3 encoder
- `-q:a 0` — highest VBR quality setting

No pitch, tempo, or speed filters are applied.

## 📈 Progress Tracking

The progress bar represents **overall batch progress** rather than the progress of an individual MP4 file.

Progress is calculated as:

```text
completed files × 100 / total files
```

The progress reaches `100%` when all files have been processed, including batches where every file was skipped.

## 💾 Settings

The application stores the last selected output directory at:

```text
%LocalAppData%\Mp4ToMp3\settings.json
```

Example:

```json
{
  "OutputFolder": "C:\\Users\\Example\\Music"
}
```

If no settings exist, the application defaults to the user's **Music** folder.

## 📦 Portable Version

The application can be published as a self-contained Windows x64 application.

```bash
dotnet publish Mp4ToMp3/Mp4ToMp3.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=false -o Mp4ToMp3-Portable
```

The resulting folder contains the application and its required .NET runtime files, together with the bundled FFmpeg executable.

### Portable package

```text
Mp4ToMp3-Portable/
├── Mp4ToMp3.exe
├── *.dll
├── *.json
├── NOTICE.txt
└── ffmpeg-7.1.1-essentials_build/
    └── bin/
        └── ffmpeg.exe
```

Users can:

1. Extract the ZIP file
2. Double-click `Mp4ToMp3.exe`
3. Add MP4 files
4. Select an output folder
5. Click **Convert**

No administrator privileges or .NET installation are required.

## 🧪 Testing

The project specification includes the following acceptance tests:

- Convert multiple MP4 files successfully
- Verify progress reaches 100%
- Verify existing MP3 files are skipped
- Verify non-MP4 files are rejected
- Test Remove and Clear
- Verify Browse and output-folder controls are correctly aligned
- Verify Convert is centered
- Verify Open Output Folder
- Verify missing FFmpeg handling
- Verify portable execution without the .NET runtime
- Verify the UI remains usable when resized

## 🚫 Non-Goals

This project intentionally keeps the feature set small.

It does **not** provide:

- Video editing
- Pitch adjustment
- Tempo/speed adjustment
- Cloud uploads
- Parallel multi-file conversion
- Conversion from formats other than `.mp4`

The goal is a focused MP4 → MP3 utility rather than a general-purpose media editor.

## 🏗️ Architecture

```text
                    ┌─────────────────┐
                    │    MainForm     │
                    │      (UI)       │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
       ┌────────────┐ ┌───────────────┐ ┌───────────────┐
       │  Settings  │ │  Conversion   │ │  Completion   │
       │   Store    │ │ Orchestrator  │ │    Dialog     │
       └────────────┘ └───────┬───────┘ └───────────────┘
                               │
                               ▼
                       ┌───────────────┐
                       │ FFmpegRunner  │
                       └───────┬───────┘
                               │
                               ▼
                         ┌───────────┐
                         │  FFmpeg   │
                         └───────────┘
```

### Main components

**`MainForm`**

Responsible for:

- Drag and drop
- File queue management
- Output folder selection
- Progress display
- Logging
- Starting conversion

**`ConversionOrchestrator`**

Responsible for:

- Sequential batch processing
- Skip-if-exists behavior
- Progress reporting
- Aggregating conversion results

**`FFmpegRunner`**

Responsible for:

- Locating `ffmpeg.exe`
- Starting the FFmpeg process
- Capturing FFmpeg errors
- Returning the process result

**`SettingsStore`**

Responsible for loading and saving the user's output folder.

**`CompletionDialog`**

Displays the final conversion summary and provides an option to open the output folder.

## 🔐 File Handling

The application:

- Stores input file paths locally in memory
- Does not upload files to a server
- Processes files locally using FFmpeg
- Does not overwrite existing MP3 files
- Accepts only `.mp4` input files

There is no cloud conversion service involved.

## 📄 License

This project uses FFmpeg for audio conversion.

FFmpeg Essentials builds are typically distributed under the **GNU General Public License (GPL) v3**. The distribution should include the appropriate FFmpeg license and attribution information in `NOTICE.txt`.

Please make sure that your distribution of FFmpeg complies with the applicable FFmpeg license requirements.

## 🤝 Contributing

Contributions, bug reports, and suggestions are welcome.

Before submitting a pull request, please make sure that:

1. The project builds successfully.
2. Existing MP3 files are never overwritten.
3. MP4 → MP3 conversion behavior is preserved.
4. The UI remains usable at different window sizes.
5. Conversion logic remains separated from UI layout code.
6. No pitch or tempo modification is introduced.

## 🐛 Known Limitations

This is intentionally a small utility and currently focuses on basic MP4 → MP3 conversion.

It does not provide:

- Audio editing
- Metadata editing
- Multiple output formats
- Audio normalization
- Custom bitrate controls
- Parallel conversion
- Video processing

## 📋 Roadmap

Potential future improvements may include:

- [ ] Application icon
- [ ] Installer package
- [ ] Automatic update support
- [ ] More detailed FFmpeg error reporting
- [ ] Audio metadata preservation/editing
- [ ] User-selectable MP3 quality
- [ ] Additional input formats

These features are not part of the current specification.

## ⭐ Acknowledgements

This project uses:

- **.NET 8** — application framework
- **Windows Forms** — desktop UI framework
- **FFmpeg** — audio extraction and conversion
- **LAME** — MP3 encoding

---

**MP4 to MP3 Converter** — a small, focused Windows utility for turning MP4 videos into high-quality MP3 audio files.
