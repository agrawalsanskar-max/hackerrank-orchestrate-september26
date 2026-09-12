# Usage & Cost Report

## Execution mode
This solution runs **entirely locally and deterministically**. No calls
were made to any external LLM, vision, or currency-rate API at runtime.

- Image-derived transaction amounts (16 values, needed to fill blank
  `amount` fields in `financial_events.csv`) were extracted **offline,
  once, ahead of time** by visual inspection of the receipt/payslip
  images and hardcoded into `IMAGE_AMOUNTS` in `build_agent.py`. No
  vision API call happens when the script runs.
- All arithmetic (currency conversion, recurrence projection, day-by-day
  balance simulation, payment-method tie-breaking) is plain Python/CSV
  processing.

## Token / API usage

| Metric | Value |
|---|---|
| API calls made at runtime | 0 |
| Input tokens consumed at runtime | 0 |
| Output tokens consumed at runtime | 0 |
| Model providers called at runtime | none |
| Total estimated runtime cost | $0.00 |

## Requests processed
- Total rows written to `output.csv`: **250**

## Notes on reproducibility
Because no network or paid API calls occur during execution, running
`python3 build_agent.py` repeatedly on the same `dataset/` folder produces
byte-for-byte identical output — the pipeline is fully deterministic.
