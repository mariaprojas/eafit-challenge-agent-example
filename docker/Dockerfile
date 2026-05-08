FROM io2060/hologram-generic-ai-agent-app:v1.12.0

# Parche 1: modelo de Anthropic
RUN sed -i "s/modelName: this.config.get('appConfig.anthropicModel') ?? 'claude-3'/modelName: process.env.ANTHROPIC_MODEL ?? 'claude-haiku-4-5-20251001'/g" /app/dist/llm/llm.service.js

# Parche 2: topP/topK
RUN FILE="/app/node_modules/.pnpm/@langchain+anthropic@0.3.33_@langchain+core@0.3.79_openai@4.104.0_ws@8.18.3_zod@3.25.76___zod@3.25.76/node_modules/@langchain/anthropic/dist/chat_models.js" && sed -i '/Object.defineProperty(this, "topP"/{n;n;n;n;s/value: -1/value: undefined/}' "$FILE" && sed -i '/Object.defineProperty(this, "topK"/{n;n;n;n;s/value: -1/value: undefined/}' "$FILE"
RUN FILE="/app/node_modules/.pnpm/@langchain+anthropic@0.3.33_@langchain+core@0.3.79_openai@4.104.0_ws@8.18.3_zod@3.25.76___zod@3.25.76/node_modules/@langchain/anthropic/dist/chat_models.cjs" && sed -i '/Object.defineProperty(this, "topP"/{n;n;n;n;s/value: -1/value: undefined/}' "$FILE" && sed -i '/Object.defineProperty(this, "topK"/{n;n;n;n;s/value: -1/value: undefined/}' "$FILE"

# Parche 3: output de Claude y memoria
RUN printf '%s\n' \
  "const fs = require('fs');" \
  "const p1 = '/app/dist/memory/langchain-session-memory.js';" \
  "let c1 = fs.readFileSync(p1, 'utf8');" \
  "c1 = c1.replace(\"const aiOutput = (output.output ?? output.response ?? '');\", \"const aiOutput = typeof (output.output ?? output.response ?? '') === 'string' ? (output.output ?? output.response ?? '') : JSON.stringify(output.output ?? output.response ?? '');\");" \
  "c1 = c1.replace(\"return m.role === 'user' ? new messages_1.HumanMessage(m.content) : new messages_1.AIMessage(m.content);\", \"const content = typeof m.content === 'string' ? m.content : JSON.stringify(m.content); return m.role === 'user' ? new messages_1.HumanMessage(content) : new messages_1.AIMessage(content);\");" \
  "fs.writeFileSync(p1, c1);" \
  "const p2 = '/app/dist/llm/llm.service.js';" \
  "let c2 = fs.readFileSync(p2, 'utf8');" \
  "c2 = c2.replace(\"this.logger.warn('Agent executor returned a non-string result. Returning JSON stringified result.');\\n                return this.sanitizeResponse(JSON.stringify(output));\", \"if (Array.isArray(output)) { const text = output.filter(b => b.type === 'text').map(b => b.text).join('\\\\n'); if (text) return this.sanitizeResponse(text); }\\n                this.logger.warn('Agent executor returned a non-string result. Returning JSON stringified result.');\\n                return this.sanitizeResponse(JSON.stringify(output));\");" \
  "fs.writeFileSync(p2, c2);" \
  "console.log('ALL PATCHES DONE');" \
  > /tmp/fix.js && node /tmp/fix.js

# Parche 4 NUEVO: hacer que el RAG no falle cuando el vector store está vacío
# En lugar de lanzar error al buscar, devuelve array vacío silenciosamente
RUN node -e " \
  const fs = require('fs'); \
  const p = '/app/dist/rag/langchain-rag.service.js'; \
  let c = fs.readFileSync(p, 'utf8'); \
  c = c.replace( \
    'throw new Error(\`Unsupported VECTOR_STORE: \${JSON.stringify(vectorStoreProvider)}\`)', \
    'this.logger.warn(\`Unsupported VECTOR_STORE: \${JSON.stringify(vectorStoreProvider)} — RAG disabled\`); this.vectorStore = null; return' \
  ); \
  fs.writeFileSync(p, c); \
  console.log('RAG fallback patch applied'); \
"
