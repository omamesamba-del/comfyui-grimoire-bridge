# comfyui-grimoire-bridge

Custom nodes for ComfyUI that let [grimoire](https://github.com/omamesamba-del/grimoire) send prompts and generation settings directly to your workflow.

## Installation

```bash
cd custom_nodes
git clone https://github.com/omamesamba-del/comfyui-grimoire-bridge.git
```

Restart ComfyUI. The nodes will appear under the **PromptBuilder** category.

## Setup in grimoire

1. Open **Settings → Generation**
2. Set **ComfyUI URL** (default: `http://127.0.0.1:8188`)
3. Set the **Slot Name** to match the slot name(s) in your workflow (e.g. `positive`)
4. Enable **Send Prompt** and/or **Send Gen Settings** as needed

## Nodes

### Grimoire Slot

A text slot controlled by grimoire. Place one node per text area in your workflow.

| Input | Type | Description |
|-------|------|-------------|
| `slot_name` | STRING | Name of the slot (e.g. `positive`, `negative`, `chara`) |
| `text` | STRING | Fallback text used when grimoire has not sent anything |

| Output | Type |
|--------|------|
| `text` | STRING |

Connect the output to any `STRING` input — typically a CLIP Text Encode node. When grimoire sends a prompt to the matching slot name, the node picks it up automatically on the next queue run.

**Example:** Add two Grimoire Slot nodes named `positive` and `negative`, connect each to a CLIP Text Encode node, then set grimoire to use the same slot names.

---

### Grimoire Join

Joins multiple text strings with a configurable separator. Empty inputs are skipped automatically.

| Input | Type | Description |
|-------|------|-------------|
| `separator` | STRING | Separator string (default: `, `) |
| `text_1` … | STRING | Click **+** on the node to add more inputs |

| Output | Type |
|--------|------|
| `text` | STRING |

Useful for combining a Grimoire Slot output with a fixed prefix or suffix.

---

### Grimoire Params

An all-in-one node that receives generation parameters from grimoire and loads the checkpoint. Replaces the separate CheckpointLoader + KSampler + EmptyLatentImage setup.

| Output | Type | Description |
|--------|------|-------------|
| `MODEL` | MODEL | Loaded model |
| `CLIP` | CLIP | CLIP encoder |
| `VAE` | VAE | VAE (external file or checkpoint-bundled) |
| `sampler_name` | COMBO | Sampler |
| `scheduler` | COMBO | Scheduler |
| `steps` | INT | Step count |
| `cfg` | FLOAT | CFG scale |
| `seed` | INT | Seed |
| `width` | INT | Width |
| `height` | INT | Height |
| `stop_at_clip_layer` | INT | CLIP skip |
| `batch_size` | INT | Batch size |
| `denoise` | FLOAT | Denoise strength |
| `upscale_by` | FLOAT | Hires upscale factor |

When grimoire sends generation settings, Grimoire Params updates automatically on the next queue run. Parameters not sent by grimoire fall back to the values set in the node itself.

## API Endpoints

These endpoints are added to ComfyUI's HTTP server by the extension:

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/pb/health` | Health check (returns version) |
| `GET` | `/pb/slots` | List currently registered slot names |
| `POST` | `/pb/set-slot` | Set a named slot's text (used by grimoire) |
| `GET` | `/pb/get-slot` | Read a named slot's current value |
| `POST` | `/pb/set-gen` | Set generation parameters (used by grimoire) |
| `GET` | `/pb/get-gen` | Read current generation parameters |
| `POST` | `/pb/register-slots` | Register slot names (called by grimoire on connect) |

## Notes

- The extension is passive — it does not interfere with the queue or generation pipeline
- Slots update on the **next queue run** (ComfyUI re-evaluates `IS_CHANGED` per execution)
- Works with any workflow; Grimoire Slot and Grimoire Params are independent and can be used separately

---

> **Note:** This extension was developed with the assistance of AI (Claude by Anthropic).

## License

MIT
