# ProcessIQ — Control Tower

Deployment build of the ProcessIQ Control Tower, a Flask application that
assesses process maturity across five gated levels: Initial, Managed, Defined,
Measured and Optimized.

A process advances only when the weighted score for the target level reaches
80% and every preceding level has also passed. Levels come from the rule engine
and a RandomForest classifier trained at boot. The assistant narrates those
results and never produces a score of its own.

## Running the container

```bash
docker build -t processiq .
docker run -p 8000:8000 -e PROCESSIQ_PASSWORD=choose-something processiq
```

The server binds `$PORT`, defaulting to 8000.

## Configuration

| Variable | Purpose | Default |
| --- | --- | --- |
| `PROCESSIQ_USER` | Sign-in username | `admin` |
| `PROCESSIQ_PASSWORD` | Sign-in password | `processiq` |
| `PROCESSIQ_SECRET` | Flask session signing key | development key |
| `LLM_API_KEY` | Token for the hosted narrator | unset |
| `LLM_BASE_URL` | OpenAI-compatible endpoint | `https://router.huggingface.co/v1` |
| `LLM_MODEL` | Narrator model | `openai/gpt-oss-20b` |
| `CELONIS_BASE_URL` | Celonis team URL | unset |
| `CELONIS_CLIENT_ID` | OAuth client id | unset |
| `CELONIS_CLIENT_SECRET` | OAuth client secret | unset |
| `CELONIS_KM_ID` | Knowledge Model id | unset |

**Set `PROCESSIQ_USER`, `PROCESSIQ_PASSWORD` and `PROCESSIQ_SECRET` before
exposing this publicly.** The defaults exist so the app runs locally without
configuration and are not safe on a public URL.

Without `LLM_API_KEY` the assistant still answers, using the same rule-engine
facts rendered as text rather than narrated by a model. Without the Celonis
variables the integration reports itself as not configured and refuses to
attempt a pull rather than silently returning fixture data.

## Runtime behaviour

The dataset, the trained model and the live-intake thread all live in process
memory, so the container runs a **single** Gunicorn worker. A second worker
would serve a different dataset.

State is not persisted. Uploaded CSVs and records produced by the live intake
stream are lost whenever the container restarts or the host puts it to sleep;
it always boots back to the bundled dataset.
