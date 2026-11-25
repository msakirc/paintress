# Paintress — Image-generation caller

> **Standalone.** Split from the [KutAI](https://github.com/msakirc/kutai) monorepo (`packages/paintress/`), where it is developed. It does not need the KutAI app; its test suite passes with only its declared dependencies installed.

[English](#english) · [Türkçe](#türkçe)

<a id="english"></a>

## Purpose

Paintress is the image-side counterpart of HaLLederiz Kadir. Given the model Fatih
Hoca picked, it calls that provider, validates the bytes with `renoir`, writes the
PNG atomically into the requested directory and returns a result record. Providers:
Pollinations (public, no key), Hugging Face Inference (FLUX.1-schnell, needs
`HF_TOKEN`) and a local ComfyUI / A1111 server managed by `clair_obscur`.

**It does NOT** choose the model (Fatih Hoca does), start or stop the local image
server (`clair_obscur` does), retry, or make any LLM call. Errors come back as
`ImageResult.error`; `generate` does not raise for provider failures.

## Install

```bash
pip install "paintress @ git+https://github.com/msakirc/paintress.git"
```

Requires Python 3.10+. Sibling KutAI packages are installed from their own repositories.

## Quick start

Calls the public Pollinations endpoint; needs network access.

```python
import asyncio
from types import SimpleNamespace

import paintress
from paintress.types import ImageSpec

# Outside KutAI, any object with `.model.provider` (+ optional `.name`,
# `.api_base`) works as the pick; inside KutAI it is a Fatih Hoca Pick.
pick = SimpleNamespace(model=SimpleNamespace(provider="pollinations", name="flux"))


async def main():
    spec = ImageSpec(prompt="a lighthouse at dusk", out_dir="out", width=512, height=512)
    res = await paintress.generate(pick, spec)
    print(res.path if not res.error else res.error)


asyncio.run(main())
```

## Public API

```python
import paintress
from paintress.types import ImageSpec, ImageResult

spec = ImageSpec(prompt="a lighthouse at dusk", out_dir="out/",
                 width=1024, height=1024, seed=None, quality_tier="fast",
                 filename_hint="hero")
res = await paintress.generate(pick, spec)      # pick: fatih_hoca Pick for an image model
# -> ImageResult(path, provider, model, cost, latency, seed_used, error)
if res.error:
    ...   # "unknown_provider:<p>" | "provider_raised:<Exc>:<msg>"
          # | "quality_failure:<renoir reason>" | "path_escape"
```

The provider is chosen by `pick.model.provider` (`"pollinations"`, `"huggingface"`,
`"clair_obscur"`); its base URL comes from `pick.model.api_base` or `.endpoint`.

## Dependencies

- `renoir` (validation), `clair_obscur` (local server URL / availability), `httpx`.
- Env: `HF_TOKEN` (Hugging Face), `CLAIR_OBSCUR_EXE` / `CLAIR_OBSCUR_BACKEND`
  (`comfyui` default, or `a1111`) / `CLAIR_OBSCUR_MODEL` (the ComfyUI checkpoint name
  written into the workflow's `ckpt_name`).

## Gotchas

- Every provider caps the response at 25 MB.
- The output path is re-checked with `realpath` after joining; a filename hint that
  would escape `out_dir` yields `error="path_escape"` and nothing is written.
- The local provider's `available()` re-reads `CLAIR_OBSCUR_EXE` on every call (no
  stale singleton).

## Tests

```powershell
python -m pytest packages/paintress/tests
```

## In KutAI

Hüsam's image lane calls `paintress.generate()` with the image model [fatih_hoca](https://github.com/msakirc/fatih-hoca) picked. Source: [`packages/paintress/`](https://github.com/msakirc/kutai/tree/main/packages/paintress) in the [KutAI monorepo](https://github.com/msakirc/kutai), where this package is developed.

## Status and limitations

- Standalone: it does not import the KutAI app, and its tests pass with only its declared dependencies.
- Version 0.1.0, part of a personal research project. Developed and tested on Windows 11 with Python 3.10; other platforms are untested.

## License

MIT. See [LICENSE](https://github.com/msakirc/kutai/blob/main/LICENSE).

---
<a id="türkçe"></a>
## Türkçe

### Amaç

Paintress, HaLLederiz Kadir'in görüntü tarafındaki karşılığıdır. Fatih Hoca'nın
seçtiği modelin sağlayıcısını çağırır, baytları `renoir` ile doğrular, PNG'yi istenen
klasöre atomik olarak yazar ve bir sonuç kaydı döner. Sağlayıcılar: Pollinations
(herkese açık, anahtarsız), Hugging Face Inference (FLUX.1-schnell, `HF_TOKEN` gerekir)
ve `clair_obscur`'un yönettiği yerel ComfyUI / A1111 sunucusu.

Modeli **seçmez** (Fatih Hoca seçer), yerel görüntü sunucusunu başlatıp durdurmaz
(`clair_obscur` yapar), yeniden denemez ve LLM çağrısı yapmaz. Hatalar
`ImageResult.error` olarak döner; `generate` sağlayıcı hatalarında istisna fırlatmaz.

### Kurulum

```bash
pip install "paintress @ git+https://github.com/msakirc/paintress.git"
```

Python 3.10+ gerekir. Kardeş KutAI paketleri kendi depolarından kurulur.

### Hızlı başlangıç

Herkese açık Pollinations uç noktasını çağırır; ağ erişimi gerekir.

```python
import asyncio
from types import SimpleNamespace

import paintress
from paintress.types import ImageSpec

# Outside KutAI, any object with `.model.provider` (+ optional `.name`,
# `.api_base`) works as the pick; inside KutAI it is a Fatih Hoca Pick.
pick = SimpleNamespace(model=SimpleNamespace(provider="pollinations", name="flux"))


async def main():
    spec = ImageSpec(prompt="a lighthouse at dusk", out_dir="out", width=512, height=512)
    res = await paintress.generate(pick, spec)
    print(res.path if not res.error else res.error)


asyncio.run(main())
```

### Genel API

```python
import paintress
from paintress.types import ImageSpec

spec = ImageSpec(prompt="alacakaranlıkta deniz feneri", out_dir="out/")
res = await paintress.generate(pick, spec)      # -> ImageResult(path, provider, ..., error)
```

### Testler

```powershell
python -m pytest packages/paintress/tests
```

### KutAI içindeki yeri

Hüsam'ın görüntü hattı, [fatih_hoca](https://github.com/msakirc/fatih-hoca)'nın seçtiği görüntü modeliyle `paintress.generate()` çağırır. Kaynak: [KutAI monorepo](https://github.com/msakirc/kutai) içinde [`packages/paintress/`](https://github.com/msakirc/kutai/tree/main/packages/paintress); paket orada geliştirilir.

### Durum ve sınırlamalar

- Bağımsız: KutAI uygulamasını import etmez; testleri yalnızca bildirilen bağımlılıklarla geçer.
- Sürüm 0.1.0, kişisel bir araştırma projesinin parçası. Windows 11 ve Python 3.10 üzerinde geliştirilip test edildi; diğer platformlar denenmedi.

### Lisans

MIT. [LICENSE](https://github.com/msakirc/kutai/blob/main/LICENSE) dosyasına bakın.
