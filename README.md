# Self-organising map: code review exercise

A review of a Kohonen self-organising map (SOM) written as a take-home exercise. The brief: assess Sam's implementation for efficiency, structure, readiness for production, and anything else worth raising.

Everything is in one notebook, [kohonen/kohonen_modified.ipynb](kohonen/kohonen_modified.ipynb). It keeps Sam's original code, then works through each change with the reasoning, the code and the result. The original brief is in [kohonen/kohonen.ipynb](kohonen/kohonen.ipynb).

## What I found

Sam's code works. It is slow, only handles three-feature (RGB) data, returns a broken map on some inputs without warning, and the trained map cannot be reused or measured.

## What I changed

| # | Change | Result |
|---|---|---|
| 1 | Vectorised the grid update with NumPy, replacing two Python loops over every node | The 100x100 run went from about 142 seconds to under 2 seconds, about 72 times faster. Output matches Sam's to floating-point precision (`np.allclose`) |
| 2 | Weights per node now follow the input, and the seed and learning rate are parameters | Any number of features trains. The same seed gives the same map |
| 3 | Input validation at the start of training | Grid sides of 1 or 2 and zero iterations now raise a clear `ValueError`. Before, they returned a map whose radius never shrank, with no warning |
| 4 | Restructured into a `SOM` class with `fit`, `predict`, `save` and `load`, plotting kept separate | A trained map can be applied to new data, saved and reloaded. The interface follows scikit-learn, so other developers already know it |
| 5 | Added quantisation error and a learning curve | Error falls from 0.22 at 1 iteration to 0.01 at 500 on the demo data. Most of the gain arrives by 100 to 200 iterations |

The maths is unchanged from Sam's version. The one algebraic edit is a dropped square root, since the distance was squared again on the next line.

## Follow-up: logging and alerting

The panel asked how the map would be tested behind a website, beyond print statements. [kohonen/kohonen_logging.ipynb](kohonen/kohonen_logging.ipynb) answers that in eight steps, starting from the finished `SOM` class: the `logging` module and its levels, logging inside the class, JSON logs for a log store, an alert handler that routes `ERROR` and above to a person with deduplication, a health check on quantisation error against a held-out reference, a FastAPI endpoint with request ids, latency and a 422 versus 500 split, tests that assert on the log records, and what each piece maps to in production. Steps 1 to 5 use the standard library only. Step 6 needs `fastapi` and `httpx`.

## Not implemented, described in the notebook

- **Tests.** A pytest file for output shape, reproducibility, BMU correctness, input validation and falling error, run on every change.

- **Productionising.** First ask what the map is for, since visualisation, clustering and anomaly detection are run and monitored differently. Then package the class as a module, separate training (a batch job that saves the model) from prediction (a CLI or API that loads it), ship in Docker with tests in CI and logging, and track quantisation error on new data so a rise flags drift and a retrain.

## Run it

```bash
pip install numpy matplotlib jupyter fastapi httpx
jupyter notebook kohonen/kohonen_modified.ipynb
jupyter notebook kohonen/kohonen_logging.ipynb
```

The timing cells run Sam's original 100x100 map, which takes about two and a half minutes.

## Files

| Path | What it is |
|---|---|
| `kohonen/kohonen.ipynb` | The brief and Sam's code, as received |
| `kohonen/kohonen_modified.ipynb` | The review: observations, each change, timings and the learning curve |
| `kohonen/kohonen.png` | Network diagram from the brief |
| `kohonen/100.png`, `kohonen/1000.png` | Sam's two output maps |
| `kohonen/som_map.png`, `kohonen/som_model.npz` | Output of the `SOM` class demo: the map image and the saved model |
| `kohonen/kohonen_logging.ipynb` | The follow-up: logging, alerting, health check, FastAPI endpoint and tests |
| `kohonen/som_model_logged.npz` | Model saved by the logging notebook, loaded by its endpoint |
