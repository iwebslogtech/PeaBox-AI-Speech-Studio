# PeaBox Speech Studio Installation Guide

This guide covers Windows and Linux installation, Python-version management, FFmpeg, local Kokoro setup, cloud-provider credentials, upgrades, verification, and common errors.

## 1. System Requirements

### Recommended

- 64-bit operating system
- Python 3.12
- 8 GB RAM
- 5 GB free disk space
- Internet connection for package installation, Kokoro's initial model download, and cloud providers
- FFmpeg for compressed audio and video transcription

### Important Python Note

Python 3.13 can remain installed for other applications. PeaBox Speech Studio should use a separate Python 3.12 virtual environment because some speech and phonemization packages may not install correctly under Python 3.13.

A virtual environment created with Python 3.13 cannot be converted to Python 3.12. Delete and recreate the environment.

## 2. Windows Installation

### Step 1: Install Python 3.12

If Python 3.12 is not installed:

```cmd
winget install --exact --id Python.Python.3.12
```

Alternatively, download 64-bit Python 3.12 from:

https://www.python.org/downloads/

During installation:

- Select **Add Python to PATH**.
- Keep the **Python Launcher** option enabled.

Close and reopen Command Prompt after installation.

### Step 2: Verify Installed Versions

```cmd
py -0p
```

Verify Python 3.12 directly:

```cmd
py -3.12 --version
```

Expected output:

```text
Python 3.12.x
```

Python 3.13 may still appear as the default. That is acceptable because the project launcher explicitly selects Python 3.12.

### Step 3: Obtain the Project

Clone with Git:

```cmd
git clone https://github.com/iwebslogtech/PeaBox-Speech-Studio.git
cd PeaBox-Speech-Studio
```

Alternatively, choose **Code > Download ZIP** on GitHub and extract it.

### Step 4: Remove an Old Environment

If `.venv` was previously created with Python 3.13:

```cmd
rmdir /s /q .venv
```

### Step 5: Start the Application

Double-click:

```text
start_windows.bat
```

The launcher should:

1. Detect Python 3.12.
2. Create `.venv` using Python 3.12.
3. Activate the environment.
4. Upgrade pip, setuptools, and wheel.
5. Install `requirements.txt`.
6. Start Streamlit.

Open:

```text
http://localhost:8501
```

### Recommended Windows Launcher

Save the following as `start_windows.bat`:

```bat
@echo off
setlocal EnableDelayedExpansion
title PeaBox Speech Studio

cd /d "%~dp0"

echo.
echo ==========================================
echo        PeaBox Speech Studio Launcher
echo ==========================================
echo.

where py >nul 2>nul
if errorlevel 1 (
    echo ERROR: Python Launcher was not found.
    echo Install 64-bit Python 3.12 and enable Python Launcher.
    pause
    exit /b 1
)

py -3.12 --version >nul 2>nul
if errorlevel 1 (
    echo ERROR: Python 3.12 was not found.
    echo Install it with:
    echo winget install --exact --id Python.Python.3.12
    pause
    exit /b 1
)

echo Python 3.12 detected:
py -3.12 --version

if exist ".venv\Scripts\python.exe" (
    for /f "tokens=2" %%V in ('".venv\Scripts\python.exe" --version 2^>^&1') do set "VENV_VERSION=%%V"
    echo Existing environment: Python !VENV_VERSION!
    echo !VENV_VERSION! | findstr /b "3.12" >nul
    if errorlevel 1 (
        echo Removing incompatible virtual environment...
        rmdir /s /q ".venv"
    )
)

if not exist ".venv\Scripts\python.exe" (
    echo Creating Python 3.12 virtual environment...
    py -3.12 -m venv ".venv"
    if errorlevel 1 (
        echo ERROR: Virtual environment creation failed.
        pause
        exit /b 1
    )
)

call ".venv\Scripts\activate.bat"
if errorlevel 1 (
    echo ERROR: Could not activate the environment.
    pause
    exit /b 1
)

echo Active runtime:
python --version
where python

python -m pip install --upgrade pip setuptools wheel
if errorlevel 1 (
    echo ERROR: pip upgrade failed.
    pause
    exit /b 1
)

python -m pip install -r requirements.txt
if errorlevel 1 (
    echo ERROR: Package installation failed.
    pause
    exit /b 1
)

python -m streamlit run app.py
if errorlevel 1 (
    echo ERROR: Application startup failed.
    pause
    exit /b 1
)

pause
```

## 3. Windows Manual Installation

Use this if the launcher fails:

```cmd
cd C:\path\to\PeaBox-Speech-Studio
rmdir /s /q .venv
py -3.12 -m venv .venv
call .venv\Scripts\activate.bat
python --version
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

The displayed Python version must begin with `3.12`.

## 4. Install FFmpeg on Windows

Install with Winget:

```cmd
winget install --exact --id Gyan.FFmpeg
```

Close and reopen Command Prompt, then verify:

```cmd
ffmpeg -version
```

If Windows cannot find FFmpeg, restart Windows or add the FFmpeg `bin` directory to the PATH environment variable.

## 5. Linux Installation

### Ubuntu or Debian

Update package information:

```bash
sudo apt update
```

Install system dependencies:

```bash
sudo apt install -y git curl ffmpeg espeak-ng build-essential python3.12 python3.12-venv
```

If the Linux release does not offer Python 3.12 in its standard repositories, install Python 3.12 using the distribution's supported method or use `pyenv`.

Clone and run:

```bash
git clone https://github.com/iwebslogtech/PeaBox-Speech-Studio.git
cd PeaBox-Speech-Studio

