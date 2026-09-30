# Alerts & Runbooks

## HighLatencyP95
- **Description:** P95 Latency is greater than 3000ms for 5 consecutive minutes.
- **Impact:** Users are experiencing slow responses.
- **Mitigation:**
  1. Open Dashboard and check the Traffic panel to see if there is a sudden spike in traffic.
  2. Filter `data/logs.jsonl` for requests with `latency_ms > 3000`.
  3. Extract `correlation_id` from the slow requests.
  4. Open Langfuse Tracing, search for the `correlation_id` to inspect which span (e.g., retrieval or generation) is causing the delay.
  5. If the LLM generation is slow, consider scaling the LLM deployment. If retrieval is slow, check vector database performance.

## ElevatedErrorRate
- **Description:** System error rate exceeds 5% for 2 minutes.
- **Impact:** Users are failing to get responses, resulting in a bad experience and dropping success rate.
- **Mitigation:**
  1. Open Dashboard and review the Errors panel.
  2. Check `data/logs.jsonl` for events marked as `request_failed` and identify the `error_type`.
  3. Search Langfuse traces with the failed `correlation_id` to pinpoint the exact failure point.
  4. Check if a recent prompt version update is causing parsing errors, rollback if necessary.
  5. Check external API dependencies (e.g., LLM provider).

## LowRetrievalSuccess
- **Description:** Retrieval success rate falls below 80% for 3 minutes.
- **Impact:** The RAG system is failing to retrieve relevant context, leading to hallucinations or generic fallback answers.
- **Mitigation:**
  1. Review the Errors panel on the dashboard to see if retrieval failures are tied to specific features.
  2. Inspect Langfuse traces for the `retrieval` span to see the exact query being used.
  3. Check if the Vector DB is experiencing timeouts.
  4. Review the knowledge base index to ensure it is healthy and up-to-date.
