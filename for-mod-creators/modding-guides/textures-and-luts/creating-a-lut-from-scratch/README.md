---
description: >-
  A description of a streamlined, refined, accessible, and relatively easy
  system for creating Cyberpunk 2077 LUT mods from scratch.
cover: ../../../../.gitbook/assets/ACES_2 (1).png
coverY: 29
---

# 🌈 Creating a LUT from scratch

{% hint style="danger" %}
This method is for LDR. Little to none is known about HDR LUT creation, so please, do not expect anything on HDR LUTs.
{% endhint %}

## What is a LUT, really?

It's imperative to understand what LUT really is before starting. Often, beginners have a misconception caused by other software which often uses different formats for LUTs and different ways to apply them, such as ReShade.

Cyberpunk LUTs are distinctly different from ReShade LUTs due to two things:

1. They are 3D textures, not 2D textures with tiles describing different parts of the RGB cube, no, Cyberpunk LUTs are full-on 3D textures sampled with trilinear filtering.
2. They are applied with an input of ARRI LogC3 data with an sRGB color space. This means you can't go and use any ReShade LUTs, as they often do not expect LogC3 output. Along with this, many tools expect ARRI LogC3 data with an ARRI Wide Gamut 3 color space when generating your LUT. If you do not convert to it, you will end up with clipping colors.

But, aside from that, they are similar. Any LUT expects an input of a color, with an output of a new color. That's it. Cyberpunk's implementation is as simple as extracting the pixel out of the LUT texture that matches the input color, and if there isn't an exact match, it interpolates linearly to get a near-perfect approximation of the true output color.

If you're still confused, I recommend looking at [this](https://reverend-greg.itch.io/lut-colouring-explained).

## Prerequisites

You only need to complete this section once.

{% hint style="info" %}
You can check out nullfractal's [github template](https://github.com/nullfrctl/Realcolorr). It contains NVTT settings, DaVinci Resolve and WolvenKit projects along with LUT file templates to play around with.
{% endhint %}

{% tabs %}
{% tab title="NVIDIA Texture Tools Exporter" %}
You can't download the NVIDIA texture tools from the [official website](https://developer.nvidia.com/nvidia-texture-tools-exporter) without an **NVIDIA account** that participates in the **developer program**. We've put a backup of the executable on the wiki's [github repository](../../../../_resources_and_assets/tools/NVIDIA_Texture_Tools_2023.2.0.zip) — you need to decide which is the lesser evil, yet another account or downloading a random .exe from the internet.
{% endtab %}

{% tab title="DaVinci Resolve" %}
DaVinci's website prompts you to fill out some data before being able to download the software. You can fill this out with fake data, it doesn't really matter.

{% embed url="https://www.blackmagicdesign.com/products/davinciresolve" %}

You can use Fusion Studio just as easily as well for parts of the guide, and DaVinci Resolve Studio is also recommended in general.
{% endtab %}

{% tab title="WolvenKit" %}
The essential tool for all Cyberpunk modding. Nightly is preferred (and what this guide has been made with), but stable works just as well with newer versions (older versions will have errors).

{% embed url="https://github.com/WolvenKit/WolvenKit-nightly-releases" fullWidth="false" %}
{% endtab %}
{% endtabs %}

## Workflow

### WolvenKit

Create a new project in WolvenKit and import the file `base\weather\24h_basic\luts\cp2077_gen_lut_nge_v017.xbm` and export to a DDS, then rename it, removing the `xbm` file extension. You should have a dummy name with a file path containing that same file in the "raw" folder. You can delete the file in there, we only need the folder set up correctly.

<figure><img src="../../../../.gitbook/assets/image (25).png" alt=""><figcaption><p>How the Project Explorer should look like right now.</p></figcaption></figure>

### DaVinci Resolve

{% hint style="info" %}
If you have issues getting Resolve to run for the first time, uninstall the panels program. It causes crashes if you don't have any DaVinci panels or sliders.
{% endhint %}

1. Create a DaVinci Resolve project.
2. Go to "Fusion" tab at the bottom pane without importing any media.
3. Right click at the "Nodes" bottom pane & select "Add Tool" -> "LUT" -> "LUT Cube Creator"
4. Click on "LUTCubeCreator1" node. From the "Inspector" right pane, change the "Type" to "Vertical" and "Size" to 32, 48, or 64. Remember this number, as you will use it later.
5. On the "Nodes" pane, hover on "MediaOut1" node & click on both circles below.
6. On one of the sides, right click & select "Views" -> "3D Histogram". You should now have a LUT cube present.
7. If you'd like more accuracy, right click and go to "3D Histogram" and select "Solid", then select "Sampling" -> 1:1.

