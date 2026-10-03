# CLAUDE.md

## Project

**pulpit-to-the-small-group** transcribes sermons and turns them into material that helps small-group (PG) leaders prepare their meetings. Sermons are in Brazilian Portuguese and typically 30–60 minutes long.

Pipeline:

1. **Whisper** transcribes the sermon audio.
2. **Gemma** organizes and summarizes the transcription for small-group leaders.

The project is open to Hacktoberfest contributions. Code is MIT licensed; the Whisper and Gemma models are not covered by that license (see README).

## Stack

Everything runs locally with Docker Compose (`docker-compose.yml`, project name `transcribe-sermons`):

| Service | Image | Role | Endpoint |
|---|---|---|---|
| `gemma` | `ollama/ollama:0.35.1` | Serves `gemma3:4b` via Ollama | `http://127.0.0.1:11434` |
| `gemma-pull` | `ollama/ollama:0.35.1` | One-shot job: pulls `gemma3:4b` once `gemma` is healthy, then exits | — |
| `whisper` | `onerahmet/openai-whisper-asr-webservice:v1.10.0` | faster-whisper, `large-v3-turbo`, int8 on CPU | `http://127.0.0.1:9000` (Swagger at `/docs`) |

Models persist in the named volumes `ollama-models` and `whisper-models`.

## Commands

```bash
docker compose up -d            # start everything (first run downloads ~5 GB of models)
docker compose logs -f gemma-pull   # follow the Gemma model download
docker compose config --quiet   # validate the compose file
docker compose down             # stop (models stay in the volumes)
```

Quick checks:

```bash
curl http://localhost:11434/api/generate -d '{"model":"gemma3:4b","prompt":"Olá","stream":false}'
curl -F "audio_file=@sermon.mp3" "http://localhost:9000/asr?language=pt&output=txt"
```

## Decisions and constraints

- **CPU-only.** The target machine has no NVIDIA GPU (AMD integrated graphics, ~15 GB RAM). Don't add CUDA/GPU images or `deploy.resources.reservations.devices` unless asked. Model sizes were chosen to fit both models in RAM at once.
- **Pinned image versions.** Never switch to `latest`; bump versions explicitly so contributors get the same environment.
- **Localhost-only ports.** The APIs have no authentication, so ports bind to `127.0.0.1`. Don't expose them on `0.0.0.0` without adding auth.
- **Named volumes** for models (faster than bind mounts on Windows/WSL, keeps the repo clean). Never commit model files.
- Always pass `language=pt` to Whisper; auto-detection wastes time and can misfire on Bible quotations.

## Repository

- Remote: `origin` → https://github.com/gerson-rjr/pulpit-to-the-small-group.git, default branch `main`.
- Commit messages in English.
