---
layout: default
title: Stable diffusion In/Outpainting
nav_order: 2
---

There are several guides out there describing the process but none of the mare 100% correct, this is the jist of how to use them that I learned from experience

# Inpainting
1. Open the image in a photo editor
2. Roughly color how you want the inpainted area to look (Even just simple colors will do, AI will redraw it and just needs something to go off of)
3. Shove the image into the inpainting section of img2img
4. Use the same prompt and settings as you did when creating the image (View png info, send to img2img for an exact replica)
5. Set masked content to `original`
6. Now's the fun part, the denoising strength:\
- Setting it higher will make the AI be more creative, setting it lower will make it follow your drawing more but will leave more of it behind
- Viably the setting is anywhere 0.4-~0.7
- I found it to work best at 0.7
7. Randomize the seed and set the batch count to a few images, so you can choose the best result
8. Hit generate!

You will now get an image (or images) that is inpainted correctly.