<figure><img src="../../../../.gitbook/assets/image (62).png" alt=""><figcaption><p>An example to show the current workspace should look like.</p></figcaption></figure>

8. On the "Nodes" pane right click and select "Add Tool" -> "Color" -> "ColorFX" -> "Color Space Transform" node. Set this node after "LUT Cube Creator" node.
9. Select "ColorSpaceTransform1" node. On the "Inspector" pane select these values:

<figure><img src="../../../../.gitbook/assets/image (106).png" alt=""><figcaption><p>Settings for the Color Space Transform node.</p></figcaption></figure>

(because tools expect ARRI Wide Gamut 3 with ARRI LogC3 data, but the game outputs sRGB with ARRI LogC3 gamma, so we need to go from sRGB color space with ARRI LogC3 gamma to an output color space of ARRI Wide Gamut 3 with output gamma of ARRI LogC3).

After this, we need to go from this LogC3 data to sRGB, which is what our display expects.

#### ACES method (recommended)

This is the method used by the original game to go from the input LogC3 to sRGB output, so it's what's recommended.

Add an ACES Transform node after the Color Space Transform node. Set the input transform to ARRI LogC3 with an output transform of sRGB and use ACES reference gamut compression.

<figure><img src="../../../../.gitbook/assets/image (90).png" alt=""><figcaption><p>How the workspace should look like after using the ACES method.</p></figcaption></figure>

#### ARRI LogC3 method

This method uses LUTs from ARRI themselves, and as such gives us two options: LogC3 and LogC3 "classic". I encourage you to use the normal LogC3 LUT, but experiment with Classic and see if you like it. Though, this method requires a change in the settings that are described later.

{% embed url="https://www.arri.com/en/learn-help/learn-help-camera-system/tools/lut-generator" %}

In the link above, select LogC wide gamut as the source format and destination format as video. Select your preference of ARRI Classic 709 and ARRI 709 in the conversion parameter. The file type can be either Blackmagic Fusion or Blackmagic DaVinci Resolve, it really doesn't matter.

Add a serial File LUT node and point to your downloaded file.

<figure><img src="../../../../.gitbook/assets/image (93).png" alt=""><figcaption><p>The workspace after following the directions for the LogC3 method.</p></figcaption></figure>

#### ARRI LogC4 method

It may share a name with the LogC3 method, but is distinct from it as it uses a more recent revision of the color science compared to the LogC3 method, but we will need to change things before and after this step if you decide to use this method.

