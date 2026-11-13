### Python

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements-apple-silicon.txt
```

### Flutter

```bash
cd app/flutter_app
flutter pub get
```

## Training Commands

Run these commands from the project root after activating your virtual environment.

### 1. Prepare the dataset

If you have the Kaggle zip:

```bash
python -m training.prepare_dataset --dataset-zip ./dataset.zip --output-dir ./data --seed 42
```

If you already extracted PlantVillage:

```bash
python -m training.prepare_dataset --dataset-dir /path/to/PlantVillage --output-dir ./data --seed 42
```

If you want the merged five-crop prepared dataset:

```bash
python -m training.prepare_dataset \
  --dataset-zip ./dataset.zip \
  --external-dataset-dir ./external_datasets/extracted/rice_inner \
  --external-dataset-dir ./external_datasets/extracted/wheat_zenodo_7307816 \
  --output-dir ./data_full \
  --seed 42
```

### 2. Train one disease model

Example for tomato:

```bash
python -m training.train_disease_model --crop tomato --data-dir ./data --artifact-dir ./models --report-dir ./reports/thesis --epochs 50 --batch-size 32 --run-name thesis_tomato
```

Other valid crops:

- `tomato`
- `potato`
- `bell_pepper`
- `wheat`
- `rice`

### 3. Train all disease models

This is the main thesis command:

```bash
python -m training.train_all_disease_models --data-dir ./data --artifact-dir ./models --report-dir ./reports/thesis --epochs 50 --batch-size 32 --run-name thesis_full_run
```

For the merged five-crop dataset, point training commands at `./data_full` instead of `./data`.

### 4. Rebuild thesis reports from trained artifacts

Use this if models are already trained and you only want the graphs and tables again:

```bash
python -m training.generate_thesis_report --artifact-dir ./models --report-dir ./reports/thesis
```

### 5. Evaluate trained models

```bash
python -m training.evaluate_models --artifact-dir ./models --report-dir ./reports
```

### 6. Export TFLite models

```bash
python -m training.export_tflite --artifact-dir ./models --export-dir ./models/exported
```

### 7. Copy trained models into the Flutter app

```bash
python -m training.sync_flutter_assets --artifact-dir ./models --export-dir ./models/exported --flutter-dir ./app/flutter_app
```

## Training Flow For A New Person

If someone else wants to train everything from scratch, the full flow is:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements-apple-silicon.txt
python -m training.prepare_dataset --dataset-zip /path/to/dataset.zip --output-dir ./data --seed 42
python -m training.train_all_disease_models --data-dir ./data --artifact-dir ./models --report-dir ./reports/thesis --epochs 50 --batch-size 32 --run-name thesis_full_run
python -m training.export_tflite --artifact-dir ./models --export-dir ./models/exported
python -m training.sync_flutter_assets --artifact-dir ./models --export-dir ./models/exported --flutter-dir ./app/flutter_app
```

## If The Models Are Already Trained

You do not need to retrain. Use this shorter path:

```bash
python -m training.generate_thesis_report --artifact-dir ./models --report-dir ./reports/thesis
python -m training.export_tflite --artifact-dir ./models --export-dir ./models/exported
python -m training.sync_flutter_assets --artifact-dir ./models --export-dir ./models/exported --flutter-dir ./app/flutter_app
cd app/flutter_app
flutter pub get
flutter build apk
```

## Android App

The Flutter app is Android-only for this MVP. It uses:

- automatic leaf-type detection before disease inference
- camera capture
- gallery selection
- unknown-leaf and low-confidence rejection for unsupported or random inputs
- local `.tflite` model per packaged crop
- local label maps
- top-1 prediction with confidence
- bilingual English and Urdu copy
- local treatment lookup from JSON
- SQLite scan history with result re-open support

Current packaged model status:

- tomato: available
- potato: available
- bell pepper: available
- wheat: available
- rice: available

### Run on an Android device

```bash
cd app/flutter_app
flutter pub get
flutter run
```

### Build the Android APK

```bash
cd app/flutter_app
flutter build apk
```

APK output:

- [app-release.apk](app/flutter_app/build/app/outputs/flutter-apk/app-release.apk)

Install with `adb` if you have it:

```bash
adb install -r ./app/flutter_app/build/app/outputs/flutter-apk/app-release.apk
```

### Quick Android test checklist

1. Install the APK on an Android device.
2. Open the app.
3. Choose an image from camera or gallery.
4. Confirm the app auto-detects `Tomato`, `Potato`, `Bell Pepper`, `Wheat`, or `Rice`.
5. Confirm the app shows:
   - detected leaf
   - predicted class
   - confidence
   - image preview

## Current Trained MVP Outputs

The latest thesis run is:

- run name: `thesis_full_5crop`
- summary: [run_manifest.json](reports/thesis_full/summary/run_manifest.json)

Latest test metrics:

- tomato: `89.44%` accuracy, `0.8736` macro F1
- potato: `97.22%` accuracy, `0.9495` macro F1
- bell pepper: `99.60%` accuracy, `0.9958` macro F1
- wheat: `71.43%` accuracy, `0.6849` macro F1
- rice: `98.97%` accuracy, `0.9898` macro F1

