---
layout: post
date: 2013-07-02 17:18
title: Texture Mode Bug in NVIDIA CUDA C++ Driver
published: true
---

After wrangling with a bug in one of our GPU-based solvers at the office, I've
finally found the reason.  NVIDIA has a bug in their current GPU drivers which
incorrectly handles the` cudaAddressModeBorder` texture addressing mode.
`cudaAddressModeBorder` is supposed to return 0 when the texture is indexed
out-of-bounds.  Instead, the current NVIDIA GPU driver (tested on driver version
320.18) uses `cudaAddressModeClamp` even if `cudaAddressModeBorder` is
specified.  For those of us relying on proper out-of-range behavior for complex
numerical algorithms, this is catastrophic.

The only solution at the time of writing is to manually bounds check your
texture calls.  While this code is trivial and short (see example below), it
reduces performance (normally, texture indexing is handled by special hardware
on NVIDIA Geforce graphics cards) and is immensely frustrating to diagnose.



Background
----------

NVIDIA describes their texture modes [here in the 6th bullet point][1] under
"Texture Memory".

[1]: <http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#texture-and-surface-memory>

>   *The addressing mode.* It is valid to call the device functions of Section
>   B.8 with coordinates that are out of range. The addressing mode defines what
>   happens in that case. The default addressing mode is to clamp the
>   coordinates to the valid range: [0, N) for non-normalized coordinates and
>   [0.0, 1.0) for normalized coordinates. If the border mode is specified
>   instead, texture fetches with out-of-range texture coordinates return zero.
>   For normalized coordinates, the warp mode and the mirror mode are also
>   available. When using the wrap mode, each coordinate x is converted to
>   frac(x)=x floor(x) where floor(x) is the largest integer not greater than x.
>   When using the mirror mode, each coordinate x is converted to frac(x) if
>   floor(x) is even and 1-frac(x) if floor(x) is odd. The addressing mode is
>   specified as an array of size three whose first, second, and third elements
>   specify the addressing mode for the first, second, and third texture
>   coordinates, respectively; the addressing mode are cudaAddressModeBorder,
>   cudaAddressModeClamp, cudaAddressModeWrap, and cudaAddressModeMirror;
>   cudaAddressModeWrap and cudaAddressModeMirror are only supported for
>   normalized texture coordinates

Specifically note that,

>   If the border mode is specified instead, texture fetches with out-of-range
>   texture coordinates return zero.

No comments are made which restrict border mode to normalized or non-normalized
coordinates and hence it should work for both.

Solution
--------

In order to work around this, I've left my code using `cudaAddressModeBorder`, I
just assume it won't actually work.  First, I setup the texture(s).  This
includes copying over a constant value providing me the size of the textures.

{% highlight cpp %}
// parametrize, allocate array on device, parametrize and copy the data
cudaChannelFormatDesc texLUT_desc = cudaCreateChannelDesc<float>();
cudaArray* real_texLUT_cuarr = (cudaArray*)texLUT_cuarr;
cudaMallocArray( &real_texLUT_cuarr, &texLUT_desc, arrLUT_param.x, arrLUT_param.y );
cudaMemcpyToArray( real_texLUT_cuarr, 0, 0, lut, arrLUT_size, cudaMemcpyHostToDevice);

// set texture paramters and bind the array on device to the texture
texLUT.addressMode[0] = cudaAddressModeBorder;
texLUT.addressMode[1] = cudaAddressModeBorder;
texLUT.filterMode = cudaFilterModeLinear;
texLUT.normalized = false;
cudaBindTextureToArray( texLUT, real_texLUT_cuarr, texLUT_desc);
cudaMemcpyToSymbol(c_LUT, &arrLUT_param, sizeof(uint2), 0, cudaMemcpyHostToDevice);
{% endhighlight %}

Then in my kernels where I use the texture I perform manual bounds checking.

{% highlight cpp %}
if( (x < 0) || (x > c_LUT.x) || (y < 0) || (y > c_LUT.y) )
{
    return 0.0f;
} else
{
    return tex2D(texLUT, x, y);
}
{% endhighlight %}

### Further Reading

I've also found [this thread][2] on the NVIDIA developer forums where other
users confirm the bug and another user has confirmed a bug report is in their
tracker.

[2]: <https://devtalk.nvidia.com/default/topic/546726/how-to-change-the-adress-mode-of-a-texture-/?offset=12#3843188>
