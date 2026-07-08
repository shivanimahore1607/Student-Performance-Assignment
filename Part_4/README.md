# Part 4 - Tabular Record Batch Scoring

इस प्रोजेक्ट में मैंने **Track B: Tabular Record Batch Scoring** को पूरा किया है।

## 1. System Prompt & Design Decisions
मैंने एक `system_prompt` डिज़ाइन किया है जो क्रेडिट स्कोरिंग के लिए काम करता है।

**System Prompt (verbatim):**
You are a credit scoring assistant.
Return the output STRICTLY as a valid JSON object with these 5 fields: 
{"risk_tier": "low|medium|high", "confidence": "low|medium|high", "recommended_action": "string", "primary_signal": "string", "flag_for_review": "true|false"}.

RULES:
1. Return ONLY the JSON object.
2. DO NOT include any introductory or concluding text.
3. DO NOT use markdown formatting.

## 2. Temperature Rationale
*   **Temperature 0.0:** इसे फाइनल स्कोरिंग के लिए रखा है ताकि आउटपुट स्थिर (stable) रहे।
*   **Temperature 0.7:** इसे तुलना करने के लिए यूज़ किया ताकि देखा जा सके कि आउटपुट में कितना बदलाव आता है।

### Temperature A/B Comparison
| Temperature | Observation |
| :--- | :--- |
| 0.0 | Consistent & Valid JSON |
| 0.7 | More creative but less predictable |

## 3. Guardrail Test Results
मैंने एक PII गार्डरेल बनाया है जो ईमेल और फोन नंबर वाले इनपुट को ब्लॉक कर देता है।
*   **Test 1 (PII present):** Input blocked successfully.
*   **Test 2 (Clean input):** Processed successfully.

## 4. End-to-End Demonstration Table
यहाँ हमारे तीन रिकॉर्ड्स का बैच स्कोरिंग परिणाम है:

| Input | LLM Output | Validation Status |
| :--- | :--- | :--- |
| {"Hours_Studied": 23, ...} | {"risk_tier": "high", ...} | Pass |
| {"Hours_Studied": 19, ...} | {"risk_tier": "high", ...} | Pass |
| {"Hours_Studied": 24, ...} | {"risk_tier": "low", ...} | Pass |