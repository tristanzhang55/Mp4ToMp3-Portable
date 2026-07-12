# MP4 to MP3 Converter — Complete Specification

Use this document as the single source of truth to build the same Windows desktop tool with Cursor (or any coding agent). Follow every requirement unless a section explicitly marks something as optional.

---

## 1. Product Overview

### 1.1 Goal

Build a small Windows desktop utility that:

- Accepts one or more **MP4** files as input
- Produces **MP3** files as output
- Extracts audio **without pitch/tempo changes** (no voice alteration)
- Supports **batch processing**
- Provides a simple English GUI for users who do not know FFmpeg

### 1.2 Target users

- General Windows users
- Users unfamiliar with command-line tools
- Users who need basic MP4 → MP3 conversion

### 1.3 Non-goals

- No video editing
- No pitch/speed/tempo controls
- No cloud upload
- No parallel multi-file conversion (keep sequential)
- Do not accept non-`.mp4` formats (no `.mov`, `.m4v`, `.mkv`, etc.)

---

## 2. Tech Stack

| Item | Requirement |
|------|-------------|
| Framework | .NET 8 |
| UI | WinForms (`net8.0-windows`) |
| UI styling | Prefer free libraries (e.g. ReaLTaiizor) **or** standard WinForms with Fluent-inspired flat styling. Do **not** require a paid Guna.UI2 license. |
| Language | C# |
| Audio engine | Bundled FFmpeg 7.x Essentials Build |
| Distribution | Self-contained `win-x64` publish (no .NET install required on target PCs) |
| UI language | English only |

### 2.1 Recommended project layout

```
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
├── ffmpeg-7.1.1-essentials_build/     # bundled FFmpeg (bin/ffmpeg.exe required)
└── test/                              # optional sample MP4s for manual testing
```

### 2.2 Project file requirements (`Mp4ToMp3.csproj`)

- `OutputType`: `WinExe`
- `TargetFramework`: `net8.0-windows`
- `UseWindowsForms`: `true`
- `Nullable`: `enable`
- Copy FFmpeg binaries into output:

```xml
<None Include="..\ffmpeg-7.1.1-essentials_build\bin\**\*"
      Link="ffmpeg-7.1.1-essentials_build\bin\%(RecursiveDir)%(Filename)%(Extension)">
  <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
</None>
```

- Also copy `NOTICE.txt` to output.

---

## 3. FFmpeg Integration

### 3.1 Version

- FFmpeg **7.x Essentials** build for Windows x64
- Expected path relative to the app base directory:

```text
{AppContext.BaseDirectory}/ffmpeg-7.1.1-essentials_build/bin/ffmpeg.exe
```

If the folder name differs slightly, keep the same relative structure: `ffmpeg-*-essentials_build/bin/ffmpeg.exe`.

### 3.2 Conversion command

For each file, run exactly this style of command (preserve pitch; extract audio only):

```bash
ffmpeg.exe -i "input.mp4" -vn -c:a libmp3lame -q:a 0 "output.mp3"
```

Meaning:

- `-vn` — discard video
- `-c:a libmp3lame` — MP3 via LAME
- `-q:a 0` — highest VBR quality

Do **not** add filters that change pitch, tempo, or speed.

### 3.3 Process execution rules

- Spawn FFmpeg as a child process (`Process`)
- `CreateNoWindow = true`
- Redirect stderr (FFmpeg logs to stderr)
- Treat exit code `0` as success
- On Convert start, if `ffmpeg.exe` is missing, show a clear error dialog and abort

### 3.4 Licensing note

FFmpeg Essentials builds are typically **GPL v3**. Ship a `NOTICE.txt` that attributes FFmpeg and points to its license/source. Include FFmpeg’s `LICENSE` with the portable package.

---

## 4. Conversion Behavior

### 4.1 Output path

```text
{OutputFolder}/{SameBaseName}.mp3
```

Example: `lesson1.mp4` → `lesson1.mp3` in the chosen output folder.

### 4.2 Skip-if-exists (required)

If the target MP3 already exists:

- **Do not overwrite**
- Log: `Skipped (already exists): {filename}.mp4`
- Count as **Skipped**
- Continue with remaining files

### 4.3 Batch processing

- Process files **sequentially** (one after another)
- On failure of one file: log the error, count as **Failed**, continue the rest
- Track totals: `Succeeded`, `Skipped`, `Failed`, and list of failed file names

### 4.4 Progress

Show **overall batch progress** only:

```text
percent = completedCount * 100 / totalCount
```

Update after each file finishes (success, skip, or fail). Progress must reach 100% when the batch ends, including all-skipped runs.

### 4.5 Logging (main window log panel)

Use these message patterns:

