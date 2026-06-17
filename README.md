// api/chat.js — Backend seguro para o App Calendário Eleitoral 2026
// Este arquivo roda no servidor (Vercel). A chave de API NUNCA é exposta ao navegador.

export default async function handler(req, res) {
  // Aceita apenas POST
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Método não permitido' });
  }

  // Pega a chave da variável de ambiente (configurada no painel do Vercel)
  const apiKey = process.env.ANTHROPIC_API_KEY;
  if (!apiKey) {
    return res.status(500).json({ error: 'Chave de API não configurada no servidor.' });
  }

  try {
    const { messages, system } = req.body;

    // Chama a API da Anthropic do lado do servidor
    // O system prompt é enviado com cache_control para ativar o prompt caching da Anthropic.
    // Isso reduz o custo em até 90% nas chamadas repetidas, pois o system prompt (fixo e grande)
    // fica armazenado em cache por até 5 minutos entre as requisições.
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': apiKey,
        'anthropic-version': '2023-06-01',
        'anthropic-beta': 'prompt-caching-2024-07-31'
      },
      body: JSON.stringify({
        model: 'claude-haiku-4-5-20251001',
        max_tokens: 1000,
        system: [
          {
            type: 'text',
            text: system,
            cache_control: { type: 'ephemeral' }
          }
        ],
        messages: messages
      })
    });

    const data = await response.json();

    if (!response.ok) {
      return res.status(response.status).json({ error: data.error?.message || 'Erro na API da Anthropic' });
    }

    return res.status(200).json(data);
  } catch (error) {
    return res.status(500).json({ error: 'Erro interno do servidor: ' + error.message });
  }
}
