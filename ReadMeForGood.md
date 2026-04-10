# ST3GG Complete Command Reference

## 🦕 STEGOSAURUS WRECKS - Full CLI Documentation

This document details every command available in the ST3GG steganography toolkit, with options, flags, and use cases.

---

## Table of Contents

1. [Main Commands](#main-commands)
2. [Encode Command](#encode-command)
3. [Decode Command](#decode-command)
4. [Analyze Command](#analyze-command)
5. [Inject Commands](#inject-commands)
6. [Info Command](#info-command)
7. [Channel Presets](#channel-presets)
8. [Encoding Strategies](#encoding-strategies)
9. [Examples & Workflows](#examples--workflows)

---

## Main Commands

### Help & Usage

```bash
stegg --help              # Show main help menu
stegg -h                  # Short version
stegg [COMMAND] --help    # Get help for specific command
```

---

## Encode Command

### Purpose
Hide secret data (text, files, or jailbreak templates) inside an image using LSB steganography with optional encryption.

### Basic Syntax
```bash
stegg encode-cmd [OPTIONS]
```

### Required Options

| Flag | Short | Type | Description |
|------|-------|------|-------------|
| `--input` | `-i` | Path | **[REQUIRED]** Path to carrier image (PNG, JPEG, etc.) |

### Data Options (choose one)

| Flag | Short | Type | Description |
|------|-------|------|-------------|
| `--text` | `-t` | String | Text message to encode |
| `--file` | `-f` | Path | File to encode (binary or text) |
| `--template` | — | String | Pre-built jailbreak template name |

### Output Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--output` | `-o` | Path | `steg_[input].png` | Output image path |
| `--inject-name` | `-j` | Bool | False | Generate prompt injection filename |

### Channel Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--channels` | `-c` | Preset | `RGB` | Color channels to use (see presets below) |
| `--bits` | `-b` | Int (1-8) | `1` | Bits per channel (higher = more capacity but less stealthy) |

### Strategy Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--strategy` | `-s` | String | `interleaved` | Data placement strategy (sequential, interleaved, spread, randomized) |
| `--seed` | — | Int | None | Random seed for reproducible randomized encoding |

### Security Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--password` | `-p` | String | None | Encrypt payload with AES-256-GCM or XOR |
| `--no-compress` | — | Bool | False | Disable compression (enabled by default) |

### Display Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--quiet` | `-q` | Bool | False | Minimal output (no banner/stats) |

### Examples

```bash
# Basic text encoding
stegg encode-cmd -i photo.png -t "secret message" -o hidden.png

# Encode file with encryption
stegg encode-cmd -i photo.png -f secret.txt -p "mypassword" -o hidden.png

# High capacity (RGBA, 2 bits/channel = ~4MB)
stegg encode-cmd -i photo.png -t "data" -c RGBA -b 2 -o hidden.png

# Use jailbreak template with injection filename
stegg encode-cmd -i photo.png --template pliny_classic -j -o output.png

# Spread strategy (better stealth)
stegg encode-cmd -i photo.png -t "message" -s spread -o hidden.png

# Randomized strategy (strongest evasion)
stegg encode-cmd -i photo.png -t "message" -s randomized --seed 42 -o hidden.png

# Only Green channel, maximum stealth
stegg encode-cmd -i photo.png -t "msg" -c G -b 1 -o hidden.png

# Encrypt + randomized + high capacity
stegg encode-cmd -i photo.png -t "secret" -c RGBA -b 3 -s randomized -p "pass123" -o hidden.png
```

### Use Cases

- **Privacy**: Hide confidential messages in innocent photos
- **Data exfiltration**: Embed research data in image files
- **Stealth communication**: Use LSB encoding to pass through undetected
- **Watermarking**: Embed provenance or tracking info in images
- **CTF challenges**: Hide flags or clues inside images

---

## Decode Command

### Purpose
Extract hidden data from steganographic images, with auto-detection of encoding configuration.

### Basic Syntax
```bash
stegg decode-cmd [OPTIONS]
```

### Required Options

| Flag | Short | Type | Description |
|------|-------|------|-------------|
| `--input` | `-i` | Path | **[REQUIRED]** Path to encoded image |

### Detection Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--auto` / `--no-auto` | `-a` | Bool | True | Auto-detect encoding config from STEG header |

### Manual Config (if `--no-auto` used)

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--channels` | `-c` | Preset | `RGB` | Channel preset to use |
| `--bits` | `-b` | Int (1-8) | `1` | Bits per channel |
| `--strategy` | `-s` | String | `interleaved` | Encoding strategy |
| `--seed` | — | Int | None | Random seed (for randomized strategy) |

### Security Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--password` | `-p` | String | None | Decryption password |
| `--no-verify` | — | Bool | False | Skip CRC32 checksum verification |

### Output Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--output` | `-o` | Path | None | Save extracted data to file (if not set, displays in console) |
| `--raw` | — | Bool | False | Output as raw hex bytes |

### Display Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--quiet` | `-q` | Bool | False | Minimal output |

### Examples

```bash
# Auto-detect and display
stegg decode-cmd -i hidden.png

# Auto-detect and save to file
stegg decode-cmd -i hidden.png -o extracted.txt

# Manual config (if header missing)
stegg decode-cmd -i hidden.png --no-auto -c RGBA -b 2

# Decrypt with password
stegg decode-cmd -i hidden.png -p "mypassword"

# With manual config + password
stegg decode-cmd -i hidden.png --no-auto -c RGB -b 1 -p "secret123" -o data.bin

# Raw hex output
stegg decode-cmd -i hidden.png --raw

# Quiet mode (no banner)
stegg decode-cmd -i hidden.png -q
```

### Use Cases

- **Discovery**: Find hidden data in suspicious images
- **Data extraction**: Recover encoded messages and files
- **Forensics**: Extract evidence from digital media
- **Verification**: Confirm image authenticity and metadata
- **Testing**: Verify encoding/decoding works correctly

---

## Analyze Command

### Purpose
Analyze an image for statistical indicators of steganographic content. Detects LSB anomalies, calculates capacity estimates, and provides a verdict on whether data is likely hidden.

### Basic Syntax
```bash
stegg analyze [IMAGE_PATH] [OPTIONS]
```

### Arguments

| Argument | Type | Description |
|----------|------|-------------|
| `IMAGE_PATH` | Path | **[REQUIRED]** Path to image to analyze |

### Options

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--full` | `-f` | Bool | False | Full analysis with all channels and techniques |

### Analysis Performed

1. **Image Information**
   - Dimensions
   - Pixel count
   - Color mode (RGB, RGBA, etc.)
   - File format

2. **Channel Analysis**
   - Mean pixel value per channel
   - Standard deviation
   - LSB ratio (0s vs 1s)
   - Chi-square anomaly indicator

3. **Capacity Estimates**
   - Capacity for each channel preset
   - Capacity for different bit depths
   - Total usable space

4. **Verdict**
   - **✓ No obvious indicators**: Natural LSB distribution
   - **⚠ Possible hidden data**: Slight LSB anomaly detected
   - **⚠ HIGH PROBABILITY**: Strong evidence of steganography

### Examples

```bash
# Basic analysis
stegg analyze photo.png

# Full analysis with detailed channel breakdown
stegg analyze suspicious.png --full

# Analyze JPEG
stegg analyze image.jpg

# Analyze with full report
stegg analyze -f captured_data.png
```

### Use Cases

- **Forensics**: Detect if an image contains hidden data
- **Security**: Screen images for data exfiltration attempts
- **Research**: Study LSB distribution patterns
- **DLP testing**: Validate detection capabilities
- **Verification**: Confirm image integrity

### Verdict Interpretation

| Indicator | Meaning | Likelihood |
|-----------|---------|-----------|
| < 0.1 | Normal LSB distribution | ✓ Unlikely hidden data |
| 0.1 - 0.3 | Slight anomaly | ⚠ Possible |
| > 0.3 | High anomaly | ⚠ Probable hidden data |

---

## Inject Commands

### Purpose
Generate prompt injection filenames, view jailbreak templates, and obfuscate text for AI manipulation attacks.

### Inject Subcommand Structure
```bash
stegg inject [SUBCOMMAND] [OPTIONS]
```

---

### inject filename

**Purpose**: Generate filenames that inject instructions into AI systems

**Syntax**:
```bash
stegg inject filename [OPTIONS]
```

**Options**:

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--template` | `-t` | String | `chatgpt_decoder` | Filename template to use |
| `--channels` | `-c` | String | `RGB` | Channel string to embed in filename |
| `--count` | `-n` | Int | `1` | Number of filenames to generate |

**Available Templates**:
- `chatgpt_decoder` - Instructions for ChatGPT to decode LSB
- `claude_decoder` - Instructions for Claude
- `gemini_decoder` - Instructions for Gemini
- `universal_decoder` - Generic decoder instructions
- `system_override` - System prompt override attempt
- `roleplay_trigger` - Roleplay-based injection
- `dev_mode` - Developer mode activation
- `subtle` - Minimal filename
- `custom` - User-defined template

**Examples**:

```bash
# Generate ChatGPT decoder filename
stegg inject filename -t chatgpt_decoder -c RGB

# Generate Claude decoder filename
stegg inject filename -t claude_decoder -c RGBA

# Generate 5 random variants
stegg inject filename -t chatgpt_decoder -n 5

# Generate custom (uses random padding)
stegg inject filename -t subtle -c RGB
```

---

### inject templates

**Purpose**: List all available jailbreak templates

**Syntax**:
```bash
stegg inject templates
```

**Output**: Shows all template names and descriptions

**Examples**:

```bash
stegg inject templates
```

---

### inject show

**Purpose**: Display full content of a jailbreak template

**Syntax**:
```bash
stegg inject show [TEMPLATE_NAME]
```

**Arguments**:

| Argument | Type | Description |
|----------|------|-------------|
| `TEMPLATE_NAME` | String | **[REQUIRED]** Name of template to display |

**Available Templates**:
- `pliny_classic` - Classic Plinian jailbreak
- `dan_classic` - DAN (Do Anything Now) jailbreak
- `developer_mode` - Developer Mode jailbreak
- `system_prompt_leak` - System prompt extraction
- `grandma_exploit` - Grandma/roleplay jailbreak
- `translation_bypass` - Translation bypass technique
- `roleplay_master` - Roleplay master template
- `token_smuggle` - Token smuggling with Unicode
- `empty` - Empty template

**Examples**:

```bash
# Show Pliny template
stegg inject show pliny_classic

# Show DAN template
stegg inject show dan_classic

# Show system prompt leak template
stegg inject show system_prompt_leak
```

---

### inject zalgo

**Purpose**: Convert text to Zalgo (Unicode combining characters) for obfuscation

**Syntax**:
```bash
stegg inject zalgo [TEXT] [OPTIONS]
```

**Arguments**:

| Argument | Type | Description |
|----------|------|-------------|
| `TEXT` | String | **[REQUIRED]** Text to convert |

**Options**:

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--intensity` | `-i` | Int (1-5) | `3` | Zalgo intensity level |

**Examples**:

```bash
# Convert with default intensity
stegg inject zalgo "HELLO WORLD"

# High intensity (very garbled)
stegg inject zalgo "SECRET MESSAGE" -i 5

# Low intensity (slight corruption)
stegg inject zalgo "TESTING" -i 1
```

---

### inject leet

**Purpose**: Convert text to leetspeak/1337 speak for obfuscation

**Syntax**:
```bash
stegg inject leet [TEXT] [OPTIONS]
```

**Arguments**:

| Argument | Type | Description |
|----------|------|-------------|
| `TEXT` | String | **[REQUIRED]** Text to convert |

**Options**:

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--intensity` | `-i` | Int (1-3) | `2` | Leet intensity level |

**Examples**:

```bash
# Convert with default intensity
stegg inject leet "HELLO WORLD"

# Maximum intensity
stegg inject leet "TESTING" -i 3

# Low intensity
stegg inject leet "SECRET" -i 1
```

---

## Info Command

### Purpose
Display system information, cryptography status, and toolkit capabilities.

### Basic Syntax
```bash
stegg info
```

### Display Information

1. **Cryptography Status**
   - AES-256-GCM availability
   - Fallback XOR availability
   - Recommended encryption method

2. **Available Channel Presets**
   - All 15 channel combinations

3. **Version Information**
   - Toolkit version
   - Tagline

### Examples

```bash
stegg info
```

---

## Channel Presets

### Overview

Channel presets determine which color channels are used for data embedding. More channels = higher capacity but potentially lower stealth.

### Available Presets

| Preset | Channels | Stealth | Capacity | Best For |
|--------|----------|---------|----------|----------|
| `R` | Red only | ⭐⭐⭐⭐⭐ Excellent | Very Low | Maximum invisibility |
| `G` | Green only | ⭐⭐⭐⭐⭐ Optimal | Very Low | Least detectable (human eye) |
| `B` | Blue only | ⭐⭐⭐⭐ High | Very Low | Max invisibility |
| `RG` | Red + Green | ⭐⭐⭐⭐ High | Low | Balanced stealth |
| `RB` | Red + Blue | ⭐⭐⭐ Medium | Low | Alternative approach |
| `GB` | Green + Blue | ⭐⭐⭐ Medium | Low | Alternative approach |
| `RGB` | Red + Green + Blue | ⭐⭐⭐ Medium | **Medium** | Default, balanced |
| `A` | Alpha only | ⭐⭐⭐⭐ High | Varies | PNG transparency |
| `RA` | Red + Alpha | ⭐⭐⭐ Medium | Medium | Extra capacity |
| `GA` | Green + Alpha | ⭐⭐⭐ Medium | Medium | Extra capacity |
| `BA` | Blue + Alpha | ⭐⭐⭐ Medium | Medium | Extra capacity |
| `RGA` | Red + Green + Alpha | ⭐⭐ Lower | High | Balanced capacity |
| `RBA` | Red + Blue + Alpha | ⭐⭐ Lower | High | Balanced capacity |
| `GBA` | Green + Blue + Alpha | ⭐⭐ Lower | High | Balanced capacity |
| `RGBA` | All channels | ⭐⭐ Lowest | **High (4MB+)** | Maximum capacity |

### Capacity Calculator

```
Capacity (bytes) = (Width × Height × NumChannels × BitsPerChannel) / 8
```

### Examples

```bash
# 1920×1080 image, RGB preset, 1 bit/channel
Capacity = (1920 × 1080 × 3 × 1) / 8 = ~770 KB

# 1920×1080 image, RGBA preset, 2 bits/channel
Capacity = (1920 × 1080 × 4 × 2) / 8 = ~2 MB

# 1920×1080 image, RGBA preset, 4 bits/channel
Capacity = (1920 × 1080 × 4 × 4) / 8 = ~4 MB
```

---

## Encoding Strategies

### Overview

Encoding strategy determines how data is distributed across pixels. Different strategies provide different trade-offs between speed, capacity, and detection resistance.

### Available Strategies

| Strategy | Algorithm | Speed | Detection Resistance | Use Case |
|----------|-----------|-------|---------------------|----------|
| `sequential` | Fill pixels in order (0,0), (0,1), (0,2)... | ⚡ Fastest | Low | Simple encoding, CTF |
| `interleaved` | Alternate pixels (0,0), (0,2), (0,4)... | ⚡ Fast | Medium | **Default, balanced** |
| `spread` | Distribute evenly across entire image | ⚡⚡ Medium | High | Better stealth, forensics |
| `randomized` | Pseudo-random order (seeded PRNG) | ⚡⚡ Medium | Very High | **Strongest evasion** |

### Recommendations

- **`sequential`**: Quick testing, CTF challenges
- **`interleaved`**: Default, good balance
- **`spread`**: Forensics, better detection resistance
- **`randomized`**: Maximum stealth with `--seed` for reproducibility

### Examples

```bash
# Sequential (fastest)
stegg encode-cmd -i photo.png -t "msg" -s sequential

# Interleaved (default)
stegg encode-cmd -i photo.png -t "msg" -s interleaved

# Spread (better stealth)
stegg encode-cmd -i photo.png -t "msg" -s spread

# Randomized (strongest evasion)
stegg encode-cmd -i photo.png -t "msg" -s randomized --seed 42
```

---

## Examples & Workflows

### Workflow 1: Basic Secret Communication

```bash
# Step 1: Encode a message
stegg encode-cmd -i carrier.png -t "Meet at location X" -o message.png

# Step 2: Send message.png (looks like innocent photo)

# Step 3: Recipient decodes
stegg decode-cmd -i message.png
# Output: "Meet at location X"
```

### Workflow 2: Encrypted File Transfer

```bash
# Step 1: Encode and encrypt a file
stegg encode-cmd -i photo.png -f secret_document.pdf -p "strong_password_123" -o hidden.png

# Step 2: Send hidden.png

# Step 3: Recipient decrypts and extracts
stegg decode-cmd -i hidden.png -p "strong_password_123" -o recovered_document.pdf
```

### Workflow 3: High Capacity Data Exfiltration

```bash
# Step 1: Check capacity
stegg analyze carrier.png

# Step 2: Encode large file with RGBA + 3 bits/channel
stegg encode-cmd -i carrier.png -f large_dataset.bin -c RGBA -b 3 -s randomized -p "secret" -o exfil.png

# Step 3: Send exfil.png (appears normal)

# Step 4: Extract
stegg decode-cmd -i exfil.png -p "secret" -o recovered_dataset.bin
```

### Workflow 4: Forensic Analysis

```bash
# Step 1: Analyze suspicious image
stegg analyze suspicious.png --full

# If verdict shows "HIGH PROBABILITY":

# Step 2: Try to decode (auto-detect)
stegg decode-cmd -i suspicious.png

# If auto-detect fails, try manual:
stegg decode-cmd -i suspicious.png --no-auto -c RGB -b 2

# If encrypted, try password list:
stegg decode-cmd -i suspicious.png --no-auto -c RGB -b 1 -p "password123"
```

### Workflow 5: Payload Injection (Research Only)

```bash
# Step 1: Generate injection filename
stegg inject filename -t claude_decoder -c RGB
# Output: mystical_image_12345_67890_ignore_the_image_and_...

# Step 2: Encode jailbreak template
stegg encode-cmd -i photo.png --template dan_classic -j -o output.png

# Step 3: Rename to injection filename (for social engineering)
# File name now contains instructions for AI to decode the image

# Step 4: Send to AI system (for authorized research/testing)
```

### Workflow 6: Multi-Layer Obfuscation

```bash
# Create highly obfuscated payload

# Step 1: Convert text to Zalgo
stegg inject zalgo "SECRET INSTRUCTIONS" -i 5
# Output: S̢̡̧̜̤̦̤̯E̵̡̧̡̗̤ C̵̖͖̥̦͎̰R̷̰͓̭̱̭E̷̛̖̣̱̞T̸̢̪͓̱...

# Step 2: Encode with randomized strategy + encryption
stegg encode-cmd -i photo.png -t "S̢̡̧̜̤̦̤̯E̵̡̧̡̗̤..." -c RGBA -b 3 -s randomized -p "key123"

# Step 3: In image metadata, add injection filename
```

---

## Common Errors & Fixes

| Error | Cause | Solution |
|-------|-------|----------|
| `Input image not found` | Wrong path | Check file exists, use absolute or relative path correctly |
| `Must provide --text, --file, or --template` | No payload specified | Add `-t`, `-f`, or `--template` option |
| `Payload too large!` | Data exceeds capacity | Use more channels (`-c RGBA`), increase bits (`-b 2-8`), or smaller payload |
| `Cryptography library not available` | Missing `cryptography` package | Run `pip install stegg[crypto]` |
| `Decryption failed` | Wrong password | Verify password matches encoding step |
| `Failed to load image` | Corrupted image | Try with different image file |

---

## Best Practices

### For Security

1. **Always encrypt sensitive data**: `-p "strong_password"`
2. **Use strong passwords**: 16+ characters, mixed case, symbols
3. **Combine techniques**: Use randomized strategy + encryption + higher bits
4. **Change carriers**: Don't reuse same image multiple times

### For Stealth

1. **Use Green channel** (`-c G`): Least detectable by human eye
2. **Use 1-bit depth** (`-b 1`): Minimal statistical anomaly
3. **Use spread strategy** (`-s spread`): Better distribution
4. **Avoid sequential** (`-s sequential`): Most obvious patterns

### For Capacity

1. **Use RGBA** (`-c RGBA`): 4x more space than RGB
2. **Increase bit depth** (`-b 2-8`): Higher capacity, lower stealth
3. **Use larger images**: 1920×1080 vs 256×256 = 30x more capacity
4. **No compression for binary**: `--no-compress` if encoding already-compressed files

---

## Environment Variables

None currently configured. All options use command-line flags.

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Error (input not found, invalid option, encoding failed, etc.) |

---

## Version Information

- **ST3GG Version**: 2.0 (v3.0 format core)
- **Python**: 3.9+
- **Dependencies**: PIL, typer, rich, cryptography (optional)

---

## License & Legal

**AGPL-3.0** — Free for researchers, CTF players, and open-source projects.

**Intended Use**: Authorized security research, CTF competitions, digital forensics education, privacy research.

**For Commercial Use**: Contact for commercial license.

---

## Support & Documentation

- **Official Site**: https://ste.gg
- **GitHub**: https://github.com/elder-plinius/st3gg
- **PyPI**: https://pypi.org/project/stegg/
- **Documentation**: examples/README.md

---

**Last Updated**: April 10, 2026

⊰•-•✧•-•-⦑ ST3GG ⦒-•-•✧•-•⊱
*every pixel has a story*
*you just can't see it*