python3.12 -m venv .venv
source .venv/bin/activate
python --version
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

Open:

```text
http://localhost:8501
```

### Linux Launcher

Save as `start_linux.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

cd "$(dirname "$0")"

if ! command -v python3.12 >/dev/null 2>&1; then
    echo "Python 3.12 is required but was not found."
    exit 1
fi

if [ -x ".venv/bin/python" ]; then
    VENV_VERSION=$(".venv/bin/python" -c "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}')")
    if [ "$VENV_VERSION" != "3.12" ]; then
        echo "Removing incompatible Python $VENV_VERSION environment."
        rm -rf .venv
    fi
fi

if [ ! -x ".venv/bin/python" ]; then
    python3.12 -m venv .venv
fi

source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

Make it executable:

```bash
chmod +x start_linux.sh
./start_linux.sh
```

## 6. Kokoro Local TTS

Select **Kokoro local** in the application and generate a short test sentence.

On the first run, the application downloads the model and voice files into:

```text
models/
```

Use a short sentence for the initial test:

```text
Welcome to PeaBox Speech Studio.
```

If the first generation appears slow:

1. Check whether files are appearing in `models/`.
2. Confirm that internet access to GitHub is allowed.
3. Wait for the initial model download to complete.
4. Check free disk space.
5. Review the terminal for HTTP, ONNX, or phonemization errors.

Subsequent generations reuse the downloaded files.

## 7. Faster-Whisper Transcription

Recommended initial settings:

```text
Model: base
Language: Auto
Device: CPU
```

Start with a short WAV or MP3 file.

Larger models provide potentially better accuracy but require more RAM and take longer. Use `tiny`, `base`, or `small` on ordinary CPU-only computers.

If MP3 or video fails while WAV works, verify FFmpeg:

```cmd
ffmpeg -version
```

or on Linux:

```bash
ffmpeg -version
```

## 8. Microsoft Azure Speech Setup

1. Open https://portal.azure.com/.
2. Create a Speech resource.
3. Open the resource.
4. Open **Keys and Endpoint**.
5. Copy one resource key.
6. Note the region, such as `centralindia`.
7. Select Azure Speech in PeaBox Speech Studio.
8. Enter the key and region.
9. Select **Discover voices**.
10. Filter by locale, select a voice, and generate audio.

Do not save the key inside source files.

## 9. Amazon Polly Setup

1. Create or use an AWS account.
2. Create an IAM user or role with the minimum required Polly permissions.
3. Permit voice listing and synthesis.
4. Create an access key only when programmatic credentials are necessary.
5. Enter the access key ID, secret key, optional session token, and AWS region.
6. Select a Polly engine.
7. Select **Discover voices**.
8. Choose a compatible voice and generate audio.

Prefer temporary credentials or an AWS role for production use.

## 10. Google Cloud Text-to-Speech Setup

1. Create a Google Cloud project.
2. Enable the Cloud Text-to-Speech API.
3. Enable billing if required by Google Cloud.
4. Create a service account with the minimum required permissions.
5. Create a JSON key only if local service-account authentication is required.
6. Keep the JSON file outside the repository.
7. Upload the JSON file through the application.
8. Enter a language code such as `en-IN`.
9. Select **Discover voices**.
10. Choose a voice and generate audio.

Delete and rotate the key if it is accidentally committed or shared.

## 11. Updating from v1 to v2

Back up local outputs before replacing files.

Using Git:

```bash
git status
git pull origin main
```

If local source files were modified, commit or stash those changes first.

After updating dependencies, recreate the environment:

### Windows

```cmd
rmdir /s /q .venv
py -3.12 -m venv .venv
call .venv\Scripts\activate.bat
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

### Linux

```bash
rm -rf .venv
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

Do not commit `.venv`, downloaded models, credentials, or generated media.

## 12. Verification Checklist

After installation, verify:

- The terminal reports Python 3.12.
- Streamlit opens at `http://localhost:8501`.
- Kokoro generates a short WAV file.
- Faster-Whisper transcribes a short audio file.
- TXT download works.
- SRT download works.
- FFmpeg is detected.
- Cloud voice discovery works for configured providers.
- Generated files appear in `outputs/`.
- Model files appear in `models/`.

## 13. Common Errors

### `No suitable Python runtime found`

Python 3.12 is not visible to the Python Launcher.

```cmd
py -0p
py -3.12 --version
```

Reinstall Python 3.12 with the launcher option if necessary.

### `Could not find a version that satisfies misaki`

The environment is probably using Python 3.13.

```cmd
rmdir /s /q .venv
py -3.12 -m venv .venv
```

### `streamlit is not recognized`

Installation failed before Streamlit was installed, or the environment is not active.

```cmd
.venv\Scripts\python.exe -m pip install -r requirements.txt
.venv\Scripts\python.exe -m streamlit run app.py
```

### `WinError 32` during transcription

Use the updated v2 `app.py`, which closes the temporary file descriptor and retries cleanup. Ensure that an older `app.py` was not retained during the update.

### Cloud authentication error

Confirm the credential, region, API enablement, IAM permissions, system time, billing status, and network access.

### Port 8501 is already in use

Start on another port:

```cmd
python -m streamlit run app.py --server.port 8502
```

Linux:

```bash
python -m streamlit run app.py --server.port 8502
```

## 14. Uninstallation

Remove only the virtual environment:

### Windows

```cmd
rmdir /s /q .venv
```

### Linux

```bash
rm -rf .venv
```

To remove everything, delete the project folder after backing up required files from `outputs/`.
