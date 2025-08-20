# PSW — Assessment de Perfis Comportamentais

Projeto estático (HTML puro) para aplicar o **PSW** com 30 pares de características, escala **1–4**, cálculo automático de **perfil** e impressão em **PDF**.

**Produção:** https://maisperformance.github.io/PSW/  
**DEV (testes):** https://maisperformance.github.io/PSW/index.dev.html

> A versão **DEV** tem painel de preenchimento automático e meta-tags `noindex, nofollow` para evitar indexação.

---

## ✨ Recursos

- Escala **1–4** em formato _segmented control_ (sempre em **linha**).
- **Progresso** (X/30) + **barra** com botões fixos (sticky): Ver Resultado, Imprimir/Salvar PDF, Recomeçar, Modo apresentação.
- **Resultado** com somas e descrição do perfil; inicia **em nova página** no PDF.
- **Modo apresentação**: oculta perguntas/identificação/auto-testes, mostra só o resultado.
- **Acessibilidade**: `<fieldset>` + `<legend>`, `label → input`.
- **Privacidade**: não envia nem armazena dados por padrão.
- **CSP** (Content Security Policy) básica para segurança.
- **Auto-testes** para validar a regra do perfil.
- **DEV panel** (somente em `index.dev.html`) para preencher cenários de teste com 1 clique.

---

## 🧮 Como funciona a pontuação e o perfil

- **Soma Linhas Brancas** (ids: 1,3,5,…,29) → `sumWhite`  
- **Soma Linhas Cinzas** (ids: 2,4,6,…,30) → `sumGray`  
- **“Alto” é apenas `> 36`** (36 **não** conta como alto).

**Mapeamento:**

| Condição                                         | Perfil       |
|--------------------------------------------------|--------------|
| `sumWhite ≤ 36` **e** `sumGray ≤ 36`             | Diplomático  |
| `sumWhite ≤ 36` **e** `sumGray > 36`             | Analítico    |
| `sumWhite > 36` **e** `sumGray > 36`             | Pragmático   |
| `sumWhite > 36` **e** `sumGray ≤ 36`             | Expressivo   |

As somas são exibidas no cartão de resultado.

---

## 🗂 Estrutura

```
PSW/
├─ index.html            # produção
├─ index.dev.html        # dev (ferramentas de teste + noindex)
└─ README.md             # este arquivo
```

> **Produção (index.html):** sem painel DEV.  
> **DEV (index.dev.html):** inclui botões para “Preencher — Diplomático/Analítico/Pragmático/Expressivo” e **Auto-testes** visíveis.

---

## ▶️ Rodando localmente

- Basta **abrir** `index.html` no navegador.  
- Opcional: use _Live Server_ no VS Code para recarregar automaticamente.

---

## 🧪 Testes rápidos

Sem responder tudo, você pode testar as regras:

- **Diplomático** → Brancas ≤ 36 e Cinzas ≤ 36  
  (Ex.: marque 8 **brancas** com **4** = 32 e 8 **cinzas** com **4** = 32)
- **Analítico** → Brancas ≤ 36, **Cinzas > 36**  
  (Ex.: 10 **cinzas** com **4** = 40)
- **Pragmático** → **Brancas > 36** e **Cinzas > 36**  
  (Ex.: 10 **brancas** com **4** = 40; 10 **cinzas** com **4** = 40)
- **Expressivo** → **Brancas > 36**, Cinzas ≤ 36  
  (Ex.: 10 **brancas** com **4** = 40; cinzas vazias/baixas)

Na **DEV** (`index.dev.html`), use os botões do painel para preencher automaticamente cada cenário e validar.

---

## 🖨 PDF / Impressão

- Clique **Imprimir / Salvar PDF**.  
- Defina **A4** e desative cabeçalhos/rodapés do navegador.  
- O **Resultado** inicia automaticamente em **nova página**.  
- As perguntas evitam quebra no meio (`break-inside: avoid`).

