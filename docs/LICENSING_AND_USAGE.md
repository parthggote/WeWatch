# WeWatch — Licensing & Permitted Use Guide

> **Status:** Internal engineering notes, **not legal advice**. Based on a read-through of the repository on 2026-10-05
> (LICENSE files, `package.json`, `requirements*.txt`, `SKILL.md` manifests). Have counsel confirm before any
> commercial/client **delivery**. Items marked ⚠️ were *not* fully verified and need a check against the upstream source.

WeWatch is an unmodified fork of **SharpAI/DeepCamera** (MIT). The repo is a mix of code under *different* licenses, so
"the repo is MIT" is true only for the code SharpAI wrote. Third-party code, bundled binaries, and ML models each carry
their own terms.

---

## 1. Quick answers

| Question | Answer |
|---|---|
| Can we run it for a **client demo**? | **Yes**, for the parts listed as ✅/🟡 below. A private demo you run yourselves is low-risk. Avoid the 🔴 parts. |
| Can we **ship it to the client** (hand over code / deploy on their infra)? | **Only after the 🟡 items are resolved** (mainly Ultralytics/YOLO and model weights) and the 🔴 items are removed. |
| Can we re-brand / sell it? | The MIT code, yes, **keeping the copyright + licence notice**. Not the Shinobi (`src/camera`) code. |
| Do we need to keep attribution? | Yes. Keep `LICENSE` and add a `THIRD_PARTY_NOTICES` file in any distribution. |

---

## 2. Component-by-component

Legend: ✅ safe for commercial use (with attribution) · 🟡 usable but with obligations/conditions · 🔴 do not use in a commercial client product · ⚠️ verify

### 2.1 First-party code (SharpAI) — ✅ MIT
| Path | Licence | Notes |
|---|---|---|
| `LICENSE` (repo root), `skills/**` scripts, `docs/`, `docker/` (SharpAI-written files) | MIT © SharpAI Dev Team | Use, modify, sublicense, sell. **Must keep copyright + MIT text.** |
| `skills/lib/env_config.py` (hardware detection) | MIT | Good to reuse. Duplicated in `skills/detection/yolo-detection-2026/scripts/`. |
| `src/fall_detection`, `src/laptop_monitor`, `src/yolov3_parking`, `src/yolov7_person_detector`, `src/yolov7_reid` (wrapper/compose files) | MIT | The **models/code they pull in** are separate (see 2.3). |
| `src/embedding`, `src/face_detection` (SharpAI parts) | MIT-covered by root LICENSE ⚠️ | Outdated stack (Flask 0.12 etc.); also see dataset/model notes in 2.3. |

### 2.2 Shinobi NVR fork — 🔴 `src/camera/src/`
- `src/camera/src/LICENSE` and `COPYING`: **Creative Commons BY-NC-SA 4.0** (NonCommercial, ShareAlike).
- `src/camera/src/package.json`: `"license": "GPLv3"`. The two statements conflict; either way it is **not**
  permissive.
- **NC = no commercial use.** Do **not** include, deploy, or build on this in a paid client project or a client's product.
- Shinobi's authors sell a commercial licence (see `src/camera/src/README.md`); that is the only legitimate route if you
  need it. Otherwise use a different NVR/ingest approach (e.g. go2rtc, FFmpeg, MediaMTX — check their licences too).
- Applies to everything under `src/camera/src/` including its bundled JS libs (`web/libs/js/*`).
- `docker/` contains Dockerfiles that build this camera container — treat those images as 🔴 as well.

### 2.3 Detection / ML dependencies — 🟡 (the important ones)

