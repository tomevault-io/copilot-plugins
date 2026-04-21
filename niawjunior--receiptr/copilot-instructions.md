## receiptr

> Pls follow this when implement ui shadcn and when implement ai


- use shadcn from https://ui.shadcn.com/docs/components pls check on app/components/ui no need to install again
- use sonner for toast
- use ai sdk for ai (@ai-sdk, @ai-sdk/openai, @ai-sdk/react)
- use ai sdk version 5 https://ai-sdk.dev/
- use opentyphoon for ocr https://playground.opentyphoon.ai/ocr

## opentyphoon example

async function extractTextFromImage(imageFile, apiKey, params) {
const formData = new FormData();
formData.append('file', imageFile);
formData.append('params', JSON.stringify(params));

try {
const response = await fetch('https://api.opentyphoon.ai/v1/ocr', {
method: 'POST',
headers: {
'Authorization': `Bearer ${apiKey}`
},
body: formData,
});

    if (response.ok) {
      const result = await response.json();

      // Extract text from successful results
      const extractedTexts = [];
      for (const pageResult of result.results || []) {
        if (pageResult.success && pageResult.message) {
          let content = pageResult.message.choices[0].message.content;
          try {
            // Try to parse as JSON if it's structured output
            const parsedContent = JSON.parse(content);
            content = parsedContent.natural_text || content;
          } catch (e) {
            // Use content as-is if not JSON
          }
          extractedTexts.push(content);
        } else if (!pageResult.success) {
          console.error(`Error processing ${pageResult.filename || 'unknown'}: ${pageResult.error || 'Unknown error'}`);
        }
      }

      return extractedTexts.join('\n');
    } else {
      console.error('Error:', response.status);
      const errorText = await response.text();
      console.error('Error details:', errorText);
      return null;
    }

} catch (error) {
console.error('Error:', error);
return null;
}
}

// Usage
const apiKey = '<YOUR_API_KEY>';
const imageFile = document.getElementById('imageInput').files[0]; // or PDF file
const model = "typhoon-ocr-preview";
const params = {
model: model,
task_type: "default",
max_tokens: 16000,
temperature: 0.1,
top_p: 0.6,
repetition_penalty: 1.2
};
const rawText = await extractTextFromImage(imageFile, apiKey, params);
console.log(rawText);

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/niawjunior) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
