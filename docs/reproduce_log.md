# ODE-GS Reproduction Log

## Goal

Reproduce ODE-GS and record the environment, commands, errors, and fixes.

## Repository

- Upstream repo: https://github.com/preacherwhite/ODE-GS
- My fork: https://github.com/iisfive/ODE-GS
- Current branch: repro/baseline-setup

## Current Status

- [x] Repository cloned
- [x] SSH key configured
- [x] Git remote configured
- [ ] Environment checked
- [ ] CUDA extensions checked
- [ ] Dataset prepared
- [ ] Interpolation smoke test
- [ ] Extrapolation smoke test
- [ ] Evaluation smoke test

## Commands and Notes

Record every command, error, fix, and result here.

## Problems

None yet.

## Next Step

Run environment and CUDA extension checks.

## Stage 0 Environment Check

### Command

```bash
(odegs) ➜ ODE-GS python - <<'PY'
import torch
print("torch:", torch.__version__)
print("cuda version:", torch.version.cuda)
print("cuda available:", torch.cuda.is_available())
if torch.cuda.is_available():
    print("gpu:", torch.cuda.get_device_name(0))
PY
torch: 2.0.0+cu118
cuda version: 11.8
cuda available: True
gpu: NVIDIA GeForce RTX 4090
(odegs) ➜ ODE-GS python - <<'PY'
for name in ["diff_gaussian_rasterization", "simple_knn._C"]:
    try:
        __import__(name)
        print(name, "OK")
    except Exception as e:
        print(name, "FAIL")
        print(e)
PY
diff_gaussian_rasterization OK
simple_knn._C OK
```
