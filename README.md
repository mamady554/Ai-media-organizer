y README
# ai-media-organizer

AI-powered media file organizer that uses a local Llama 3.2 model (via Ollama) to classify and sort movies, TV shows, and music into a clean Plex-compatible folder structure. No external APIs — runs entirely on your own hardware.

## How it works

1. Strips encoding junk from filenames (`1080p`, `BluRay`, `x265`, `YIFY`, release group tags, etc.)
2. Sends the cleaned filename to a local Llama 3.2 model running via Ollama
3. Parses the JSON response to classify the file as a movie, TV episode, or music track
4. Moves the file into a structured folder hierarchy

## Output structure

```
/mnt/media/
  Movies/
    Film Title (2023)/
      Film Title (2023).mkv

  TVShows/
    Show Name/
      Season 01/
        S01E04.mkv

  Music/
    Artist/
      Album/
        01 - Song Title.flac
```

## Requirements

- Python 3
- [Ollama](https://ollama.com) running locally with Llama 3.2 pulled
- `requests` library

```bash
pip3 install requests --break-system-packages
ollama pull llama3.2
```

## Install

**Option 1 — One-liner:**

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/mamady554/ai-media-organizer/main/install.sh)
```

**Option 2 — Clone and run:**

```bash
git clone https://github.com/mamady554/ai-media-organizer.git
cd ai-media-organizer
pip3 install requests --break-system-packages
python3 media_organizer.py --source /mnt/media/dump
```

## Usage

```bash
# Dry run — preview what would move (default)
python3 media_organizer.py --source /mnt/media/dump

# Apply moves
python3 media_organizer.py --source /mnt/media/dump --live

# Use a different Ollama model
python3 media_organizer.py --source /mnt/media/dump --model mistral --live

# Watch the log (if running via auto-watcher)
tail -f /var/log/media_organizer.log
```

## Auto-watch with inotify (optional)

Automatically trigger the organizer whenever new files land in the dump folder. Included in [proxmox-server-setup](https://github.com/mamady554/proxmox-server-setup) as a systemd service.

```bash
# Manual watcher setup
apt install inotify-tools
inotifywait -m -r -e close_write,moved_to /mnt/media/dump |
while read path action file; do
    sleep 5
    python3 /root/media_organizer.py --source /mnt/media/dump --live
done
```

## Stack

`Python 3` `Ollama` `Llama 3.2` `inotify-tools` `systemd`

## License

MIT
