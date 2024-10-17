---
title: ASCII Theater - AI-Powered ASCII Art Movie Generator
date: 2024-08-17
categories: [AI Projects, ASCII Art]
tags: [python, openai, ollama, ascii art, animation, cli]
description: ASCII Theater is a Python-based project that creates and displays animated ASCII art movies using AI-generated stories, offering a unique command-line cinematic experience.
---

![ASCII Theater Scene Window](https://raw.githubusercontent.com/jmcdice/ascii-movie-project/refs/heads/main/images/scene_window.png)
![ASCII Theater ASCII Screenshot](https://raw.githubusercontent.com/jmcdice/ascii-movie-project/refs/heads/main/images/ascii_screnshot.png)

# ASCII Theater

ASCII Theater is an innovative Python project that brings together AI-generated storytelling and ASCII art to create a unique animated movie experience in your command-line interface. This project showcases the creative potential of combining cutting-edge AI technology with the nostalgic charm of ASCII art.

## Key Features

- AI-powered story and scene generation using OpenAI or local LLM (Ollama)
- Conversion of story scenes into ASCII art frames
- Command-line ASCII art movie player with scene information display
- Customizable frame rate for playback
- Movie selection interface for playing existing movies

## How It Works

1. The project uses AI (either OpenAI or a local LLM via Ollama) to generate a story and scenes.
2. These scenes are then converted into ASCII art frames.
3. The ASCII art frames are played in sequence in the command-line interface, creating an animated movie effect.
4. Users can adjust playback settings like frame rate and choose from existing movies to watch.

## Getting Started

To get started with ASCII Theater, you'll need Python 3.7+ and either an OpenAI API key or Ollama installed for local LLM support.

1. Clone the repository:
   ```
   git clone https://github.com/jmcdice/ascii-movie-project.git
   cd ascii-movie-project
   ```

2. Set up a virtual environment and install dependencies:
   ```
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

3. Configure your LLM settings:
   ```
   cp .env_example .env.sh
   ```
   Then edit `.env.sh` with your API keys or Ollama settings.

## Usage

### Generating a New Movie

You can generate a new ASCII art movie using different AI providers:

```
python scripts/generate_movie.py --provider anthropic --topic "Space exploration"
python scripts/generate_movie.py --provider openai --topic "Underwater adventure"
python scripts/generate_movie.py --provider ollama --topic "Time travel"
```

### Playing a Movie

To watch an ASCII art movie:

```
python scripts/play_movie.py
```

You can adjust playback settings:

```
python scripts/play_movie.py --delay 0.5 --scene-delay 3
```

## Contributing

Contributions to ASCII Theater are welcome! Feel free to submit a Pull Request on the [GitHub repository](https://github.com/jmcdice/ascii-movie-project).

ASCII Theater demonstrates the creative possibilities at the intersection of AI, programming, and visual art. It's a perfect project for those interested in exploring AI-generated content, ASCII art, or unique ways of storytelling through code.
