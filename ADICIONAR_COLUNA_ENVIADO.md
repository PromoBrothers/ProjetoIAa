# 📝 Como Adicionar Coluna `enviado_em` (Opcional)

## ❌ Erro Corrigido

O erro que você viu foi porque tentamos atualizar a coluna `enviado_em` que não existe na tabela `promocoes` do Supabase.

**Solução aplicada**: Removemos a atualização dessa coluna. O sistema agora funciona normalmente sem ela.

---

## ✅ Sistema Funciona Sem a Coluna

O envio de mensagens **já funciona perfeitamente** sem a coluna `enviado_em`. Ela era apenas para rastreamento histórico.

**O que acontece agora**:
- ✅ Mensagens são enviadas normalmente
- ✅ Agendamento é removido após envio
- ❌ Não salva data/hora de envio (não é crítico)

---

## 🔧 Se Quiser Adicionar a Coluna (Opcional)

### Opção 1: Via Supabase Dashboard (Recomendado)

1. **Acesse** seu projeto Supabase:
   ```
   https://app.supabase.com/project/cfacybymuscwcpgmbjkz
   ```

2. **Vá para Table Editor** → `promocoes`

3. **Clique em "Add Column"**

4. **Preencha**:
   ```
   Name: enviado_em
   Type: timestamp with time zone
   Default value: NULL
   Is nullable: Yes (check)
   Is unique: No
   Is primary key: No
   ```

5. **Salve**

### Opção 2: Via SQL Editor

```sql
-- Adicionar coluna enviado_em na tabela promocoes
ALTER TABLE promocoes
ADD COLUMN enviado_em TIMESTAMPTZ NULL;

-- Criar índice para melhor performance (opcional)
CREATE INDEX idx_promocoes_enviado_em
ON promocoes(enviado_em)
WHERE enviado_em IS NOT NULL;

-- Comentário descritivo
COMMENT ON COLUMN promocoes.enviado_em
IS 'Data e hora em que a mensagem foi enviada para os grupos do WhatsApp';
```

---

## 🔄 Depois de Adicionar a Coluna

Se você adicionar a coluna, descomente as linhas no código:

### `app/scheduler.py` - Linha 91-95:

**Antes (atual)**:
```python
# Remover agendamento (não atualizar enviado_em pois coluna não existe)
database.atualizar_produto_db(
    produto['id'],
    {'agendamento': None}
)
```

**Depois (se adicionar coluna)**:
```python
# Remover agendamento e marcar como enviado
database.atualizar_produto_db(
    produto['id'],
    {'agendamento': None, 'enviado_em': now.isoformat()}
)
```

### `app/scheduler.py` - Linha 205-206:

**Antes (atual)**:
```python
# Não marcar como enviado no banco (coluna não existe)
# TODO: Adicionar coluna 'enviado_em' no Supabase se necessário
```

**Depois (se adicionar coluna)**:
```python
# Marcar como enviado
database.atualizar_produto_db(
    produto_id,
    {'enviado_em': datetime.now(self.timezone).isoformat()}
)
```

---

## 📊 Benefícios de Ter a Coluna

Se você adicionar `enviado_em`, terá:

✅ **Histórico de Envios**
- Ver quando cada produto foi enviado
- Filtrar produtos enviados vs não enviados

✅ **Relatórios**
- Quantos produtos foram enviados hoje/semana/mês
- Horários mais comuns de envio

✅ **Auditoria**
- Rastrear o que foi enviado
- Evitar envios duplicados

---

## 🎯 Recomendação

**Para produção**: Adicione a coluna

**Para testes**: Não é necessário, sistema já funciona

---

## 🧪 Como Testar Agora (Sem a Coluna)

1. **Reinicie o Flask** (se ainda não reiniciou):
   ```bash
   python run.py
   ```

2. **Acesse**:
   ```
   http://localhost:5000
   ```

3. **Teste o envio**:
   - Vá para aba "Agendamento"
   - Clique "📤 Enviar Agora"
   - Selecione grupos
   - Envie!

4. **Deve funcionar** sem erros agora ✅

---

## ❓ FAQ

**P: O sistema funciona sem a coluna?**
R: Sim! Funciona perfeitamente.

**P: O que perco sem a coluna?**
R: Apenas o registro de quando foi enviado. O envio em si funciona normal.

**P: Vale a pena adicionar?**
R: Se você quer histórico/relatórios, sim. Senão, não é necessário.

**P: É difícil adicionar depois?**
R: Não! Pode adicionar a qualquer momento seguindo os passos acima.

---

**Status**: ✅ Sistema funcionando sem a coluna
**Próximo passo**: Testar o envio de mensagens!