| Item | Where used | Licence (upstream) | Implication |
|---|---|---|---|
| **Ultralytics** (`ultralytics>=8.3.0`) | `skills/detection/yolo-detection-2026*`, requirements files | **AGPL-3.0** (or paid Ultralytics Enterprise licence) | 🟡 Fine for internal demo/R&D. If you **distribute** the software or offer it as a **network service** to a client, AGPL requires releasing the corresponding source of your whole combined work under AGPL — or buying the Enterprise licence. Decide this *before* delivery. |
| `yolo26n.onnx`, `yolo26n_names.json` (committed in `skills/detection/yolo-detection-2026/`) | YOLO skill | Ultralytics-derived weights → AGPL-3.0 ⚠️ | Same as above. |
| YOLOv7 (`src/yolov7_*`) | legacy | **GPL-3.0** (WongKinYiu) ⚠️ | 🔴/🟡 Strong copyleft; avoid in a closed-source deliverable. |
| YOLOv2/v3 / darknet (`src/yolov2_parking`, `yolov3_parking`) | legacy | darknet is public domain/YOLO licence; weights vary ⚠️ | Check weights provenance. |
| **Depth Anything V2** | `skills/transformation/depth-estimation` | **Small = Apache-2.0**; **Base/Large/Giant = CC-BY-NC-4.0** ⚠️ | 🟡 Use *Small* only for commercial work. The config offers base/large — disable those. |
| **SAM 2** | `skills/segmentation/sam2-segmentation` | Apache-2.0 ⚠️ | ✅ with attribution. |
| **Google Coral / libedgetpu / ai-edge-litert** | Coral TPU skills | Apache-2.0 ⚠️ | ✅. The committed `.so/.dll/.deb` binaries should be re-downloaded from upstream, not redistributed from this repo. |
| `libusb-1.0.dll` | Coral Windows skill | LGPL-2.1 ⚠️ | 🟡 Dynamic linking OK; keep notice. |
| `UsbDk_*.msi` | Coral Windows skill | Apache-2.0 ⚠️ | ✅ keep notice. |
| `torch`, `torchvision`, `onnxruntime`, `opencv-python-headless`, `numpy`, `Pillow` | skills | BSD/MIT/Apache-style | ✅ |
| OpenVINO | `yolo-detection-2026-openvino` | Apache-2.0 | ✅ |
| FaceNet / MTCNN code (`src/embedding`) | legacy | MIT (davidsandberg/facenet) ⚠️ | ✅ code; **but see datasets below.** |
| Face-recognition **training data** referenced by scripts (MS-Celeb-1M, VGG-Face) | `src/embedding/decode_msceleb_dataset.py`, `download_vgg_face_dataset.py` | Research / non-commercial; MS-Celeb-1M was withdrawn | 🔴 Do not train or ship models derived from these in a commercial product. Check any pre-trained model weights in `src/embedding/model*`. |
| `ncnn`, `pybind11`, LLVM `cindex.py` (vendored in `src/face_detection/assets/...`) | legacy | BSD-3 / BSD-3 / Apache+LLVM exception | ✅ keep notices. Not needed for a new project. |
| `docker-novnc` (Andrew Phillips) | `src/*/src/docker-novnc` | MIT | ✅ |
| Aegis (desktop host app) | referenced by all skills | **Not in this repo; separate SharpAI product with its own terms** ⚠️ | Skills expect Aegis as the host. For a client project you either implement your own JSONL host or obtain Aegis terms from SharpAI. |

### 2.4 Privacy / legal use-case notes (not code licences)
- **Face recognition / biometric data**: regulated (GDPR Art. 9, BIPA in Illinois, CCPA, India DPDP Act, etc.). Needs consent/legal basis, retention policy, and often a DPIA. Do not demo with real people's faces without consent.
- **License-plate reading** and **continuous CCTV** monitoring have local legal restrictions. Mark in the client contract who is the data controller.
- Third-party **camera brand** skills (`eufy`, `reolink`, `tapo`) may breach those vendors' ToS depending on how they access the device ⚠️.

---

## 3. What you MAY do

