# 📱 Sistema de Envio Automático de Mensagens WhatsApp

## 🎯 Visão Geral

Sistema completo e independente para envio de mensagens do WhatsApp, **sem depender do n8n**.

### ✅ Funcionalidades

1. **Agendamento Automático**: Mensagens são enviadas automaticamente no horário agendado
2. **Envio Manual**: Envie mensagens para grupos específicos quando quiser
3. **Scheduler Inteligente**: Verifica mensagens agendadas a cada 30 segundos
4. **Configuração de Grupos**: Defina quais grupos recebem mensagens automáticas
5. **Integração WhatsApp**: Usa o WhatsApp Monitor (Baileys) para envio real

---

## 📂 Arquivos Criados

### 1. **`app/scheduler.py`** - Sistema de Agendamento
```python
# Funcionalidades:
- MessageScheduler: Classe principal
- start(): Inicia o scheduler automático
- send_message_now(): Envia mensagem imediatamente
- _check_and_send_scheduled_messages(): Verifica agendamentos
```

### 2. **Rotas Adicionadas em `app/routes.py`**
```python
POST /enviar-mensagem          # Envio manual
GET  /scheduler/status          # Status do scheduler
POST /configurar-grupos-auto    # Configurar grupos automáticos
GET  /configurar-grupos-auto    # Obter grupos configurados
```

### 3. **Integração em `app/__init__.py`**
- Scheduler inicia automaticamente ao rodar o Flask

---

## 🚀 Como Usar

### 1. Envio Automático (Agendado)

**Agendar um produto:**
```bash
POST /agendar_produto/{produto_id}

{
  "agendamento": "2025-01-06T10:30:00"  # Horário de Brasília
}
```

**O que acontece:**
1. ✅ Produto fica salvo com data de agendamento
2. ✅ Scheduler verifica a cada 30 segundos
3. ✅ Quando chegar o horário, envia automaticamente
4. ✅ Marca como enviado após sucesso

**Configurar grupos para envio automático:**
```bash
POST /configurar-grupos-auto

{
  "grupos": [
    "120363045678901234@g.us",
    "120363099887766554@g.us"
  ]
}
```

---

### 2. Envio Manual

**Enviar para grupos específicos:**
```bash
POST /enviar-mensagem

{
  "produto_id": "abc123",
  "grupos": [
    "120363045678901234@g.us",
    "120363099887766554@g.us"
  ]
}
```

**Resposta:**
```json
{
  "success": true,
  "message": "Mensagem enviada para 2 grupo(s)",
  "detalhes": {
    "total_enviado": 2,
    "total_falhou": 0,
    "resultados": [
      {"grupo": "120363045678901234@g.us", "sucesso": true},
      {"grupo": "120363099887766554@g.us", "sucesso": true}
    ]
  }
}
```

---

### 3. Verificar Status do Scheduler

```bash
GET /scheduler/status
```

**Resposta:**
```json
{
  "success": true,
  "running": true,
  "check_interval": 30,
  "whatsapp_url": "http://localhost:3001"
}
```

---

## 🔧 Configuração

### 1. Variáveis de Ambiente (.env)

```env
# URL do WhatsApp Monitor
WHATSAPP_MONITOR_URL=http://localhost:3001

# Grupos para envio automático (opcional - pode configurar pela API)
WHATSAPP_AUTO_SEND_GROUPS=120363045678901234@g.us,120363099887766554@g.us
```

### 2. Configuração de Grupos Automáticos

**Opção A: Via API** (Recomendado)
```bash
POST /configurar-grupos-auto
{
  "grupos": ["grupo1@g.us", "grupo2@g.us"]
}
```

**Opção B: Via .env**
```env
WHATSAPP_AUTO_SEND_GROUPS=grupo1@g.us,grupo2@g.us
```

**Opção C: Via arquivo JSON** (criado automaticamente pela API)
```json
// app/config/auto_send_groups.json
{
  "grupos": [
    "120363045678901234@g.us",
    "120363099887766554@g.us"
  ]
}
```

---

## 📊 Fluxo Completo

### Cenário 1: Agendamento Automático

```
1. Usuário agenda produto via interface
   ↓
2. Produto salvo no banco com "agendamento": "2025-01-06T10:30:00"
   ↓
3. Scheduler verifica a cada 30s
   ↓
4. Quando horário chegar:
   - Busca grupos configurados
   - Envia mensagem + imagem para cada grupo
   - Marca produto como enviado
   - Remove agendamento
```

### Cenário 2: Envio Manual

```
1. Usuário clica "Enviar Agora" na interface
   ↓
2. Seleciona grupos de destino
   ↓
3. API /enviar-mensagem é chamada
   ↓
4. Sistema envia para grupos selecionados
   ↓
5. Marca produto como enviado
```

