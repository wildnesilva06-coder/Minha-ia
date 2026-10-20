
import express from "express";
import OpenAI from "openai";
import path from "node:path";
import { fileURLToPath } from "node:url";

const app = express();
const port = process.env.PORT || 3000;

const dirname = path.dirname(fileURLToPath(import.meta.url));

app.use(express.json({ limit: "30kb" }));
app.use(express.static(path.join(dirname, "public")));

const client = process.env.OPENAI_API_KEY
  ? new OpenAI({ apiKey: process.env.OPENAI_API_KEY })
  : null;

app.get("/api/health", (req, res) => {
  res.json({ ok: true, configured: Boolean(client) });
});

app.post("/api/chat", async (req, res) => {
  try {
    if (!client) {
      return res.status(503).json({
        error: "A chave da API não foi configurada no servidor."
      });
    }

    const messages = req.body?.messages;

    if (
      !Array.isArray(messages) ||
      messages.length === 0 ||
      messages.length > 20
    ) {
      return res.status(400).json({
        error: "Histórico de mensagens inválido."
      });
    }

    const valid = messages.every(m =>
      m &&
      ["user", "assistant"].includes(m.role) &&
      typeof m.content === "string" &&
      m.content.length > 0 &&
      m.content.length <= 8000
    );

    if (!valid || messages[messages.length - 1].role !== "user") {
      return res.status(400).json({
        error: "As mensagens enviadas são inválidas."
      });
    }

    const result = await client.responses.create({
      model: process.env.OPENAI_MODEL || "gpt-4.1-mini",
      instructions:
        "Seu nome é Minha IA. Responda em português brasileiro, " +
        "com clareza, simpatia e honestidade. Ajude com estudos, " +
        "perguntas gerais e programação.",
      input: messages.map(m => ({
        role: m.role,
        content: m.content
      })),
      max_output_tokens: 1200
    });

    res.json({
      reply: result.output_text || "Não consegui gerar uma resposta."
    });
  } catch (error) {
    console.error("Erro na API:", error.message);
    res.status(error.status === 429 ? 429 : 500).json({
      error: error.status === 429
        ? "Limite da API atingido. Tente novamente mais tarde."
        : "Erro ao consultar a IA. Confira a configuração do servidor."
    });
  }
});

app.listen(port, "0.0.0.0", () => {
  console.log("Minha IA iniciada na porta " + port);
});
