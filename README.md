# PeaBox AI Speech Studio

PeaBox Speech Studio is a local-first Python application for AI speech generation, audio and video transcription, subtitle creation, and optional cloud text-to-speech services.

The application combines free local speech models with commercial cloud providers in one Streamlit interface. Users can generate narration with Kokoro, transcribe recordings with Faster-Whisper, discover cloud voices dynamically, export audio and subtitles, and estimate provider costs before generating speech.

## Highlights

- Local neural text-to-speech with Kokoro
- Local speech-to-text with Faster-Whisper
- Microsoft Azure Speech integration
- Amazon Polly integration
- Google Cloud Text-to-Speech integration
- Dynamic cloud voice discovery
- Multiple languages and voice options
- Audio and video transcription
- TXT transcript and SRT subtitle export
- Script character count and indicative cost estimates
- Local model and output storage
- Streamlit web interface
- Windows and Linux support

## Intended Users

PeaBox Speech Studio is designed for:

- Course creators and trainers
- YouTube and video creators
- Audiobook and podcast producers
- Consultants and business teams
- Educators and students
- Developers experimenting with speech APIs
- Users who prefer local processing and provider flexibility

## Current Capabilities

### Local Text-to-Speech

Kokoro provides local neural narration without per-character charges. The first generation downloads the required model and voice files. Later generations reuse the files stored in the local `models` directory.

Available controls include:

- Language selection
- Voice selection
- Speaking speed
- Output filename
- In-browser playback
- WAV download

### Cloud Text-to-Speech

#### Microsoft Azure Speech

- API-key and region-based connection
- Dynamic voice discovery
- Locale filtering
- Voice metadata display
- Speaking-rate adjustment
- Pitch adjustment
- MP3 generation and download

#### Amazon Polly

- AWS access-key authentication
- Optional temporary session token
- Configurable AWS region
- Standard, neural, generative, and long-form engines
- Dynamic compatible-voice discovery
- MP3 generation and download

#### Google Cloud Text-to-Speech

- Service-account JSON authentication
- Language-code filtering
- Dynamic voice discovery
- Voice sample-rate information
- MP3 generation and download

### Local Transcription

Faster-Whisper processes supported audio and video locally. The application uses CPU-compatible INT8 inference by default.

Supported input formats include:

- WAV
- MP3
- M4A
- FLAC
- OGG
- MP4
- MOV
- MKV
- WebM

Available transcription models include:

- tiny
- base
- small
- medium
- large-v3

Exports include:

- Plain-text transcript
- SRT subtitle file
- Timestamped segment table

The temporary-file workflow includes a Windows file-lock correction. The application closes the temporary file descriptor before Faster-Whisper opens the media and retries cleanup when Windows briefly retains a lock.

## Application Structure

```text
PeaBox-Speech-Studio/
├── app.py
├── requirements.txt
├── start_windows.bat
├── start_linux_mac.sh
├── README.md
├── INSTALLATION.md
├── LICENSE
├── .gitignore
├── data/
├── models/
└── outputs/
```

### Storage

```text
models/
```

Stores downloaded local speech models and voice data.

```text
outputs/
```

Stores generated WAV and MP3 files, transcripts, and subtitle files.

```text
data/
```

Reserved for local application data and future project history.

## Quick Start

### Windows

1. Install 64-bit Python 3.12.
2. Clone or download this repository.
3. Delete any `.venv` folder created with another Python version.
4. Run `start_windows.bat`.
5. Open `http://localhost:8501` if the browser does not open automatically.

### Linux

