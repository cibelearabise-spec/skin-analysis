# Skin Analysis: análise de pele com Gemini

App mobile em HTML único (`index.html`) + uma função serverless (`api/analyze.js`).
O app tira ou escolhe a foto, o backend chama o Gemini e o resultado preenche as telas de
análise, preocupações, ingredientes e rotina.

```
skin-analysis/
├── index.html        ← o app (sem dependências externas, exceto Google Fonts)
└── api/analyze.js    ← backend (Vercel). É aqui que a chave do Gemini fica guardada
```

## 1. Ver o app sem configurar nada

Abra `index.html` no navegador e use a tela de análise, ou abra com `?demo` no final do
endereço para ir direto ao resultado de exemplo. O modo demonstração não envia nada para
lugar nenhum.

## 2. Colocar a análise real no ar (Vercel, plano gratuito)

1. **Gere a chave** em https://aistudio.google.com/apikey (entre com a conta Google e clique
   em criar chave).
2. **Suba a pasta** `skin-analysis` para um repositório no GitHub.
3. Na Vercel, clique em **Add New → Project**, escolha o repositório e importe.
   Não precisa de build: a Vercel serve o `index.html` e transforma `api/analyze.js` em
   `/api/analyze` automaticamente.
4. Em **Settings → Environment Variables**, adicione:

   | Nome | Valor | Obrigatória |
   |---|---|---|
   | `GEMINI_API_KEY` | a chave do passo 1 | sim |
   | `GEMINI_MODEL` | nome do modelo (padrão: `gemini-flash-latest`) | não |
   | `ALLOWED_ORIGIN` | domínios autorizados a chamar a API, separados por vírgula | só se o HTML ficar em outro domínio |

5. Faça um **Redeploy** para as variáveis valerem. Abra o endereço do projeto e teste com
   uma foto.

**Sobre o modelo:** os nomes dos modelos do Gemini mudam com frequência. O padrão
`gemini-flash-latest` acompanha a versão Flash mais recente. Se preferir fixar uma versão,
copie o nome exato da documentação (ai.google.dev) para `GEMINI_MODEL`. Se aparecer erro de
"modelo não encontrado", é aqui que se ajusta.

## 3. Empacotar como app Android

Como o HTML vai rodar fora do domínio da Vercel, duas coisas mudam:

1. No `index.html`, descomente a linha no primeiro `<script>` e coloque o endereço do seu
   projeto:
   ```js
   window.SKIN_API_URL = "https://SEU-PROJETO.vercel.app/api/analyze";
   ```
2. Na Vercel, preencha `ALLOWED_ORIGIN` com a origem que o WebView usa. Para conferir qual é,
   veja o erro de CORS no `chrome://inspect` com o aparelho conectado. Costuma ser
   `https://appassets.androidplatform.net` quando se usa o `WebViewAssetLoader`.

## Privacidade e limites

- **A chave do Gemini nunca vai no HTML.** Só existe na variável de ambiente do servidor.
- **A foto não é salva.** Ela é reduzida no aparelho (máx. 1024 px), enviada uma vez e mantida
  apenas em memória. O `localStorage` guarda só o último resultado em texto, os passos da
  rotina marcados e o consentimento.
- **Consentimento antes do envio.** O app pede autorização antes da primeira foto (a foto de
  rosto é dado pessoal sensível pela LGPD). O botão "Apagar dados" na tela de Perfil limpa
  tudo o que ficou no aparelho.
- **Termos do Google:** na camada gratuita da API do Gemini, os dados enviados podem ser usados
  pelo Google para melhorar os produtos deles. Para uso com pacientes ou público real,
  confira os termos vigentes e considere a camada paga, onde as condições são diferentes.
- **É leitura estética, não diagnóstico.** O prompt instrui o modelo a não diagnosticar nem
  indicar medicamentos, e a sinalizar "procure um profissional" quando notar algo suspeito.
  As notas são estimativas de IA sobre uma foto, sem valor clínico.
- **Limites de uso:** a camada gratuita tem limite de requisições por minuto e por dia. Quando
  estoura, o app mostra "Muitas análises agora". Se o app for público, considere limitar o uso
  por IP ou exigir login.
- **Limites de tamanho:** o backend recusa imagens acima de ~2,6 MB (já cobertas pela redução
  automática do app) e tipos que não sejam JPEG, PNG ou WebP.

## Como o app e o backend se falam

`POST /api/analyze` com `{ "image": "<base64 sem prefixo>", "mimeType": "image/jpeg" }`.
Resposta com sucesso: `{ valid: true, score, skinType, photoQuality, confidence, summary,
metrics, concerns, ingredients, routine, seeProfessional, professionalNote }`.
Se a foto não servir: `{ valid: false, reason }`.
