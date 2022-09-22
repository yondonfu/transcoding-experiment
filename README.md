# Transcoding Experiments

## Generate Transcoded Renditions

Use constrained encoding mode to constrain the maximum bitrate for each rendition so the first rendition has the lowest bitrate and quality and
the last rendition has the greatest bitrate and quality. Refer to the [ffmpeg h264 docs](https://trac.ffmpeg.org/wiki/Encode/H.264) for more details
on the constrained encoding mode.

```
ffmpeg -i data/mini_bbb.mp4 -c:v libx264 -crf 23 -maxrate 0.01M -bufsize 0.02M data/output1.mp4
ffmpeg -i data/mini_bbb.mp4 -c:v libx264 -crf 23 -maxrate 0.05M -bufsize 0.1M data/output2.mp4
ffmpeg -i data/mini_bbb.mp4 -c:v libx264 -crf 23 -maxrate 0.1M -bufsize 0.2M data/output3.mp4
ffmpeg -i data/mini_bbb.mp4 -c:v libx264 -crf 23 -maxrate 0.2M -bufsize 0.4M data/output4.mp4
ffmpeg -i data/mini_bbb.mp4 -c:v libx264 -crf 23 -maxrate 0.4M -bufsize 0.8M data/output5.mp4
ffmpeg -i data/mini_bbb.mp4 -c:v libx264 -crf 23 -maxrate 0.8M -bufsize 1.6M data/output6.mp4
```

`ls -lah data` will display the size of each file.

- `mini_bbb.mp4` is the source and will be the largest
- `output1.mp4` will be the smallest rendition
- `output6.mp4` will be the biggest rendition

## Generate VQA Scores for Transcoded Renditions

Use the `libvmaf` ffmpeg filter to calculate the VMAF, SSIM and PSNR scores for each transcoded rendition relative to the source. Refer to 
the [ffmpeg libvmaf docs](https://ffmpeg.org/ffmpeg-filters.html#libvmaf) for more details on the libvmaf filter.

```
ffmpeg -i data/output1.mp4 -i data/mini_bbb.mp4 -lavfi libvmaf='feature=name=float_ssim|name=psnr:log_path=output1.json' -f null -
ffmpeg -i data/output2.mp4 -i data/mini_bbb.mp4 -lavfi libvmaf='feature=name=float_ssim|name=psnr:log_path=output2.json' -f null -
ffmpeg -i data/output3.mp4 -i data/mini_bbb.mp4 -lavfi libvmaf='feature=name=float_ssim|name=psnr:log_path=output3.json' -f null -
ffmpeg -i data/output4.mp4 -i data/mini_bbb.mp4 -lavfi libvmaf='feature=name=float_ssim|name=psnr:log_path=output4.json' -f null -
ffmpeg -i data/output5.mp4 -i data/mini_bbb.mp4 -lavfi libvmaf='feature=name=float_ssim|name=psnr:log_path=output5.json' -f null -
ffmpeg -i data/output6.mp4 -i data/mini_bbb.mp4 -lavfi libvmaf='feature=name=float_ssim|name=psnr:log_path=output6.json' -f null -
```

A JSON file will be created for each transcoded rendition with the VMAF, SSIM and PSNR score data.

### Results

The following VQA scores were generated with:

- MacBook Pro (13 in, 2020, M1), Apple M1, 16 GB RAM
- ffmpeg 5.1.1

Notes on VQA scores:

- The max value for VMAF is 100
- The max value for SSIM is 1.0
- The typical value for PSNR is between 30 and 50 DB given a bit depth of 8 bits where higher is better

|               | VMAF      | SSIM     | PSNR-Y    | PSNR-CB   | PSNR-CR   |
| ------------- | --------- | -------- | --------- | --------- | --------- |
| `output1.mp4` | 9.17055   | 0.377001 | 10.193230 | 17.522872 | 24.376067 |
| `output2.mp4` | 6.818648  | 0.689800 | 23.198833 | 24.843488 | 28.833926 |
| `output3.mp4` | 8.578323  | 0.799100 | 28.653974 | 36.363270 | 37.092051 |
| `output4.mp4` | 35.450671 | 0.897993 | 32.023443 | 40.398312 | 40.848148 |
| `output5.mp4` | 60.888921 | 0.970633 | 35.933751 | 43.536530 | 43.811918 |
| `output6.mp4` | 78.526422 | 0.989975 | 39.349071 | 45.856691 | 46.085067 |

The JSON files for each transcoded rendition can be found in the `vqa` directory.

The source and transcoded files can also be found at these locations:

```
mini_bbb.mp4 -> https://transcoding-experiment.infura-ipfs.io/ipfs/QmVKJcamAKA7pNrJhGkARhdqSB6WauDhfhg42ZRiJqowoG
output1.mp4 -> https://transcoding-experiment.infura-ipfs.io/ipfs/QmVYCGWbAURzG4hAgjX2ZuoUeUDpQREDmWaf7zAB2uVqso
output2.mp4 -> https://transcoding-experiment.infura-ipfs.io/ipfs/QmZXykrhthfzarp7J7a8cgs3ghdNfmZnLHFsXa1So76EwU
output3.mp4 -> https://transcoding-experiment.infura-ipfs.io/ipfs/QmQHyjtJezrnpgD5uVip9WAS9SkUSuR8d9wufoAdPGBRna
output4.mp4 -> https://transcoding-experiment.infura-ipfs.io/ipfs/QmTgQCb8gCqEcaVM1yRqRmweAymQUvVi1YL417cUfujkSU
output5.mp4 -> https://transcoding-experiment.infura-ipfs.io/ipfs/QmSGnNkMAvA8z7N5aorv2wWkB3u66VoNXnYnwaeTMFoWxL
output6.mp4 -> https://transcoding-experiment.infura-ipfs.io/ipfs/QmfMNUz4TrVYzipBWbTfnr45Mzvt7HhDN6QXG7wmBTpcyv
```

## C2PA Manifests

Use the [c2patool](https://github.com/contentauth/c2patool) to add [C2PA](https://c2pa.org/specifications/specifications/1.1/index.html) manifests to assets. A manifest consists of a signed claim with a set of assertions about the asset that describe the provenance of the asset.

First generate a transcoded rendition:

```
ffmpeg -i data/mini_bbb.mp4 -c:v libx264 -crf 23 -maxrate 0.8M -bufsize 1.6M data/output6.mp4
```

Add a manifest to the source file:

```
c2patool data/mini_bbb.mp4 -m c2pa/mini_bbb_manifest.json -o c2pa/signed_mini_bbb.mp4
```

`signed_mini_bbb.mp4` is the updated source file with an embedded C2PA manifest.

Show the detailed manifest embedded in the new source file:

```
c2patool -d c2pa/signed_mini_bbb.mp4
```

Add a manifest to the rendition file and specify the source file as the parent:

```
# As of c2patool v0.3.6, the -p flag will look for the provided file relative to the directory that is specified by -m
# See https://github.com/contentauth/c2patool/issues/74
# In order to workaround this issue, we can copy the parent file to the directory that c2patool will be looking in
cp data/mini_bbb.mp4 c2pa/
c2patool data/output6.mp4 -m c2pa/mini_bbb_rendition_manifest.json -p ../mini_bbb.mp4 -o c2pa/signed_output6.mp4
# We can remove the copied parent file
rm c2pa/mini_bbb.mp4
```

Show the detailed manifest embedded in the new rendition file:

```
c2patool -d c2pa/signed_output6.mp4
```