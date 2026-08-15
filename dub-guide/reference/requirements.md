# Требования и подготовка окружения

## Операционная система

| OS | Поддержка | Примечание |
|---|-----------|-----------|
| Windows 10/11 | ✅ Полная | Рекомендуется для первого дубляжа |
| macOS | ✅ Полная | На M1/M2 может быть медленнее |
| Linux | ✅ Полная | На сервере может быть ограничение по памяти |

## Системные требования

### Минимум
- **ОЗУ:** 4 ГБ (для коротких видео)
- **Место на диске:** 5 ГБ свободного (промежуточные файлы)
- **Интернет:** требуется при первом запуске (скачивание модели demucs)

### Рекомендуемо
- **ОЗУ:** 8+ ГБ (для видео > 10 минут)
- **Видеокарта:** NVIDIA (для ускорения demucs)
- **Место на диске:** 20+ ГБ

### Серверные ограничения (Linux без видеокарты)
- **Видео < 5 минут:** 5-15 минут обработки
- **Видео 5-30 минут:** 1-3 часа обработки
- **Видео > 30 минут:** не рекомендуется (может упасть по памяти)

**Решение:** раздели видео на части по 5 минут, обработай отдельно, потом склей.

## Зависимости Python

### 1. Синтез речи (edge-tts)
```bash
pip install edge-tts
```
**Размер:** ~10 МБ  
**Зависит от:** requests

### 2. Отделение музыки (demucs)
```bash
pip install demucs torch torchaudio
```
**Размер:** ~1 ГБ (torch для CPU)  
**На видеокарте:** еще +500 МБ (CUDA)  
**Скорость:** на CPU медленнее в 10-100 раз, чем на GPU

### 3. Распознавание речи (faster-whisper)
```bash
pip install faster-whisper
```
**Размер:** модель `small` = 140 МБ, `medium` = 480 МБ  
**Опционально:** нужна только если нет готового транскрипта

### Проверка всех зависимостей
```bash
python -m edge_tts --help
python -m demucs --help
python -m faster_whisper --help
ffprobe -version
ffmpeg -version
```

Если какая-то команда не найдена → установи её пакет.

## FFmpeg и FFprobe

### Установка

**Windows (через WinGet):**
```bash
winget install ffmpeg
```

**macOS (через Homebrew):**
```bash
brew install ffmpeg
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get install ffmpeg
```

### Проверка
```bash
ffmpeg -version
ffprobe -version
```

## Шрифты для субтитров

### Windows
- **Для кириллицы/латиницы:** Arial (встроенный) ✅
- **Для китайского/японского:** Microsoft YaHei (встроенный) ✅

### macOS
- **Для кириллицы/латиницы:** Arial или Helvetica ✅
- **Для китайского:** PingFang SC (встроенный) ✅

### Linux (сервер)
- **Для кириллицы/латиницы:** DejaVu Sans (встроенный) ✅
- **Для китайского:** недостаточно (нужна установка)
  ```bash
  sudo apt-get install fonts-wqy-zenhei  # Для китайского
  ```

**⚠️ На сервере без CJK-шрифтов:** собирай китайские субтитры только на Windows/macOS.

## Видеокарта (необязательно, но ускоряет)

### Если есть NVIDIA
```bash
pip install nvidia-cublas-cu12 nvidia-cudnn-cu12
```
Ускорение demucs: **5-20 раз** (в зависимости от модели видеокарты).

### Если есть AMD (ROCm)
```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm5.7
```

### Если Apple Silicon (M1/M2)
```bash
pip install torch torchvision torchaudio
```
PyTorch автоматически использует Metal для ускорения.

## Проверка всего окружения (скрипт)

Сохрани как `check_env.py` и запусти:
```python
#!/usr/bin/env python3
import sys
import subprocess

checks = [
    ("edge-tts", "python -m edge_tts --help"),
    ("demucs", "python -m demucs --help"),
    ("ffmpeg", "ffmpeg -version"),
    ("ffprobe", "ffprobe -version"),
]

optional = [
    ("faster-whisper", "python -m faster_whisper --help"),
    ("NVIDIA CUDA", "python -c 'import torch; print(torch.cuda.is_available())'"),
]

print("=" * 50)
print("ТРЕБУЕМЫЕ ЗАВИСИМОСТИ")
print("=" * 50)

for name, cmd in checks:
    try:
        subprocess.run(cmd.split(), capture_output=True, check=True)
        print(f"✅ {name}")
    except:
        print(f"❌ {name} — ТРЕБУЕТСЯ УСТАНОВКА")

print("\n" + "=" * 50)
print("ОПЦИОНАЛЬНЫЕ")
print("=" * 50)

for name, cmd in optional:
    try:
        subprocess.run(cmd.split(), capture_output=True, check=True)
        print(f"✅ {name}")
    except:
        print(f"⚠️  {name} — опционально")
```

Запуск:
```bash
python check_env.py
```

## Быстрая установка (все сразу)

```bash
pip install edge-tts demucs faster-whisper torch torchaudio
```

На Linux может потребоваться `sudo apt-get install ffmpeg` предварительно.

## Если что-то не работает

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `ModuleNotFoundError: No module named 'edge_tts'` | Не установлен пакет | `pip install edge-tts` |
| `ModuleNotFoundError: No module named 'demucs'` | Не установлен пакет | `pip install demucs` |
| `ffmpeg: command not found` | ffmpeg не в PATH | Переустанови ffmpeg или добавь в PATH |
| `RuntimeError: CUDA not available` | Видеокарта не найдена (нормально) | Будет использована CPU (медленнее) |
| `MemoryError` при demucs на большом видео | Недостаточно оперативной памяти | Раздели видео на куски (шаг 8) |
