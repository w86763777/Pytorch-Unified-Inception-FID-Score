# Pytorch Implementation of Inception Score and FID Score

- Incpetion on CIFAR10
    |      |Official  |This implementation|
    |------|----------|-------------------|
    |Train |11.24±0.20|11.26±0.20         |
    |Test  |10.98±0.22|10.97±0.33         |

- FID on CIFAR10 (Test 10k)
    |                   |Official FID|This implementation|
    |-------------------|------------|-------------------|
    |CIFAR10 (Train 50k)|3.15        |3.15               |

- FID on CIFAR10 (Train 50k)
    |                   |Official FID|This implementation|
    |-------------------|------------|-------------------|
    |CIFAR10 (Test 10k) |3.15        |3.15               |

- Official implementation: [Inception Score](https://github.com/openai/improved-gan), [Fréchet Inception Distance](https://github.com/bioinf-jku/TTUR)

- The implementation of FID is inspired 
by [pytorch-fid](https://github.com/mseitzer/pytorch-fid)

- The official implementation of Inception Score ignores the bias term of last
layer in inception v3, this is the most important detail to reimplement it

- Due to the difference of framework implementations, both scores are slightly different from official implementations

## Requirements
- torch 1.7.1
- torchvision 0.8.2
- tqdm 4.55.1
- scipy 1.5.4
- Install requirements
    ```
    pip install -r requirements.txt
    ```

## Example
- Prepare Statistics for calculating FID Score
    - [Download](https://drive.google.com/drive/folders/1UBdzl6GtNMwNQ5U-4ESlIer43tNjiGJC?usp=sharing) Precalculated Statistics for your dataset or
    - Calculate statistics for your dataset. See [example](./calc_stats.py)
    ```
    python calc_stats.py \
        --dataset cifar10 \
        --output ./cifar10_test.npz
    ```

- Calculate Inception Score and FID Score at a time. Both score share same
Inception v3 outputs so only one forward propagation is needed for each image
    ```
    python calc_score.py \
        --dir ./cifar10/train \
        --stats ./cifar10_test.npz
    ```

## Integrate into training scripts
```python
import torch
from score.both import get_inception_and_fid_score


# Support: Load every images before calculation
images = np.array(...)  # Channel first, Normalized to [0, 1]
                        # e.g. shape = [N, 3, 32, 32]
                        # the image size will be resize to [299, 299] to match
                        # Inception V3 input size
IS, FID = get_inception_and_fid_score(
    images, args.stats, use_torch=args.use_torch, verbose=True)
print(IS, FID)


# Support: Load images on demand
def images_generator(images):
    for image in images:
        yield image

IS, FID = get_inception_and_fid_score(
    images_generator(files), args.stats, num_images=len(files),
    use_torch=args.use_torch, verbose=True, parallel=False)
print(IS, FID)
```

## TODO

- [x] Dynamic loading images
- [x] Multi-GPU computing