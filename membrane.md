---
id: membrane
title: Membrane Model
sidebar_label: Model
---

## Example membrane

```json
{
    "id": "a membrane",
    "capabilities":
    {
        "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-deuyfGpMsMSc3J8Lojq43uto2gS8zkymmRGraqPoRRqDje9RrNfhgGvjpFnnz9OkHOeKKM6lVoE9W7ogOsZ62g",
        "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-mqQ3qEMmjhw6USmeFwzbGf9KQ7biri8x7j9j454AWZ4XzzCjTOrF5Jp9_V0xZypsimcOgm5W9B4biSRsWUMcyQ"
    }
}
```

## Attributes

| Attribute | Description
| ---------- | ----------- |
| `id`       | A unique membrane id
| `capabilities` | Capabilities for interacting with the membrane

## Capabilities

| Capability | Details
| ------------ | ------- |
| `export`     | Exports new capabilities through the membrane
| `revoke`     | Revokes the entire membrane (and any previously exported capabilities)