| Event | Log line |
|-------|----------|
| Start file | `Processing {name}...` |
| Success | `Done: {name}` |
| Skip | `Skipped (already exists): {name}` |
| Fail | `Failed: {name} — {short error}` |
| Reject non-MP4 drop | `Rejected (not MP4): {name}` |

### 4.6 Validation before Convert

Block Convert and show a message if:

1. Queue is empty → “Add at least one MP4 file before converting.”
2. Output folder missing/invalid → “Select a valid output folder before converting.”
3. FFmpeg missing → clear “FFmpeg Missing” error

While converting: disable Add / Remove / Clear / Browse / Convert (and preferably the file list / drop zone). Re-enable when finished.

---

## 5. UI Specification

### 5.1 Design language

- English UI
- Resizable window
- Clean Windows 11 / Fluent-inspired look: light gray background, Segoe UI, flat buttons, subtle borders
- Polished enough to ship as a small commercial utility
- **Do not** copy ASCII wireframes pixel-for-pixel; match the structure and hierarchy

### 5.2 Main window title

`MP4 to MP3 Converter`

Suggested defaults:

- Size ≈ `820 × 720`
- Minimum size ≈ `720 × 640`
- Start centered on screen

### 5.3 Main window layout (top → bottom)

```text
┌──────────────────────────────────────────────────────────┐
│  MP4 to MP3 Converter                         (title bar)│
├──────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────────────────────────┐  │
│  │     Drag & drop MP4 files here                     │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  lesson1.mp4                                       │  │
│  │  lesson2.mp4                                       │  │
│  │  lecture.mp4                                       │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  [ Add Files ]   [ Remove ]   [ Clear ]                  │
│                                                          │
│  Output Folder                                           │
│  [ path text box ........................ ] [ Browse ]   │
│                                                          │
│                    [ Convert ]                           │
│                                                          │
│  Progress                                                │
│  ████████████████████░░░░░░░░░░░░░░░░░░░░░░░░  68%       │
│                                                          │
│  Log                                                     │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Processing lesson2.mp4...                          │  │
│  │ Done: lesson2.mp4                                  │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

### 5.4 Layout rules (critical — avoid messy UI)

These rules are mandatory because Material/third-party buttons often have unpredictable preferred sizes:

1. Use **predictable button sizes**. Prefer standard WinForms `Button` with flat styling for action buttons if a UI library causes overlaps.
2. Lay out controls **top-to-bottom** with consistent margins/gaps (e.g. margin 24px, gap 8–12px).
3. **No overlapping** controls. Buttons must not overlap each other or labels.
4. **Add Files / Remove / Clear**: same height, evenly spaced in one row under the file list.
5. **Browse** button height must **exactly match** the Output Folder text box height (e.g. both 28px).
6. **Convert** button must be **horizontally centered** in the window (not right-aligned).
7. Progress bar spans most of the width; percent label sits to its right.
8. Log box fills remaining vertical space when the window is resized.
9. Recalculate positions on `Resize` so the layout stays clean when the window grows/shrinks.

### 5.5 Control behaviors

| Control | Behavior |
|---------|----------|
| Drop zone + file list | Accept drag-and-drop of files; only enqueue `.mp4`; reject others with a log message |
| Add Files | `OpenFileDialog`, multi-select, filter `MP4 files (*.mp4)\|*.mp4` |
| Remove | Remove **selected** list items (and their full paths from the queue) |
| Clear | Empty the queue and list |
| Output Folder text box | Read-only display of selected folder path |
| Browse | `FolderBrowserDialog`; save chosen path to settings |
| Convert | Start async batch conversion without freezing the UI (`async/await` + `IProgress<T>`) |
| Progress bar | Overall batch percent |
| Log | Append-only multiline text box |

### 5.6 File queue rules

- Store full paths internally; show file names in the list
- Deduplicate by full path (case-insensitive)
- Only `.mp4` extension accepted

### 5.7 Settings persistence

- Persist last-used output folder to:

```text
%LocalAppData%\Mp4ToMp3\settings.json
```

Example content:

```json
{ "OutputFolder": "C:\\Users\\Example\\Music" }
```

- On first launch (no settings): default to `Environment.SpecialFolder.MyMusic`
- Save folder when Browse succeeds and when Convert starts successfully

### 5.8 Custom completion dialog (required)

Do **not** use a plain `MessageBox` for completion. Show a dedicated modal form:

- Title: `Conversion Complete`
- Body: `{X} converted, {Y} skipped, {Z} failed`
- Primary button: **Open Output Folder** → open the folder in Explorer (`Process.Start` with `UseShellExecute = true`)
- Secondary button: **Close**
- Style consistent with the main window (flat buttons, Segoe UI, light theme)
- Fixed dialog, centered on parent, no maximize/minimize

---

## 6. Architecture

```text
MainForm
  ├── File queue (in-memory HashSet/list of full paths)
  ├── SettingsStore          → read/write last output folder
  ├── ConversionOrchestrator → sequential batch + skip + progress
  │     └── FFmpegRunner     → locate & run ffmpeg.exe
  └── CompletionDialog       → summary + open folder
