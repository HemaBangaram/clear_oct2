# Sim2Real Clear Add-on (Physics-Guided Sim2Real Diffusion)

Clear counterpart of the fog / rain add-ons. DreamBooth-fine-tuned Stable Diffusion v1.5 learns the
`<clear_sim>` concept from CARLA clear images; at test time a real clear frame is translated to the
CARLA-clear style (Real2Sim) via img2img + IP-Adapter Plus.

## Layout
```
training/
  training_clear_sim.ipynb        DreamBooth fine-tuning
  dataset_clear_sim/              <- put CARLA clear TRAINING images here (5-20 images)
testing/
  testing_clear_real2sim.ipynb    inference (real clear -> sim style)
  evaluation.ipynb              CLIP-DS, cosine, LPIPS, Gram style difference
  input_clear_real/               <- put real clear TEST images here
  style_clear_sim/                <- put ONE CARLA clear style reference here
  test_vid/                     (optional) test videos
clear_kaggle_run.ipynb            end-to-end runner for Kaggle (GPU)
```

## Run order
1. Add images to `training/dataset_clear_sim/`, `testing/input_clear_real/`, `testing/style_clear_sim/`.
2. Create a GitHub repo named `sim2real-clear-addon` and push this folder
   (or edit the clone URL in `clear_kaggle_run.ipynb`; it currently points to
   `https://github.com/HemaBangaram/sim2real-clear-addon.git`).
3. Open `clear_kaggle_run.ipynb` on Kaggle (GPU) -> Run All.
   Order: train -> inference -> evaluation -> zip results.

Training outputs `checkpoint_clear_sim/` and `model_clear_sim/` (under `training/`);
inference writes to `testing/output_clear_real2sim/`. These are listed in `.gitignore`
so they are never pushed.

## Names used by this repo (identical pattern to the fog repo)
| item | value |
|---|---|
| instance prompt (training) | `<clear_sim> simulation` |
| sample prompt (training notebook) | `<clear_sim> a road on a clear day` |
| trained model folder | `training/model_clear_sim` |
| style reference folder | `testing/style_clear_sim` |
| evaluation captions | "a road in clear weather in real world" -> "a road in clear weather in simulation" |

All other code and hyper-parameters (learning rate 5e-6, 300 steps, IP-Adapter scale 0.9,
img2img strength 0.3, 10 steps) are unchanged from the fog repo, so results are comparable.

## Data source
Real test images come from the BDD100K weather sorter (`bdd_clear.zip` -> `clear/images/`,
daytime clear/overcast, blurry frames already moved out to `blurry/`). Copy a handful into
`testing/input_clear_real/` for a quick check; for bulk conversion use the multi-variant pipeline
and point it at the unzipped `clear/images/` folder.

## Notes specific to clear
- CARLA training images should be **dry, clear daytime** scenes (vary sun angle / town if you can).
  Avoid images with any rain, fog or night lighting so the concept stays clean.
- Clear is the lowest-risk domain shift (colour / texture / lighting style), so the default
  img2img strength of 0.3 is a sensible start. Raise to 0.4-0.5 for a stronger CARLA look and check
  for hallucinated objects.
- Unlike the fog / rain repos there is no atmospheric-haze physics to validate here; if you use the
  multi-variant pipeline, keep the haze stage off (or minimal) for clear.
- (Suggestions only - none of these values have been tuned on your data.)
