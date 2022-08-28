# Snippets for Stable Diffusion Colab

Repository of useful snippets for upgrading your stable diffusion colab notebook. The goal of this is to make it easy to add improvements to your notebook, without having to start fresh with a different notebook every time you want a specific feature. Snippets aren't necessarily drap+drop, knowledge of coding might be required.


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

