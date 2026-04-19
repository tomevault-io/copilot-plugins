## crm-facilitaai

> * Configura o Cursor AI Agent para priorizar:

/**
 * Configura o Cursor AI Agent para priorizar:
 * - Código resiliente (anti-bugs)
 * - Boas práticas (SOLID, clean code)
 * - Sugestões de otimização
 * - Padrões do seu projeto
 */
const cursorAgentRules = {
  // 1. REGRAS DE QUALIDADE
  quality: {
    strictTypeChecking: true, // Força tipagem em TS/JSDoc
    maxFunctionLines: 20, // Alerta se função > 20 linhas
    complexityThreshold: 5, // Ciclomática máxima
    avoidPatterns: [ // Padrões proibidos
      "console.log", 
      "anyType", 
      "var "
    ],
    replacementSuggestions: { // Auto-correções
      "console.log": "logger.debug", 
      "anyType": "unknown" 
    }
  },

  // 2. CONTEXTO DO PROJETO (Atualize com seus dados!)
  projectContext: {
    framework: "Next.js", // React, Vue, etc.
    mainDependencies: ["axios", "zod", "prisma"],
    styleGuide: "https://meucrm.com/style-guide",
    filesToReference: [ // Arquivos que o Agent DEVE considerar
      "src/lib/utils.ts", 
      "src/types/index.d.ts"
    ]
  },

  // 3. APRENDIZADO CONTÍNUO
  learning: {
    analyzePastBugs: true, // Sugere fixes baseados em bugs anteriores
    trackTechDebt: true // Marca "TODO" como tech debt
  }
};

// 4. MONITORAMENTO EM TEMPO REAL (Via Cursor API)
function enableRealTimeFeedback() {
  // Hooks para eventos do editor (ex.: salvar, digitar)
  cursor.on('fileSave', (file) => {
    const suggestions = analyzeCode(file.content, cursorAgentRules);
    cursor.showSuggestions(suggestions); // Exibe no editor
  });
}

// 5. ANALISADOR DE CÓDIGO (Simulação)
function analyzeCode(code, rules) {
  const issues = [];
  
  // a. Verifica padrões proibidos
  rules.quality.avoidPatterns.forEach(pattern => {
    if (code.includes(pattern)) {
      issues.push({
        line: code.split('\n').findIndex(line => line.includes(pattern)) + 1,
        message: `Evite "${pattern}". Use: ${rules.quality.replacementSuggestions[pattern] || 'alternativa segura'}`,
        severity: "warning"
      });
    }
  });

  // b. Verifica complexidade (exemplo simplificado)
  const functionLines = code.match(/function\s+\w+\(.*\)\s*\{([\s\S]*?)\}/g);
  if (functionLines && functionLines[0].split('\n').length > rules.quality.maxFunctionLines) {
    issues.push({
      line: code.split('\n').findIndex(line => line.includes("function")),
      message: `Função muito longa (> ${rules.quality.maxFunctionLines} linhas). Quebre em funções menores.`,
      severity: "error"
    });
  }

  return issues;
}

// Ativa o agente
enableRealTimeFeedback();
console.log("🔍 Cursor AI Agent otimizado para qualidade!");

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodncb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
