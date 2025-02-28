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

# Install (Windows)
1. Install Python 3.10 from the [Microsoft Store](https://www.microsoft.com/store/productId/9PJPW5LDXLZ5?ocid=pdpshare)
2. Install the [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads)
3. Install [ffmpeg](https://www.ffmpeg.org/). Make sure ffmpeg is added to your `PATH` environment variable (Eg. `C:\FFmpeg\bin`)
```sh
> git clone https://github.com/benbaker76/diffvg
> cd diffvg
> git submodule update --init --recursive
> python -m venv .venv
> .\.venv\Scripts\activate
> pip install torch==2.4.0+cu124 torchvision==0.19.0+cu124 --index-url https://download.pytorch.org/whl/cu124
> pip install -r requirements.txt
> set LIBDIR=.\.venv\Lib\site-packages
> python setup.py bdist_wheel
> pip install dist\diffvg-0.0.1-cp310-cp310-win_amd64.whl
# Test it's working
> python apps\refine_svg.py
usage: refine_svg.py [-h] [--use_lpips_loss] [--num_iter NUM_ITER] svg target
refine_svg.py: error: the following arguments are required: svg, target
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

For the GAN models, see `apps/generative_models/train_gan.py`. Generate samples from a pretrained using `apps/generative_models/eval_gan.py`.

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