```bash
sudo apt update
sudo apt install -y python3.12 python3.12-venv ffmpeg espeak-ng git

git clone https://github.com/iwebslogtech/PeaBox-Speech-Studio.git
cd PeaBox-Speech-Studio

python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

See [INSTALLATION.md](INSTALLATION.md) for detailed Windows, Linux, FFmpeg, Kokoro, and cloud-provider setup.

## Cloud Provider Setup Links

### Microsoft Azure Speech

- Create a Speech resource: https://portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices
- Azure Speech documentation: https://learn.microsoft.com/azure/ai-services/speech-service/
- Azure Speech pricing: https://azure.microsoft.com/pricing/details/speech/

### Amazon Polly

- Create an AWS account: https://aws.amazon.com/free/
- AWS access-key guide: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html
- Amazon Polly console: https://console.aws.amazon.com/polly/
- Amazon Polly pricing: https://aws.amazon.com/polly/pricing/

### Google Cloud Text-to-Speech

- Create a Google Cloud project: https://console.cloud.google.com/projectcreate
- Enable the Text-to-Speech API: https://console.cloud.google.com/apis/library/texttospeech.googleapis.com
- Manage service accounts: https://console.cloud.google.com/iam-admin/serviceaccounts
- Google Cloud TTS pricing: https://cloud.google.com/text-to-speech/pricing

## Cost Estimates

The application calculates indicative provider costs from the number of characters in the script and the selected pricing family.

Displayed estimates are for planning only. Actual billing can vary because of:

- Provider free tiers
- Account credits
- Region
- Voice or model family
- Currency conversion
- Taxes
- Enterprise agreements
- Provider price changes

Always treat the provider invoice and current pricing page as authoritative.

## Security and Privacy

- Local Kokoro generation remains on the machine.
- Faster-Whisper transcription remains on the machine.
- Cloud speech text is sent to the selected cloud provider when a cloud provider is used.
- Credentials entered in the app should remain limited to the active Streamlit session.
- Credentials must never be committed to GitHub.
- Google service-account JSON files must not be copied into the repository.
- Use least-privilege IAM roles and rotate exposed credentials immediately.
- Do not expose the Streamlit port directly to the public internet without authentication and TLS.

## Responsible Use

Use only audio, voices, and recordings that you are authorized to process. Do not use PeaBox Speech Studio to impersonate individuals, mislead listeners, commit fraud, or create deceptive attribution.

Generated speech should be disclosed as synthetic when the context could otherwise mislead an audience.

## Troubleshooting

### The launcher uses Python 3.13 instead of 3.12

Delete the current environment and recreate it explicitly:

```cmd
rmdir /s /q .venv
py -3.12 -m venv .venv
```

### Misaki cannot be installed

Confirm that the environment is using Python 3.12:

```cmd
.venv\Scripts\python.exe --version
```

If the output shows Python 3.13, delete `.venv` and recreate it with `py -3.12`.

### Streamlit is not recognized

Run Streamlit through the active Python interpreter:

```cmd
.venv\Scripts\python.exe -m streamlit run app.py
```

### MP3 or video transcription fails

Install FFmpeg and reopen the terminal. Verify:

```cmd
ffmpeg -version
```

### Kokoro takes several minutes

The first run downloads the model and voice files. Check whether files are appearing in `models/`. Later generations should not repeat the download.

### Cloud voice discovery fails

Check:

- API credentials
- Provider region
- Enabled service or API
- IAM permissions
- Billing status
- Firewall and proxy settings

## Development

Create and activate a Python 3.12 environment, then install dependencies:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

On Windows:

```cmd
py -3.12 -m venv .venv
call .venv\Scripts\activate.bat
python -m pip install --upgrade pip setuptools wheel
python -m pip install -r requirements.txt
python -m streamlit run app.py
```

## Roadmap

Potential future enhancements include:

- Batch speech generation
- Project history and job queue
- DOCX, PDF, and PowerPoint narration
- Pronunciation dictionaries
- Multi-speaker scripts
- Audiobook chapter management
- VTT subtitle export
- Speaker diarization
- Translation-assisted dubbing
- Desktop packaging
- Docker deployment
- REST API
- Authenticated multi-user deployment

## Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Make focused changes.
4. Test on a clean Python 3.12 environment.
5. Do not include API keys, model files, generated media, or personal recordings.
6. Submit a pull request with a clear description and testing notes.

## License

This application is released under the MIT License unless the repository states otherwise. Third-party libraries, cloud services, and downloaded models retain their respective licences and terms.

## Project Links

- Repository: https://github.com/iwebslogtech/PeaBox-Speech-Studio
- Issues: https://github.com/iwebslogtech/PeaBox-Speech-Studio/issues
- Releases: https://github.com/iwebslogtech/PeaBox-Speech-Studio/releases
