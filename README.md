# Snippets for Stable Diffusion Colab

Repository of useful snippets for upgrading your stable diffusion colab notebook. The goal of this is to make it easy to add improvements to your notebook, without having to start fresh with a different notebook every time you want a specific feature. Snippets aren't necessarily drap+drop, knowledge of coding might be required.


## :cloud: Saving to Google Drive

### Move Files to Google Driver Folder

The following moves files from Colab folder: `/content/stable-diffusion/outputs/txt2img-samples/samples/*` into Drive folder: `/content/drive/MyDrive/AI/stable_diffusion/` (Looks like "AI" > "stable_diffusion" in Drive). Finally, it deletes the files from Colab in preparation for the next run. Adjust paths to liking.

<details>
  <summary>Code</summary>
  
  ```py
  !cp -r /content/stable-diffusion/outputs/txt2img-samples/samples/* /content/drive/MyDrive/AI/stable_diffusion/ && rm /content/stable-diffusion/outputs/txt2img-samples/samples/*
  ```
</details>

### Using filenames that have the prompt, seed, and datetime.

Changes inference code to put the prompt in the filename, with the seed and datetime as well. Datetime is mainly added to ensure filenames are unique, but might be able to be removed without issue.

<details>
  <summary>Code</summary>

  Code to create variables for datetime and slugPrompt. A "slug" version of the prompt is created that is filename friendly. This code is in the `run_inference()` method, before the for loop stuff.
  ```py
  # * Variables for saved image filenames
  # date + time
  datetimeStr = datetime.datetime.now().isoformat()
  # Filename-safe prompt string
  slugPrompt = "".join(c if c.isalnum() else "_" for c in opt.prompt)
  ```
  
  The image file saving code with the new filename setup. Note that the prompt is limited to 150 characters. Linux has a limit of ~255 characters, need to ensure the whole filename stays under that.
  ```py
  Image.fromarray(x_sample.astype(np.uint8)).save(
      os.path.join(sample_path, f'{slugPrompt[:150]}_{opt.seed}_{datetimeStr}_{base_count:05}.png'))
  ```
  
  The same thing for grid:
  ```py
  Image.fromarray(grid.astype(np.uint8)).save(os.path.join(outpath, f'grid-{slugPrompt[:150]}_{opt.seed}_{datetimeStr}_{grid_count:04}.png'))
  ```
</details>

## :warning: Memory Leaks

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

## :computer: Batching

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

### Running a Batch of Seeds

Code to run the inference multiple times, each time with a new seed, from a comma-separated input string.

<details>
  <summary>Code</summary>

  The widget code needs to modify the seed object. Renamed "seeds", widget type is now `Text`.
  ```py
  widget_opt['seeds'] = widgets.Text(
      layout=layout, style=style,
      description='multiple seeds for batch runs (separate by comma',
      value='42',
      disabled=False
  )
  ```
  
  Modified run inference code.
  ```py
  
  # Create an object for the individual seed that mimics the Widget object
  class Option:
    def __init__(self, seed):
      self.value = seed
    def __str__(self):
      return self.value

  # Get iterable list from widget seeds string
  widgetDict = get_widget_extractor(widget_opt)
  
  # Split seeds string into individual seed values. Remove hanging empty value if it ends with a comma
  seeds = [s for s in widgetDict['seeds'].value.split(',') if s]
  
  # Run batch
  for seed in seeds:
      # Add seed to dict manually
      widgetDict['seed'] = Option(int(seed))
      # Run inference
      run(widgetDict)
      print('Done! Seed is:', seed, end='\n\n')

  print('Batch complete!')
  ```
</details>
