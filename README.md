# vdo-creator-action-workflow

Runs the weekly documentary pipeline from the private
[`SathiyarajS/video-creator`](https://github.com/SathiyarajS/video-creator) repository.
This repository holds only the workflow and its settings; the pipeline code stays
in `video-creator`.

## What a run does

1. Checks out `video-creator` (`main` by default) with a read-only deploy key.
2. Runs its test suite. A failing test stops the run before any paid call.
3. Runs the pipeline: script, narration, images, render, Short.
4. Uploads the YouTube-ready files to Azure Blob storage and sends the links by
   Telegram and email.

Nothing the run produces is stored on GitHub: no artifacts, no file listings in the
run summary.

## Triggers

- **Schedule:** Saturday 09:00 IST (03:30 UTC).
- **Manual:** Actions → *Weekly video* → *Run workflow*. Inputs: build the Short,
  an optional subject name, and which `video-creator` branch, tag or commit to run.

## Settings

**Secrets** (credentials and personal details):
`VIDEO_CREATOR_DEPLOY_KEY`, `OPENAI_API_KEY`, `POLLINATIONS_API_KEY`, `AZURE_TTS_KEY`,
`AZURE_STORAGE_ACCOUNT_NAME`, `AZURE_STORAGE_ACCOUNT_KEY`, `AZURE_STORAGE_CONTAINER_NAME`,
`AZURE_TABLE_CONNECTION_STRING`, `CONTACT_EMAIL`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`,
`SMTP_USER`, `SMTP_APP_PASSWORD`, `EMAIL_TO`.

**Variables** (settings that are not secret; an unset one uses the code default):
model choices (`SCRIPT_MODEL`, `UTILITY_MODEL`, `LEDGER_MODEL`, `ARC_MODEL`,
`NARRATION_MODEL`, `EDITORIAL_MODEL`), image settings (`IMAGE_PROVIDER`,
`OPENAI_IMAGE_MODEL`, `OPENAI_IMAGE_QUALITY`, `IMAGE_FALLBACK_MODELS`,
`THUMBNAIL_FALLBACK_MODELS`, `POLLINATIONS_MODEL`, `POLLINATIONS_REFERENCE_MODEL`,
`IMAGE_WORKERS`), voice (`AZURE_TTS_REGION`, `AZURE_TTS_VOICE`) and
`AZURE_STORAGE_ALLOW_PUBLIC`.

## Rotating the deploy key

Delete the `vdo-creator-action-workflow (read-only)` deploy key on `video-creator`,
add a new read-only key there, and store its private half as `VIDEO_CREATOR_DEPLOY_KEY`
here.
