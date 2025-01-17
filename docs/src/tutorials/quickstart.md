# Quickstart

```julia
using Flux, Metalhead
```

Using a model from Metalhead is as simple as selecting a model from the table of [available models](@ref API-Reference). For example, below we use the pre-trained ResNet-18 model.
```julia
using Flux, Metalhead

model = ResNet(18; pretrain = true)
```

Now, we can use this model with Flux like any other model.

First, let's check the accuracy on a test image from ImageNet.
```julia
using Images

# test image
img = Images.load(download("https://cdn.pixabay.com/photo/2015/05/07/11/02/guitar-756326_960_720.jpg"));
```
We'll use the popular [DataAugmentation.jl](https://github.com/lorenzoh/DataAugmentation.jl) library to crop our input image, convert it to a plain array, and normalize the pixels.
```julia
using DataAugmentation, OneHotArrays

DATA_MEAN = (0.485, 0.456, 0.406)
DATA_STD = (0.229, 0.224, 0.225)

augmentations = CenterCrop((224, 224)) |>
                ImageToTensor() |>
                Normalize(DATA_MEAN, DATA_STD)
data = apply(augmentations, Image(img)) |> itemdata

# image net labels
labels = readlines(download("https://raw.githubusercontent.com/pytorch/hub/master/imagenet_classes.txt"))

onecold(model(Flux.unsqueeze(data, 4)), labels)
```

Below, we train it on some randomly generated data.

```julia
using OneHotArrays: onehotbatch

batchsize = 1
data = [(rand(Float32, 224, 224, 3, batchsize), onehotbatch(rand(1:1000, batchsize), 1:1000))
        for _ in 1:3]
opt = ADAM()
ps = Flux.params(model)
loss(x, y, m) = Flux.Losses.logitcrossentropy(m(x), y)
for (i, (x, y)) in enumerate(data)
    @info "Starting batch $i ..."
    gs = gradient(() -> loss(x, y, model), ps)
    Flux.update!(opt, ps, gs)
end
```