```

Suggested responsibilities:

| Type | Responsibility |
|------|----------------|
| `MainForm` | UI, drag-drop, queue editing, progress/log display, start conversion |
| `FFmpegRunner` | Resolve `ffmpeg.exe`, run conversion process, return success + stderr |
| `ConversionOrchestrator` | Loop files, skip-exists, report progress, aggregate results |
| `SettingsStore` | Load/save `settings.json` |
| `BatchConversionResult` / `ConversionProgress` | Result and progress DTOs |
| `CompletionDialog` | Post-batch summary UI |

Keep conversion logic out of the Designer file. Keep layout code separate from conversion logic so UI tweaks do not break FFmpeg behavior.

---

## 7. Portable Distribution

### 7.1 Publish command

```bash
dotnet publish Mp4ToMp3/Mp4ToMp3.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=false -o Mp4ToMp3-Portable
```

Requirements:

- `--self-contained true` so friends need no .NET install
- `PublishSingleFile=false` so `ffmpeg.exe` remains a separate file
- Target: Windows 10/11 **64-bit**

### 7.2 Portable package contents

Include at least:

- `Mp4ToMp3.exe` (+ runtime DLLs from publish)
- `ffmpeg-7.1.1-essentials_build/bin/ffmpeg.exe`
- `NOTICE.txt`
- Short `README.txt` with: unzip → double-click exe → add files → convert

Optional size reduction: omit `ffplay.exe` / `ffprobe.exe` from the shared zip if unused.

### 7.3 Sharing

Zip the publish folder (e.g. `Mp4ToMp3-Portable.zip`) and share. Recipients unzip anywhere and run `Mp4ToMp3.exe` (no admin required).

---

## 8. Acceptance / Test Checklist

Manual tests (use sample MP4s if available):

1. **First convert** — Add 4 MP4s, choose empty output folder, Convert → 4 MP3s created; progress ends at 100%; completion dialog shows `4 converted, 0 skipped, 0 failed`.
2. **Skip existing** — Convert again with same output folder → all skipped; progress still reaches 100%; dialog shows skips.
3. **Reject non-MP4** — Drop a `.txt` (or other) → log `Rejected (not MP4): ...`; file not added.
4. **Remove / Clear** — Selected remove works; Clear empties the list.
5. **Browse height** — Browse button height matches Output Folder text box; no overlaps with labels/buttons.
6. **Convert centered** — Convert button is horizontally centered.
7. **Open folder** — Completion dialog “Open Output Folder” opens Explorer at the output path.
8. **Missing FFmpeg** — If binary removed, Convert shows a clear error.
9. **Portable run** — Published self-contained build starts on a machine without the .NET SDK/runtime installed.
10. **UI resize** — Resizing the window does not cause overlaps; log area grows.

---

## 9. Implementation Notes for Cursor Agents

1. Create the solution/project first, then services, then forms.
2. Verify FFmpeg is copied to `bin` output before testing conversion.
3. Prefer **standard flat WinForms buttons** for Add/Remove/Clear/Browse/Convert if a third-party control library causes overlap or oversized buttons.
4. Never change conversion semantics when fixing UI layout.
5. Do not overwrite existing MP3s.
6. Do not add pitch/tempo filters.
7. Keep the UI English.
8. Ship `NOTICE.txt` for FFmpeg GPL attribution.

---

## 10. Prompt Starter (optional)

Paste this into Cursor after attaching this specification:

```text
Implement the MP4 to MP3 Converter exactly as described in SPECIFICATION.md.
Create a .NET 8 WinForms app, bundle FFmpeg, support batch convert with skip-if-exists,
overall progress, drag-and-drop, settings persistence, custom completion dialog,
and a self-contained win-x64 portable publish. Keep the UI clean with no overlapping
controls; center the Convert button; match Browse height to the output path text box.
```

---

## 11. Reference Command Summary

```bash
# Create / build
dotnet new sln -n Mp4ToMp3
dotnet new winforms -n Mp4ToMp3 -o Mp4ToMp3 -f net8.0
dotnet sln add Mp4ToMp3/Mp4ToMp3.csproj
dotnet build Mp4ToMp3/Mp4ToMp3.csproj -c Release

# Portable publish
dotnet publish Mp4ToMp3/Mp4ToMp3.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=false -o Mp4ToMp3-Portable

# FFmpeg conversion (core)
ffmpeg.exe -i "input.mp4" -vn -c:a libmp3lame -q:a 0 "output.mp3"
```
