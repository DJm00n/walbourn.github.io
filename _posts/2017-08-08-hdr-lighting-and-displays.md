---
layout: msdnpost
title: HDR Lighting and Displays
date: 2017-08-08 22:33
author: Chuck Walbourn
comments: true
categories: [github, hdr]
---
High Dynamic Range (<a href="https://en.wikipedia.org/wiki/High-dynamic-range_imaging">HDR</a>) lighting has been used in games for a long time, popularized by titles like Valve's <a href="https://en.wikipedia.org/wiki/Half-Life_2">Half-Life 2</a> using DirectX 9.0c. The rendering uses float-point render targets, allowing the lighting to exceed the normal 0 to 1 range. Then the final result is <a href="https://en.wikipedia.org/wiki/Tone_mapping">tone-mapped</a> back into normal range for display. The result is much improved contrast, making it easier to see a mix of dark interiors with bright exteriors, more realistic outdoor lighting, and a host of special effects.
<!--more-->

Recent advances in display technology are adding the ability to render a wider range of luminance directly on the display, enabling a wider gamut of colors and better handling of HDR images and scenes. The 4k Ultra High Definition (4K UHD) standard includes the <a href="https://en.wikipedia.org/wiki/High-dynamic-range_video#HDR10">HDR10 Media Profile</a> supported by game consoles like the Xbox One S and Xbox One X, as well as by Windows PCs running the Windows 10 Creators Update. A range of HDR10 capable TVs and monitors are becoming available.

<h1>DeviceResources</h1>

The <code>DeviceResources</code> abstraction (<a href="https://github.com/Microsoft/DirectXTK/wiki/DeviceResources">DX11</a> and <a href="https://github.com/Microsoft/DirectXTK12/wiki/DeviceResources">DX12</a>) used in <a href="https://github.com/walbourn/directx-vs-templates"><strong>directx-vs-templates</strong></a> now supports an options flag for HDR10 display output. To enable this support, you must build the code using the <a href="https://walbourn.github.io/windows-10-creators-update-sdk/">Windows 10 Creators Update SDK (15063)</a> which also implies that you are using VS 2017--the Windows 10 SDK (15063) doesn't officially support use with VS 2015. The code will run on older versions of Windows, but you can only get HDR10 output if running on a PC with the Windows 10 Creators Update as well as the required video card, driver, and display combination.

Note the latest <code>DeviceResources</code> will build with VS 2015 and/or the Windows 10 Anniversary Update SDK (14393), but won't be able to detect HDR10 display output.

<h1>DirectX Tool Kit</h1>

In order to render an HDR10 swapchain on PC, UWP, and Xbox One, a postprocessing step is typically required to rotate the color space appropriately. To simplify this, I've added a <code>ToneMapEffect</code> (along with other post-processing support) to DirectX Tool Kit (<a href="https://github.com/Microsoft/DirectXTK/releases">DX11</a> and <a href="https://github.com/Microsoft/DirectXTK12/releases">DX12</a>). The <code>ToneMapEffect</code> class supports preparing the HDR10 swapchain (rotating from Rec.709 to Rec.2020 color primaries, and applying the ST.2084 curve), as well as traditional tone-mapping for supporting classic Standard Dynamic Range (SDR) displays.

Details on how to use these classes are on the wiki (<a href="https://github.com/Microsoft/DirectXTK/wiki/PostProcess">DX11</a> and <a href="https://github.com/Microsoft/DirectXTK12/wiki/PostProcess">DX12</a>) and in the tutorials (<a href="https://github.com/Microsoft/DirectXTK/wiki/Using-HDR-rendering">DX11</a> and <a href="https://github.com/Microsoft/DirectXTK12/wiki/Using-HDR-rendering">DX12</a>).

I've also added HDR10 display support and tone-mapped SDR display support to the DirectX Tool Kit Model Viewer (<a href="https://github.com/walbourn/directxtkmodelviewer/releases">DX11</a> and <a href="https://github.com/walbourn/directxtk12modelviewer/releases">DX12</a>).

<h1>Samples</h1>

The official HDR sample is on [DirectX-Graphics-Samples](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HDR). See also the <strong>SimpleHDR</strong> sample in the [Xbox-ATG-Samples](https://github.com/microsoft/Xbox-ATG-Samples) GitHub repository--there are DirectX 11 and DirectX 12 versions for UWP, Win32 PC, and Xbox One XDK.

<h1>DirectXTex</h1>

Note that last year I added support for the <a href="https://en.wikipedia.org/wiki/RGBE_image_format">Radiance RGBE</a> (<code>.hdr</code>) file format as a source for HDR textures to <a href="https://github.com/Microsoft/DirectXTex/releases">DirectXTex</a>. The ``texconv`` and ``texassemble`` tools were updated to support <code>.hdr</code> at this time. I also added a <code>-tonemap</code> switch (using a Reinhard local operator) to the command-line tools to support conversions of HDR textures to SDR range for debugging and diagnostics.

In addition to <code>.hdr</code> file format support, I added instructions/support for opting in to support OpenEXR (<code>.exr</code>) as well. See <a href="https://github.com/Microsoft/DirectXTex/wiki/Adding-OpenEXR">Adding OpenEXR</a> for more information. Thanks to some community efforts, this is now easier to do by obtaining the required <a href="http://openexr.com/">OpenEXR</a> and <a href="http://zlib.net/">ZLib</a> libraries from NuGet.

This support is also include in the latest <a href="https://github.com/walbourn/contentexporter/releases">DirectX SDK Sample Content Exporter</a>.

<strong>Update:</strong> The 'official' header signature for the <code>.hdr</code> file format is the string <code>#?RADIANCE</code>. This is recognized by all the known readers, but it turns out a few codebases also make use of an alternative <code>#?RGBE</code> signature. While not official, it's common enough that I've updated DirectXTex in the <a href="https://github.com/Microsoft/DirectXTex/releases">February 2018 release</a> to accept either version. Note that it still writes files with <code>#?RADIANCE</code> as many readers (including the <a href="https://docs.microsoft.com/en-us/windows/desktop/directx-sdk--august-2009-">deprecated D3DX9 library</a>) don't support the alternative signature.

<strong>Note:</strong> There are some potential application compatibility issues if you use proprietary solutions such as the "NVAPI" from NVidia on newer versions of Windows 10 to enable HDR10 output. It is recommended you only make use of the proprietary APIs on systems that lack support for ``IDXGIOutput6``.