---

## 🎛 Personalização

- **Título do projeto:** editável na tag `<title>` e `<h1>` (já “PSW - Assessment de Perfis Comportamentais”).  
- **Logo:** dentro do HTML (`<img class="logo">`).  
  - Você pode substituir o `src` por um **link público** (PNG/SVG) da sua marca.  
- **Cores:** altere variáveis CSS em `:root`:
  ```css
  --magenta: #D00E56;
  --purple: #332984;
  --text: #e7ebff;
  ```
- **Textos do resultado**: objeto `DESCRIPTIONS` no `<script>`.

---

## 🚀 Deploy

### GitHub Pages (usado aqui)
1. `index.html` na **raiz** do repositório (branch `main`).  
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch**  
   **Branch:** `main` • **Folder:** `/ (root)`  
3. A URL padrão será:  
   `https://<seu-usuario>.github.io/<nome-do-repo>/`  
   - DEV: `…/index.dev.html`

> Se algo não aparecer, verifique **Actions → pages build and deployment** e/ou faça **hard refresh** (Ctrl/Cmd+Shift+R).  
> Se quiser, adicione um arquivo **`.nojekyll`** vazio na raiz.

### Netlify (alternativa)
- **Importar do GitHub** (deploy contínuo): **Build command** em branco, **Publish directory** = `.`.  
- **Deploy manual**: arraste uma pasta/ZIP com `index.html` na **raiz**.

---

## 🔌 Próximo passo: Integração com **n8n** (opcional)

Para enviar os resultados para um **Webhook** no n8n, acrescente no final da função `showResult()`:

```html
<script>
  // ...
  // dentro de showResult(), após montar resultDiv.innerHTML:
  fetch('https://SEU-N8N/webhook/psw', {
    method: 'POST',
    headers: {'Content-Type':'application/json'},
    body: JSON.stringify({
      nome, data,
      respostas: Array.from(responses.entries()), // [[id,valor],...]
      sumWhite: r.sumWhite, sumGray: r.sumGray, perfil: r.perfil,
      origem: location.href, timestamp: new Date().toISOString()
    })
  }).catch(console.error);
</script>
```

> **CORS no n8n:** autorize seu domínio (`N8N_CORS_ORIGINS=https://SEU-SITE.github.io`).  
> **Fluxo n8n:** Webhook → (If por perfil) → (Google Sheets/Airtable/Notion/Email/PDF) → (HTTP Response opcional).

---

## 🔒 Segurança & Privacidade

- **Sem coleta** por padrão (somente Nome/Data visuais).  
- **CSP** aplicada (`default-src 'self'` etc.).  
- A versão **DEV** usa `noindex, nofollow` + `canonical` para a principal.
- Se futuramente extrair JS/CSS para arquivos externos, ajuste a **CSP** (ideal: remover `unsafe-inline` e usar `nonce` ou arquivos externos com `sha256`).

---

## 🛠 Solução de problemas

- **404 / Página branca:** verifique se `index.html` está na **raiz**.  
- **Mudança não aparece:** aguarde o **build do Pages** (Actions) e faça **hard refresh**.  
- **Quebra estranha no PDF:** o layout evita quebras em perguntas; se o conteúdo for muito, diminua paddings no `@media print`.

---

## 🗺 Roadmap (sugestões)

- Tema claro/escuro (toggle).  
- Botão “Copiar link de resultado” (com hash das respostas).  
- PWA (offline): `manifest.json` + ícones.  
- Geração de **PDF customizado** com cabeçalho da empresa (via n8n).

---

## 📝 Licença

Defina a licença conforme sua política (ex.: **Todos os direitos reservados** / **Privado**).

---

## 🤝 Contribuição

- Para ajustes: crie uma **branch** e abra um **Pull Request**.  
- Mantenha a regra do perfil ( > 36 ) e não altere os **Auto-testes** sem necessidade.
