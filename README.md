
# NAM Training Config (Tone3000‑style Simple Setup)

This repo contains a minimal setup to train a Neural Amp Modeler (NAM) model from your own DI + amp recordings, using the `nam-full` command.

It assumes:

- You have a **DI track** and a **processed/amp track** recorded at the same time in your DAW.
- They are exported as two mono WAV files with the **same length** and **same sample rate**.
- They are already **aligned** (latency ≈ 0 samples), so we use `delay: 0`.

## Official Neural Amp Modeler resources

- **GitHub repository** (source code, issues, releases): <https://github.com/sdatkinson/neural-amp-modeler>
- **Documentation** : <https://neural-amp-modeler.readthedocs.io/en/latest/>

## 1. Requirements

- Python **3.10+**
- `pip` (Python package manager)
- Ideally Linux for GPU acceleration support

## 2. Create a Python virtual environment

It’s strongly recommended to use a virtual environment so NAM and its dependencies don’t mess with your global Python installation.

In the root of this repo, run:

```bash
python -m venv env
```

This creates a folder called `env/` which contains an isolated Python.

### Activate the environment

**Linux / macOS:**

```bash
source env/bin/activate
```

**Windows (PowerShell):**

```powershell
.\env\Scripts\Activate.ps1
```

**Windows (CMD):**

```bat
env\Scripts\activate.bat
```

You should now see `(env)` in front of your prompt.

To **deactivate** later:

```bash
deactivate
```

## 3. Install Neural Amp Modeler (NAM)

With the environment activated, install NAM from PyPI:

```bash
pip install neural-amp-modeler
```

## 4. Prepare your audio

Export these from your DAW:

- `input.wav` – your **DI track** (what you played into the amp)
- `output.wav` – the **amp/mic/FX output** recorded at the same time

Requirements:

- Same **sample rate**
- Same **bit depth**
- Same **number of samples**

Place both files in the same folder as this `README.md` and the JSON config files.

This setup assumes **latency ≈ 0** (tracks are sample‑aligned in your DAW). If you know there is latency, you can edit `data.json` and change the `"delay"` field to the number of samples.

## 5. Config files overview

This repo includes three JSON files:

- `data.json`
- `model.json`
- `learning.json`

They are set up in a **simple, “just use the whole file”** way:

- The entire audio file is used for **training**.
- Validation is disabled .
- `delay` is set to **0** samples.

If you change file names, remember to update `x_path` and `y_path` in `data.json`.

## 6. Run training with `nam-full`

With the virtual environment activated and your WAV files in place:

```bash
nam-full data.json model.json learning.json final
```

Explanation:

- `data.json` – data configuration (paths, delay, slicing)
- `model.json` – model definition
- `learning.json` – trainer / dataloader configuration
- `final` – output directory for checkpoints and logs

Training time depends on:

- CPU/GPU speed
- Length of your audio
- `max_epochs` settings in `learning.json`

## 7. Where is the trained model?

After `nam-full` finishes, look inside the `final/` folder. You’ll see:

- Logs
- A final exported NAM model

You can then load it in your VST plugin NAM compatible !

## 8. Using the GPU instead of the CPU

If you have a compatible GPU and a GPU-enabled PyTorch installed, you can tell NAM to use it.

In `learning.json`, change the `accelerator` field, for example:

```json
"trainer": {
"accelerator": "gpu",
"devices": 1,
"max_epochs": 100,
"num_sanity_val_steps": 0
}
```
To install the PyTorch dependencies according to your hardware, you can use this site: https://pytorch.org/get-started/locally/

Regarding compatibility, here is what I found:
- Nvidia [CUDA](https://developer.nvidia.com/cuda-gpus) compatible GPU
- Intel [Meteor Lake](https://www.intel.com/content/www/us/en/ark/products/codename/90353/products-formerly-meteor-lake.html) iGPU and above 
- Intel [Alchemist](https://www.intel.com/content/www/us/en/ark/products/codename/226095/products-formerly-alchemist.html) GPU
- AMD (only on linux): I'll let you check if your GPU supports ROCm 6.4

**You will certainly have fewer problems on Linux.**

If this still does not work, you can consult the official documentation: <https://neural-amp-modeler.readthedocs.io/en/latest/installation.html#trouble-using-the-gpu>

## 9. Adjusting latency later (optional)

If you discover you *do* have latency between DI and amp tracks:

1. Measure it in your DAW:
- Zoom into a pick transient on both tracks.
- Count the offset in **samples** (or convert from ms using `samples = ms * sample_rate / 1000`).

2. Edit `data.json`:

```json
"common": {
"x_path": "input.wav",
"y_path": "output.wav",
"delay": <your_latency_in_samples>
}
```

3. Run `nam-full` again.

It’s usually safer to **slightly overestimate** delay (e.g. use 90 if you measured ~80) than to underestimate, so the model never has to “predict the future.”

**However, if you have captured the input and output at the same time (guitar DI + amp loadbox in the same audio interface at the same time, for example), you can keep it at 0 for the best result.**
