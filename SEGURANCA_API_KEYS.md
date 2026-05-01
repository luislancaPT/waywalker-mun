# 🔐 Guia de Segurança - API Keys

## ⚠️ NUNCA Faça Isto

❌ **NÃO coloque API keys diretamente no código**
❌ **NÃO faça commit de API keys no GitHub**
❌ **NÃO compartilhe API keys por email/chat**
❌ **NÃO coloque API keys em README.md**

---

## ✅ Opção 1: Backend Proxy (MAIS SEGURO)

### **Para Produção Real:**

Crie um servidor backend que guarda as API keys secretamente.

**Arquitetura:**
```
iPhone/Browser → Seu Backend → Claude API
                    ↑
              (Keys secretas aqui)
```

### **Implementação com Cloudflare Workers (Grátis):**

1. **Criar Worker:**
```javascript
// worker.js
export default {
  async fetch(request, env) {
    // Verificar origem
    const origin = request.headers.get('Origin');
    if (origin !== 'https://seu-site.github.io') {
      return new Response('Unauthorized', { status: 403 });
    }
    
    // Pegar dados do request
    const body = await request.json();
    
    // Chamar Claude API com KEY segura
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': env.ANTHROPIC_API_KEY, // Key secreta!
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify(body)
    });
    
    return response;
  }
}
```

2. **Configurar no Cloudflare:**
   - Dashboard → Workers → Create Worker
   - Paste código acima
   - Environment Variables → Add:
     - `ANTHROPIC_API_KEY` = sua key real
   - Deploy

3. **Usar no WayWalker:**
```javascript
// Em vez de:
fetch('https://api.anthropic.com/v1/messages', {
  headers: { 'x-api-key': 'sk-ant...' } // ❌ EXPOSTO
})

// Use:
fetch('https://waywalker-proxy.seu-dominio.workers.dev', {
  // ✅ Sem API key no código!
})
```

**Vantagens:**
- ✅ Keys 100% secretas
- ✅ Controle de acesso (só seu site)
- ✅ Rate limiting fácil
- ✅ Monitoramento de uso
- ✅ Grátis (100.000 requests/dia)

---

## ✅ Opção 2: Restrições de API Key

### **Para Uso Educacional/Temporário:**

Se não pode criar backend, pelo menos **restrinja** as API keys:

### **Google Cloud Console:**
1. APIs & Services → Credentials
2. Clique na API Key
3. **Application restrictions:**
   - HTTP referrers
   - Adicione: `https://seu-site.github.io/*`
4. **API restrictions:**
   - Restrict key
   - Selecione apenas: Drive API, Maps JavaScript API
5. Save

### **Anthropic Console:**
1. https://console.anthropic.com
2. Settings → API Keys
3. **Restrictions** (se disponível):
   - Rate limits: 10 requests/hour
   - Monthly budget: $5 USD
4. **Alerts:**
   - Email se uso > $2 USD

**Com isto:**
- ❌ Ainda exposta no código
- ✅ Mas só funciona do seu domínio
- ✅ E com limites de uso

---

## ✅ Opção 3: Modo Híbrido (RECOMENDADO PARA MUN)

### **Workflow Seguro:**

1. **No Campo (iPhone):**
   - Grava vídeo + GPS + narração
   - **Export JSON** (sem usar API)
   - JSON não tem keys, só dados

2. **No Desktop (Casa/Lab):**
   - Importa JSON
   - Processa com API (ambiente controlado)
   - Publica no Atlas

**Vantagens:**
- ✅ Zero exposição de keys no mobile
- ✅ API só usada em ambiente seguro
- ✅ Mais barato (processa em batch)
- ✅ Controle total do professor

---

## 🔄 Versão Atualizada SEM API Keys

Vou criar uma versão do index.html que:
1. Remove API keys hardcoded
2. Pede ao utilizador para inserir
3. Guarda em localStorage (só no device)

### **Como usar:**
1. Abrir app
2. Settings → API Configuration
3. Inserir suas keys (uma vez)
4. Keys guardadas localmente (não vão para GitHub)

---

## 🚨 Checklist de Segurança

Antes de fazer commit no GitHub:

- [ ] Removi TODAS as API keys do código
- [ ] Criei arquivo `.gitignore`
- [ ] Adicionei `.env` ao `.gitignore`
- [ ] Keys reais só em `.env` (não commitado)
- [ ] README.md não tem keys
- [ ] index.html não tem keys hardcoded
- [ ] Verifiquei histórico Git (pode ter keys antigas)
- [ ] Se já commitei keys, revoquei e criei novas

---

## 🔍 Como Verificar se Já Expôs Keys

### **GitHub:**
```bash
# Procurar no histórico
git log --all --full-history --source --all -- '*'

# Procurar por padrões
git grep -i "sk-ant" $(git rev-list --all)
```

### **Se encontrou keys antigas:**
1. Revogue TODAS no Anthropic Console
2. Crie novas keys
3. Use método seguro (Backend ou Settings UI)

---

## 💰 O que Pode Acontecer se Keys Vazarem

**Anthropic API Key exposta:**
- ❌ Qualquer pessoa pode usar
- ❌ Gasto ilimitado na sua conta
- ❌ Pode custar $100s ou $1000s USD
- ❌ Você é responsável pelo pagamento

**Google API Keys expostas:**
- ❌ Quota esgotada rapidamente
- ❌ Possível suspensão da conta
- ❌ Dados do projeto comprometidos

---

## ✅ Ação Imediata - AGORA

1. **Pare tudo**
2. **Revogue API keys antigas** (Anthropic + Google)
3. **Remova keys do código**
4. **Crie novas keys**
5. **Use método seguro**

---

## 📞 Suporte

Se já expôs keys:
1. **Anthropic:** support@anthropic.com
2. **Google:** Console → Support
3. Explique situação, peça ajuda para revogar/proteger

---

**🔒 Segurança não é opcional - é essencial!**
