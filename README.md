# KiCad Auto-Export Pipeline

This repository contains a GitHub Actions workflow that automatically exports several assets from a KiCad project. The workflow is intended to run via `workflow_call` and uses a GitHub App for authentication.

The pipeline performs the following tasks:

1. Retrieves a GitHub App token.
2. Checks out the repository.
3. Pulls the KiCad 9 Docker image.
4. Clones the official KiCad 3D model library.
5. Exports the following from the KiCad project:

   * Schematic (SVG)
   * PCB (front and back 2D SVG)
   * 3D board renders (front and back PNG)
   * STEP model
6. Crops the rendered board images using ImageMagick.
7. Commits and pushes the generated files back to the repository.

All outputs are saved into the `images/` directory.

## Workflow File

The workflow is defined in `.github/workflows/test2.yml` (or the filename you choose). It is triggered using `workflow_call` and requires two secrets:

* `APP_ID`
* `APP_PRIVATE_KEY`

These must belong to a GitHub App with permission to write repository contents.

## Key Features

### KiCad Exporting

The workflow runs KiCad inside a Docker container. It sets both `KICAD8_3DMODEL_DIR` and `KICAD9_3DMODEL_DIR` so that 3D models resolve correctly when using KiCad 8 or 9 formats.

It exports:

* `images/sch.svg` — Schematic
* `images/pcbf.svg` — PCB front view
* `images/pcbb.svg` — PCB back view
* `images/board.front.png` — 3D render (front)
* `images/board.back.png` — 3D render (back)
* `images/board.step` — STEP mechanical model

3D renders are produced using a transparent background and basic quality to avoid ambient occlusion shadows.

### Image Cropping

ImageMagick trims transparent borders from:

* `board.front.png`
* `board.back.png`

This keeps the final images clean and tightly framed.

### Automatic Commit and Push

After generating the assets, the workflow commits the updated `images/` directory and pushes the results to the `main` branch using the GitHub App token.

## Usage

To use this workflow from another workflow:

```yaml
jobs:
  call-kicad-export:
    uses: your/repository/.github/workflows/test2.yml@main
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```

## Requirements

* A GitHub App with the `contents: write` permission.
* The repository must contain a `kicad/` folder with:

  * One `.kicad_sch` file
  * One `.kicad_pcb` file

## Output

After each workflow run, the latest images and STEP model will be committed to the `images/` directory in the repository head.
