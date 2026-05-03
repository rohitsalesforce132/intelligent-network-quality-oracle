# Copilot Instructions - Intelligent Network Quality Oracle

## Architecture
- **Framework**: Quarkus 3.34.3 for API layer, Spring AI for LLM integration
- **Pattern**: 4-layer architecture: API Gateway → Prediction Service → Data Ingestion → Storage
- **Key Classes**: `QualityPredictionResource` (REST), `PredictionService` (core logic), `ModelRouter` (AI model selection), `PredictionCache` (multi-tier caching)

## Code Conventions
- Use Quarkus reactive patterns (Uni/Multi) for non-blocking I/O
- All AI calls wrapped in circuit breakers (Resilience4j annotations)
- Every external call has a fallback (cached → historical → rule-based)
- Redis key format: `quality:prediction:{locationId}:{windowSeconds}:{featureHash}`
- Metrics use Micrometer annotations with tenant labels

## Key Patterns
- **Model Router**: Routes queries by complexity. Simple → Haiku ($0.25/M), Moderate → GPT-4o-mini ($0.60/M), Complex → GPT-4o ($15/M)
- **3-Tier Cache**: L1 (Caffeine, 30s), L2 (Redis, 60s), L3 (predictive pre-caching)
- **Advisor Pipeline**: PII scrubbing → context window management → logging → metrics
- **Structured Output**: All AI responses parsed into POJOs via Spring AI's `.entity()` method

## Testing
- Testcontainers for integration tests (PostgreSQL, Redis, Kafka)
- Mock LLM responses with WireMock
- Test cache invalidation, circuit breaker behavior, fallback chains
- Performance tests target p99 < 100ms for predictions

## Configuration
- `application.properties` for Quarkus config
- Environment variables for secrets (API keys, DB passwords)
- Kubernetes ConfigMaps for dynamic config (model routing rules, cache TTLs)