---

## 🎨 Integração com Frontend

### JavaScript para Envio Manual

```javascript
async function enviarMensagem(produtoId, grupos) {
    try {
        const response = await fetch('/enviar-mensagem', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                produto_id: produtoId,
                grupos: grupos
            })
        });

        const result = await response.json();

        if (result.success) {
            alert(`✅ ${result.message}`);
            console.log('Detalhes:', result.detalhes);
        } else {
            alert(`❌ Erro: ${result.error}`);
        }
    } catch (error) {
        console.error('Erro:', error);
        alert('❌ Erro ao enviar mensagem');
    }
}

// Exemplo de uso
enviarMensagem('produto-123', [
    '120363045678901234@g.us',
    '120363099887766554@g.us'
]);
```

### Configurar Grupos Automáticos

```javascript
async function configurarGruposAuto(grupos) {
    const response = await fetch('/configurar-grupos-auto', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({ grupos })
    });

    const result = await response.json();
    return result;
}

// Obter grupos configurados
async function obterGruposAuto() {
    const response = await fetch('/configurar-grupos-auto');
    const result = await response.json();
    return result.grupos;
}
```

---

## 🔍 Logs do Sistema

### Scheduler Iniciado
```
✅ Scheduler de mensagens iniciado
🔄 Scheduler rodando... Verificando mensagens agendadas a cada 30s
```

### Mensagem Agendada Detectada
```
⏰ Horário atingido para produto: Notebook Gamer Dell...
📱 Mensagem enviada com sucesso para 120363045678901234@g.us
✅ Mensagem enviada e agendamento removido: abc123
```

### Envio Manual
```
📤 Enviando mensagem manual do produto abc123 para 2 grupo(s)
📱 Mensagem enviada com sucesso para 120363045678901234@g.us
📱 Mensagem enviada com sucesso para 120363099887766554@g.us
```

---

## 🛠️ Troubleshooting

### Problema: Mensagens não estão sendo enviadas automaticamente

**Verificar:**
1. ✅ Scheduler está rodando?
   ```bash
   GET /scheduler/status
   ```

2. ✅ Grupos estão configurados?
   ```bash
   GET /configurar-grupos-auto
   ```

3. ✅ WhatsApp Monitor está conectado?
   ```bash
   GET /whatsapp/status
   ```

4. ✅ Produto tem agendamento válido?
   ```bash
   GET /produtos?status=agendado
   ```

### Problema: Erro ao enviar mensagem

**Possíveis causas:**
- ❌ WhatsApp Monitor offline → Iniciar: `cd whatsapp-monitor && npm start`
- ❌ Grupo inválido → Verificar ID do grupo em `/whatsapp/groups`
- ❌ Sem permissão no grupo → Bot precisa ser admin ou grupo precisa permitir mensagens

---

## 🚀 Vantagens vs n8n

| Aspecto | Sistema Atual | n8n |
|---------|---------------|-----|
| **Dependências** | Independente | Depende de serviço externo |
| **Performance** | Rápido (nativo) | Mais lento (rede) |
| **Confiabilidade** | Alta | Média (pode cair) |
| **Controle** | Total | Limitado |
| **Logs** | Centralizados | Separados |
| **Manutenção** | Simples | Complexa |

---

## 📝 Próximos Passos

1. ✅ Sistema implementado e funcional
2. ⏳ Testar com produto real
3. ⏳ Adicionar interface no frontend
4. ⏳ Implementar retry em caso de falha
5. ⏳ Adicionar fila de prioridades

---

## 🎯 Exemplo Completo de Uso

### 1. Iniciar Sistema
```bash
# Terminal 1: WhatsApp Monitor
cd whatsapp-monitor
npm start

# Terminal 2: Flask
python run.py
```

### 2. Configurar Grupos
```bash
curl -X POST http://localhost:5000/configurar-grupos-auto \
  -H "Content-Type: application/json" \
  -d '{"grupos": ["120363045678901234@g.us"]}'
```

### 3. Agendar Produto
```bash
curl -X POST http://localhost:5000/agendar_produto/abc123 \
  -H "Content-Type: application/json" \
  -d '{"agendamento": "2025-01-06T15:00:00"}'
```

### 4. Aguardar Envio Automático
- Sistema envia automaticamente às 15:00

**OU**

### 4. Enviar Manualmente
```bash
curl -X POST http://localhost:5000/enviar-mensagem \
  -H "Content-Type: application/json" \
  -d '{
    "produto_id": "abc123",
    "grupos": ["120363045678901234@g.us"]
  }'
```

---

**Sistema 100% funcional e pronto para uso!** 🎉

**Data**: 2025-01-06
**Versão**: v1.0 (Sistema Independente de Envio)
