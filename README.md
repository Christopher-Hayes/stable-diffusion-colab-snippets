# Snippets for Stable Diffusion Colab

Repository of useful snippets for upgrading your stable diffusion colab notebook. The goal of this is to make it easy to add improvements to your notebook, without having to start fresh with a different notebook every time you want a specific feature. Snippets aren't necessarily drap+drop, knowledge of coding might be required.


## Saving to Google Drive

### Move Files to Google Driver Folder

The following moves files from Colab folder: `/content/stable-diffusion/outputs/txt2img-samples/samples/*` into Drive folder: `/content/drive/MyDrive/AI/stable_diffusion/` (Looks like "AI" > "stable_diffusion" in Drive). Finally, it deletes the files from Colab in preparation for the next run. Adjust paths to liking.

<details>
  <summary>Code</summary>
  
  ```py
  !cp -r /content/stable-diffusion/outputs/txt2img-samples/samples/* /content/drive/MyDrive/AI/stable_diffusion/ && rm /content/stable-diffusion/outputs/txt2img-samples/samples/*
  ```
</details>

## Memory Leaks

### Clear memory leaks from exceptions

Exceptions in Colab will hold all objects in memory. If you start a new run which should've fit into VRAM, but doesn't, this might be why. A trick to getting around this is to create a new exception, which will free up the memory tied to the previous exception.

<details>
  <summary>Code</summary>

  New exception to free VRAM:
  ```py
  1/0
  ```

  Clear VRAM for real this time:
  ```py
  import gc

  gc.collect()
  torch.cuda.empty_cache()
  ```
</details>

## Batching

### Prevent Colab from clearing the previous image when doing multiple runs

Note that `clear_output()` in Colab will of course clear the output produced by the code. For the dream code, this includes the image output, so remove that from the inference code if you're running a batch and want every image to show.


<details>
  <summary>Code</summary>
  
  Colab notebooks vary quite a bit. But, try looking for the code below and comment out the `clear_output()` as shown.
  
  ```py
  # display
  if opt.display_inline:
      #clear_output()
      display(Image.fromarray(grid.astype(np.uint8)))
  ```
</details>

