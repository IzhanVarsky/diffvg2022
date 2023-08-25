# What this fork is about??

Here I fixed some bugs of the amazing DiffVG work.

### Known bugs

NOT fixed in this repo:

* BAD support of RadialGradient.
* BAD support of Ellipse. Currently, it has only one radius, but it should have two.

Fixed in this repo:

* Problems with installing for TensorFlow:
    * Solution: DISABLED TF support.
* Problems with running on GPU.
    * [Solution](https://github.com/BachiLi/diffvg/issues/29#issuecomment-994807865)
* Problem with `import diffvg` on Windows.
    * Solution: rename the `your_venv\lib\site-packages\diffvg-0.0.1-_some_versions_\diffvg` file to `diffvg.pyd`.
    * DO IT MANUALLY!
* Crashing with `style="fill:url(#n)"`
  in [pydiffvg/parse_svg.py](./pydiffvg/parse_svg.py).
    * Solution: removed bugged `if` clause.
* Crashing when parsing svg with wrong order of tags with `id` attribute.
    * (*) Solution: topsort visual tags. Current solution is not a full topsort, it sorts only in `<defs>`!
* No saving info about `gradientUnits` attribute in linear gradients.
    * (*) Solution: added such field to `LinearGradient` class. IT IS NOT PASSED to Renderer, so it cannot be used while
      optimizations.
* Saving only the first path when it consists of multiple subpaths.
    * Solution: added `for` loop for adding all the subpaths.
* Crashing when using `float64` points.
    * Solution: added cast to `torch.float`.
* Bad reading of X,Y coordinates of `<rect>`.
    * Fixed.

### Modifications

* When saving, an extra group tag is added.
    * Solution: removed extra group.
* Forcing compile for GPU for `RTX 3090`.
    * Solution: when running `setup.py install` pass `RTX_3090=1` flag.
    * It is helpful when compiling in Docker.
* Increased `max_hit_shapes` param for huge SVGs.

### Added functions

* `svg_to_str()` in [pydiffvg/save_svg.py](./pydiffvg/save_svg.py)
    * Returns SVG as string
    * Can convert lines in `<path>` only as cubic curves.
* Using `viewbox` param when saving. See [pydiffvg/save_svg.py](./pydiffvg/save_svg.py).
* Normalizing coordinates to `[0, 1]` interval and rescaling it. See [pydiffvg/utils.py](./pydiffvg/utils.py).
* `svg_str_to_scene()` in [pydiffvg/parse_svg.py](./pydiffvg/parse_svg.py).
    * Reads SVG from string

### How to install?

On Windows before all operations:

* Install Visual Build Tools. Choose `C++ classic apps` while installing. I tested only 2019
  version. [Link to the latest release (2022 version)](https://aka.ms/vs/17/release/vs_BuildTools.exe).
* Installing full Visual Studio instead of Build Tools also should help (but I am not sure).

Common for Windows and Linux:

* `pip install torch svgwrite svgpathtools cssutils numba torch-tools cmake visdom scikit-learn scikit-image matplotlib`
* `git clone https://github.com/IzhanVarsky/diffvg --recursive`
* `cd ./diffvg`
* `python3 ./setup.py install`

On Windows after all operations:

* Rename the `your_venv\lib\site-packages\diffvg-0.0.1-_some_versions_\diffvg` file to `diffvg.pyd`.

#### Troubleshooting

My current working version of python on windows is python3.10.7.

* Error: `LINK : fatal error LNK1104: cannot open file "python310.lib"`. Check that cmake finds the correct PythonLibs.
  It seems to be that cmake always finds the latest python version on computer.

# diffvg

Differentiable Rasterizer for Vector Graphics
https://people.csail.mit.edu/tzumao/diffvg

diffvg is a differentiable rasterizer for 2D vector graphics. See the webpage for more info.

![teaser](https://user-images.githubusercontent.com/951021/92184822-2a0bc500-ee20-11ea-81a6-f26af2d120f4.jpg)

![circle](https://user-images.githubusercontent.com/951021/63556018-0b2ddf80-c4f8-11e9-849c-b4ecfcb9a865.gif)
![ellipse](https://user-images.githubusercontent.com/951021/63556021-0ec16680-c4f8-11e9-8fc6-8b34de45b8be.gif)
![rect](https://user-images.githubusercontent.com/951021/63556028-12ed8400-c4f8-11e9-8072-81702c9193e1.gif)
![polygon](https://user-images.githubusercontent.com/951021/63980999-1e99f700-ca72-11e9-9786-1cba14d2d862.gif)
![curve](https://user-images.githubusercontent.com/951021/64042667-3d9e9480-cb17-11e9-88d8-2f7b9da8b8ab.gif)
![path](https://user-images.githubusercontent.com/951021/64070625-7a52b480-cc19-11e9-9380-eac02f56f693.gif)
![gradient](https://user-images.githubusercontent.com/951021/64898668-da475300-d63c-11e9-917a-825b94be0710.gif)
![circle_outline](https://user-images.githubusercontent.com/951021/65125594-84f7a280-d9aa-11e9-8bc4-669fd2eff2f4.gif)
![ellipse_transform](https://user-images.githubusercontent.com/951021/67149013-06b54700-f25b-11e9-91eb-a61171c6d4a4.gif)

# Install

```
git submodule update --init --recursive
conda install -y pytorch torchvision -c pytorch
conda install -y numpy
conda install -y scikit-image
conda install -y -c anaconda cmake
conda install -y -c conda-forge ffmpeg
pip install svgwrite
pip install svgpathtools
pip install cssutils
pip install numba
pip install torch-tools
pip install visdom
python setup.py install
```

# Install using poetry

## prerequisite

install python 3.7, poetry and ffmpeg

```
# install poetry (mac, linux)
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -

# install ffmpeg

(macos)
brew install ffmpeg

(linux)
sudo apt install ffmpeg

or use conda
conda install -y -c conda-forge ffmpeg
```

## Install python packages

```
# install all python dependencies
poetry install

# install pydiffvg
poetry run python setup.py install
```

Now to run the apps, just add `poetry run` before each of the commands below, e.g.

```
poetry run python single_circle.py
```

# Building in debug mode

```
python setup.py build --debug install
```

# Run

```
cd apps
```

Optimizing a single circle to a target.

```
python single_circle.py
```

Finite difference comparison.

```
finite_difference_comp.py [-h] [--size_scale SIZE_SCALE]
                               [--clamping_factor CLAMPING_FACTOR]
                               [--use_prefiltering USE_PREFILTERING]
                               svg_file
```

e.g.,

```
python finite_difference_comp.py imgs/tiger.svg
```

Interactive editor

```
python svg_brush.py
```

Painterly rendering

```
painterly_rendering.py [-h] [--num_paths NUM_PATHS]
                       [--max_width MAX_WIDTH] [--use_lpips_loss]
                       [--num_iter NUM_ITER] [--use_blob]
                       target
```

e.g.,

```
python painterly_rendering.py imgs/fallingwater.jpg --num_paths 2048 --max_width 4.0 --use_lpips_loss
```

Image vectorization

```
python refine_svg.py [-h] [--use_lpips_loss] [--num_iter NUM_ITER] svg target
```

e.g.,

```
python refine_svg.py imgs/flower.svg imgs/flower.jpg
```

Seam carving

```
python seam_carving.py [-h] [--svg SVG] [--optim_steps OPTIM_STEPS]
```

e.g.,

```
python seam_carving.py imgs/hokusai.svg
```

Vector variational autoencoder & vector GAN:

For the GAN models, see `apps/generative_models/train_gan.py`. Generate samples from a pretrained
using `apps/generative_models/eval_gan.py`.

For the VAE models, see `apps/generative_models/mnist_vae.py`.

If you use diffvg in your academic work, please cite

```
@article{Li:2020:DVG,
    title = {Differentiable Vector Graphics Rasterization for Editing and Learning},
    author = {Li, Tzu-Mao and Luk\'{a}\v{c}, Michal and Gharbi Micha\"{e}l and Jonathan Ragan-Kelley},
    journal = {ACM Trans. Graph. (Proc. SIGGRAPH Asia)},
    volume = {39},
    number = {6},
    pages = {193:1--193:15},
    year = {2020}
}
```
