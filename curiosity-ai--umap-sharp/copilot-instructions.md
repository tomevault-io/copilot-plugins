## umap-sharp

> UMAP C# is a C# reimplementation of the UMAP (Uniform Manifold Approximation and Projection) algorithm for non-linear dimensionality reduction. It takes high-dimensional vectors and produces 2D/3D (or higher) embeddings suitable for visualization or downstream ML tasks.

# AGENTS

## Repository overview
UMAP C# is a C# reimplementation of the UMAP (Uniform Manifold Approximation and Projection) algorithm for non-linear dimensionality reduction. It takes high-dimensional vectors and produces 2D/3D (or higher) embeddings suitable for visualization or downstream ML tasks.

## Key concepts
- **Embedding**: The lower-dimensional representation produced from your input vectors.
- **Neighbor graph / fuzzy manifold**: UMAP builds a k-nearest-neighbor graph and optimizes an embedding that preserves local structure.
- **Epochs**: Optimization iterations. The recommended number of epochs is returned by `InitializeFit`.
- **Distance function**: Controls how similarity between vectors is measured (e.g., cosine distance).
- **Random generator**: Controls stochastic behavior and optional multi-threading.

## Usage styles
- **One-shot embedding**: Initialize, run the recommended number of steps, then read the embedding.
- **Custom epochs / iterative refinement**: Provide a custom epoch count in the constructor or call `Step` manually in a loop.
- **Custom distance function**: Provide your own distance function to match your data characteristics.

## Key methods and functionality
- `Umap.InitializeFit(float[][] vectors)`: Prepares the model and returns the recommended number of epochs.
- `Umap.Step()`: Performs one optimization step; call repeatedly to reach the desired number of epochs.
- `Umap.GetEmbedding()`: Returns the final embedding vectors in the same order as the input.
- Constructor options: dimensions, distance function, random generator, number of neighbors, custom number of epochs, and progress reporter.

## Typical workflow
1. Create `Umap` with desired configuration.
2. Call `InitializeFit` with your `float[][]` vectors to build the graph and get the epoch count.
3. Call `Step` for the recommended (or custom) number of epochs.
4. Call `GetEmbedding` to retrieve the reduced-dimension vectors.

## Parallelization notes
If you provide a thread-safe random generator, optimization steps may run multi-threaded for performance. To force single-threaded execution, use the provided option to disable threading on the default generator.

---
> Source: [curiosity-ai/umap-sharp](https://github.com/curiosity-ai/umap-sharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