1. **Run a client demo** with the YOLO detection skill (CPU/GPU) on sample video/RTSP, plus `depth-estimation` using **Depth-Anything-V2-Small**, and `sam2-segmentation`. Do it from your own machine/cloud, not as a hosted service for the client's own users.
2. **Copy and modify MIT-licensed code** (skills, `env_config.py`, docs, benchmark scripts) into the client project, keeping the MIT notice in those files or in `THIRD_PARTY_NOTICES`.
3. **Use the protocol/architecture ideas** (JSONL stdin/stdout skill protocol, hardware-aware model loading). Ideas and protocols are not copyrightable; the *code* is MIT anyway.
4. **Use the HomeSec-Bench fixtures/benchmarks** for internal evaluation ⚠️ (check image provenance in `skills/analysis/*/fixtures` if you publish results).
5. **Swap in a permissively-licensed detector** (e.g. RT-DETR/YOLOX/YOLO-NAS-style Apache/MIT models — verify each) to avoid the AGPL question entirely.

## 4. What you must NOT do (without more work)

1. ❌ Ship, host, or build on `src/camera/src` (Shinobi) in a commercial product.
2. ❌ Deliver Ultralytics-based code/weights to a client or run it as their SaaS **without** either (a) AGPL compliance (source release) or (b) an Ultralytics Enterprise licence.
3. ❌ Use Depth-Anything-V2 Base/Large in a commercial setting (CC-BY-NC).
4. ❌ Train/ship face models using MS-Celeb-1M / VGG-Face, or redistribute pre-trained weights of unknown provenance from `src/embedding`.
5. ❌ Strip the MIT copyright notice or claim the SharpAI code as solely your own.
6. ❌ Redistribute the committed vendor binaries (`libedgetpu*.so/.dll/.deb`, `UsbDk`, `libusb`) from this repo; fetch them from upstream and ship their notices.
7. ❌ Use the default credentials in `docker/.env` (`ADMIN_PASSWORD=admin`, MySQL `shinobi`) in anything exposed beyond localhost.
8. ❌ Imply SharpAI/Aegis endorsement, or use their name/logo/trademarks in client materials.

---

## 5. Recommended actions before the demo / delivery

**Before the demo (low effort):**
- [ ] Run only the skills listed in §3.1; do not start the Shinobi/`docker/` stack.
- [ ] Use sample footage you own or have rights to (no real faces).
- [ ] Set Depth-Anything model to `depth-anything-v2-small`.
- [ ] Don't expose the demo publicly; change default passwords.

**Before delivering to the client:**
- [ ] Decide on the detector: buy Ultralytics Enterprise, accept AGPL, or replace with a permissive model.
- [ ] Remove `src/camera/`, `docker/` camera builds, `src/yolov7_*`, `src/yolov2_*`, and legacy face-recognition `src/embedding/` (or confirm each separately).
- [ ] Delete committed binaries/models/logs (`*.so`, `*.dll`, `*.deb`, `*.msi`, `*.pb`, `*.ckpt`, `docker_out.log`, `*.bak`) and download from upstream instead.
- [ ] Create `THIRD_PARTY_NOTICES.md` listing each dependency + licence (run `pip-licenses` / `license-checker` against the final dependency set).
- [ ] Get written terms from SharpAI if the client project depends on Aegis.
- [ ] Have legal review, especially for any face/biometric or plate-recognition feature.

---

## 6. Suggested minimal reuse set (cleanest path)

Copy into a **new** repo (not this fork) only:
- `skills/lib/env_config.py` (+ its tests)
- The skill protocol docs: `docs/skill-development.md`, `docs/detection-protocol.md`, `docs/skill-params.md`
- `skills/detection/yolo-detection-2026/scripts/detect.py` **only after** resolving the Ultralytics decision
- `skills/segmentation/sam2-segmentation` and `skills/transformation/depth-estimation` (Small model only)

Everything under `src/` and `docker/` should be treated as legacy / reference only.
