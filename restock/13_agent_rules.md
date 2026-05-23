# Agent Rules

## Core Directives
The AI Agent must adhere to these rules to ensure the system remains focused, correct, and maintainable.

### Must Do
- **Adhere to Spec**: Every feature must be built according to the provided specification documents.
- **Simplicity First**: Keep the architecture and UI minimal; avoid over-engineering.
- **Data Integrity**: Prioritize calculation correctness over visual flair.
- **Standardized Keys**: Use `ItemKey` and `StockLocationKey` for all data operations.
- **Aggregation**: Always aggregate data before performing comparisons.
- **Efficient Lookups**: Build indexes for movement data before performing evidence lookups to handle large datasets.
- **Clear Reasoning**: Every status or action generated must include a human-readable reason.

### Must Not Do
- **No ERP Creep**: Do not add features like multi-user workflows, approvals, or real-time sync.
- **No Cross-Customer Substitution**: Strictly block any auto-offsetting of stock between different customers.
- **No Raw Matching**: Never use raw item names for reconciliation; always map to keys first.
- **No Premature Finalization**: Do not mark variances as "Loss" or "Over" until T+1 data is processed.
- **No Inefficient Loops**: Avoid iterating over 50,000+ rows within nested loops; use vectorized Pandas operations.
- **No Unit Confusion**: Never use `TonQty` for generating transfer or action quantities.

## Coding Standards
- **Modularity**: Keep services small and focused on a single responsibility.
- **Decoupling**: Separate business logic (engines) from the API and UI layers.
- **Explicit Validation**: Fail fast and provide clear error messages for invalid data.
- **No Hardcoding**: Use configuration or column names; never rely on hardcoded column indices.
- **Testability**: Write code that can be easily verified against the defined test cases.