Follow the same link described in the [#arri-logc3-method](./#arri-logc3-method "mention") section until you reach this part. Download the "ARRI LogC4 LUT Package".

<figure><img src="../../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption><p>The LogC4 LUTs we will use.</p></figcaption></figure>

Once you have downloaded it, extract the Rec.709 Gamma 2.4 65-point .CUBE file from the zip file. Add a serial node after the Color Space Transform called File LUT and point the LUT file to your extracted .CUBE file.

We have to make some changes to the Color Space Transform node, change the output color space to ARRI Wide Gamut 4 (instead of 3) and then the output gamma to ARRI LogC4. Your LUT should now look correctly exposed.

You can mess with the Tone Mapping Method, but just select None as a good option.

We need to change an option afterward that will be described later for both the LogC3/4 methods.

<figure><img src="../../../../.gitbook/assets/image (81).png" alt=""><figcaption><p>How your workspace should look like, including the CST settings.</p></figcaption></figure>

#### Resolve Color Managed method

This method uses DaVinci's integrated tone mappers to go from the LogC3 input to sRGB output. It produces a neutral and accurate output. It's also very minimal.

For this method to work, we just need to change the settings in the Color Space Transform.

Instead of having an output color space of ARRI Wide Gamut 3, set the color space to sRGB and gamma to sRGB as well. Then, change the tone mapping method to DaVinci, or some other method if you know what they mean.

<figure><img src="../../../../.gitbook/assets/image (34).png" alt=""><figcaption><p>Your workspace after this method. It's easy and minimal, but has very low contrast.</p></figcaption></figure>

This method creates a color cube that is very "unbiased"--it has fidelity in all directions, which, while creating a good representation of true color, can result in unnatural saturation. To account for this, the saturation compression gamut compression method can be selected.

<figure><img src="../../../../.gitbook/assets/image (30).png" alt=""><figcaption><p>The color cube after saturation compression.</p></figcaption></figure>

#### For all methods

After you have used your preferred method, you need to apply gamma correction. If you do not do this, you end up with incorrect gamma, which can mainly be seen on skin tones.

For the **ACES** method, you need to add another Color Space Transform node, with input color space and gamma of sRGB, with the output color space of sRGB but output gamma of linear.

<figure><img src="../../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption><p>ACES method after gamma correction.</p></figcaption></figure>

For both **ARRI** methods, nearly the exact same steps are taken as the **ACES** method, but, instead of sRGB input color space and gamma, we use an input color space of Rec.709, but input gamma of Gamma 2.4. Keep the output color space at sRGB and output gamma at linear.

<figure><img src="../../../../.gitbook/assets/image (45).png" alt=""><figcaption><p>LogC4 after gamma correction.</p></figcaption></figure>

For the **Resolve Color Managed** method, simply change the output gamma in the color space transform to linear, but turn on Apply Forward OOTF.

<figure><img src="../../../../.gitbook/assets/image (84).png" alt=""><figcaption><p>Gamma corrected RCM method.</p></figcaption></figure>

This is the final stage. Add another node - "Add Tool" -> "Resolve FX Transform -> "Transform" (not any other transform node) and switch on "Flip Vertical" on the right pane for correct output.

Right click on the LUT picture and save your image as a TGA or PNG file.

### NVIDIA Texture Tools (NVTT)

With preset:

1. Create text file, then open & paste this:

   ```
   --format rgba32f --quality highest --no-mips --dxt10 --extract-from-atlas --atlas-depth 64
   ```

   Save & change (rename) file's extension to `.dpf`

2. Open NVTT & load your LUT.
3. Click "Load Preset" & select your preset. Load preset **after** loading your LUT.
4. "Depth of Volume in Atlas" -> size of your LUT (same as you used above. If you forgot - open LUT as image & see its width in px - 32/48/64). Drag Z slider in top-right - should be no vertical movement.
5. "Save as" (button from bottom-right) -> select `.dds` file (or type name with `.dds`).
6. Put that `.dds` file in "source" folder of your WolvenKit project.

---

With no preset:

1. Open NVTT & load your LUT.
2. "Format" -> 32x4f
3. "Extract From Atlas" -> on
4. "Depth of Volume in Atlas" -> size of your LUT (same as you used above. If you forgot - open LUT as image & see its width in px - 32/48/64). Drag Z slider in top-right - should be no vertical movement.
5. "Regenerate Mipmaps" -> off
6. "Compression Effort" -> "Highest"
7. "DDS: Use DXT10 Header" -> on
8. "KTX2: Zstandard Supercompression" -> off
9. "Save as" (button from bottom-right) -> select `.dds` file (or type name with `.dds`).
10. Put that `.dds` file in "source" folder of your WolvenKit project.

<figure><img src="../../../../.gitbook/assets/image (76).png" alt=""><figcaption><p>NVTT correctly set up with the RCM (Resolve Color Managed) LUT in it.</p></figcaption></figure>

### WolvenKit again

1. In WolvenKit top menu click on "Tools" -> "Import Tool".
2. Select your LUT `.dds` file.
3. Set "TextureGroup" -> `TEXG_Generic_LUT`.
4. Uncheck all "IsGamma", "GenerateMipMaps", "IsStreamable", and "PremultiplyAlpha".
5. Set "Compression" -> `TCM_None`.

{% hint style="danger" %}
If your file doesn't have "RawFormat" as `TRF_HDRFloat`, then something in the DDS importing went wrong, and you need to re-set the format as 32x4f in NVTT.
{% endhint %}

<figure><img src="../../../../.gitbook/assets/image (87).png" alt=""><figcaption><p>Settings set up correctly.</p></figcaption></figure>

6. Import the DDS.
7. Double click on your imported `.xbm` file.
8. Go to `renderTextureResource` -> `renderResourceBlobPC` -> `header` -> `textureInfo` -> `type` & set `TEXTYPE_3D`.

<figure><img src="../../../../.gitbook/assets/image (18).png" alt=""><figcaption><p>Changing the texture type.</p></figcaption></figure>

9. Rename your XBM file to `cp2077_gen_lut_nge_v017.xbm`.
10. Place it in `archive/base/weather/24h_basic/luts` folder.
11. On the top menu, click "Pack Mod". You'll find `.zip` file in your project folder of your WolvenKit project (or use one from "packed" folder). Install your mod as usual.

Your LUT is correctly set up now. You can launch & try!

## Results

<div>

<figure><img src="../../../../.gitbook/assets/1.png" alt=""><figcaption><p>Vanilla</p></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/2.png" alt=""><figcaption><p>RCM</p></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/3.png" alt=""><figcaption><p>ACES</p></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/4 (1).png" alt=""><figcaption><p>LogC4</p></figcaption></figure>

</div>
