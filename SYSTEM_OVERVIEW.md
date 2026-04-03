# FaceCheck Escolar - MVP Sistema de Reconhecimento Facial

## O que foi construído

Um sistema completo de reconhecimento facial para controle de entrada de alunos em escolas com as seguintes funcionalidades:

### ✅ Features Implementadas

1. **Cadastro de Alunos** (`/src/components/RegisterStudent.tsx`)
   - Upload de foto ou captura via câmera
   - Extração automática de "face descriptor" (128 números que representam os traços faciais)
   - Armazenamento seguro no Supabase Storage
   - Cadastro de responsáveis e números de WhatsApp

2. **Captura de Entrada em Tempo Real** (`/src/components/CheckInCapture.tsx`)
   - Câmera ativa que procura por rostos a cada 2 segundos
   - Comparação automática com banco de dados de alunos
   - Identificação com score de confiança (50%+)
   - Registro automático da entrada no banco

3. **Registro de Logs** (`/src/components/AttendanceLog.tsx`)
   - Visualização de todos os registros de entrada
   - Atualização em tempo real (Supabase Realtime)
   - Score de confiança de cada identificação

4. **Notificação WhatsApp** (Edge Function)
   - Edge Function que simula envio de WhatsApp
   - Integra dados de aluno, responsável e timestamp
   - Pronta para integrar Twilio/Evolution API

---

## Stack Tecnológico

- **Frontend**: React 18 + TypeScript + Tailwind CSS + Vite
- **Backend**: Supabase (PostgreSQL + Storage + Edge Functions)
- **Reconhecimento Facial**: Face-API.js (browser-based)
- **Cosine Similarity**: Cálculo matemático para comparação de rostos

---

## Como o Reconhecimento Facial Funciona

### 1. **Cadastro (Extração de Features)**
```
Foto do Aluno
     ↓
Face-API.js detecta o rosto
     ↓
Extrai 128 números matemáticos (face descriptor)
     ↓
Armazena no banco junto com foto
```

### 2. **Reconhecimento (Comparação)**
```
Câmera captura um novo rosto
     ↓
Face-API.js extrai novo face descriptor
     ↓
Compara com TODOS os descriptors cadastrados
     ↓
Calcula similaridade (cosine similarity)
     ↓
Se similaridade > 50% = Match! ✅
```

### 3. **Registro**
```
Match encontrado
     ↓
Cria log no banco (attendance_logs)
     ↓
Aciona Edge Function para WhatsApp
     ↓
Responsável recebe mensagem
```

---

## Banco de Dados

### Tabelas Criadas

```sql
students
├── id (UUID)
├── name (texto)
├── photo_url (URL do storage)
├── face_descriptor (JSON com 128 números)
├── active (booleano)
└── created_at

guardians
├── id (UUID)
├── name (texto)
├── phone_number (WhatsApp)
└── email

student_guardians
├── student_id
├── guardian_id
├── relationship (pai, mãe, avó)
└── is_primary (booleano)

attendance_logs
├── id (UUID)
├── student_id (FK)
├── log_type ('entry' ou 'exit')
├── confidence_score (0-100%)
├── timestamp
└── created_at
```

---

## Como Usar o Sistema

### 1️⃣ **Cadastrar um Aluno**

1. Clique em "Cadastro" no menu
2. Preencha:
   - Nome do aluno
   - Nome do responsável
   - Telefone/WhatsApp
   - Tire uma foto (câmera ou upload)
3. Clique em "Registrar Aluno"
4. Sistema extrai face descriptor automaticamente

### 2️⃣ **Registrar Entrada**

1. Clique em "Entrada" no menu
2. Deixe a câmera apontada para a criança
3. Sistema procura automaticamente a cada 2 segundos
4. Quando encontrar um match, exibe confirmação
5. Registro salvo + WhatsApp enviado

### 3️⃣ **Visualizar Registros**

1. Clique em "Logs"
2. Veja todas as entradas com:
   - Foto do aluno
   - Hora exata
   - Score de confiança

---

## Arquitetura do Projeto

```
src/
├── components/
│   ├── RegisterStudent.tsx      → Cadastro de alunos
│   ├── CheckInCapture.tsx       → Captura de entrada
│   └── AttendanceLog.tsx        → Visualização de logs
├── lib/
│   ├── supabase.ts              → Cliente Supabase
│   └── faceapi.ts               → Integração Face-API
├── App.tsx                       → Aplicação principal
└── index.css                     → Estilos globais

supabase/
└── functions/
    └── send-whatsapp-notification/
        └── index.ts              → Edge Function (WhatsApp)
```

---

## Configuração do Supabase

### Migrations Aplicadas ✅

1. `001_initial_schema` - Criação de todas as tabelas
2. `002_create_storage_bucket` - Bucket de armazenamento de fotos
3. `003_storage_policies` - Políticas de acesso ao storage

### Storage Configurado ✅

- Bucket: `school-photos`
- Acesso público para leitura
- Pronto para upload de fotos

### Edge Function Deployada ✅

- `send-whatsapp-notification` - Simula envio de WhatsApp

---

## Limitações do MVP (Por Design)

⚠️ **Essas limitações foram deixadas simples para demonstração:**

1. **RLS Aberto** - Todos podem ver todos os dados (teste apenas)
2. **WhatsApp Simulado** - Precisa integrar Twilio/Evolution API
3. **Threshold Baixo (50%)** - Aceita matches menos precisos (produção: 70%+)
4. **Câmera Contínua** - Consome mais bateria (pode ser otimizado)
5. **Um Ponto de Captura** - Versão multi-entrada é simples scale

---

## Próximos Passos para Produção

### 🔒 Segurança
- [ ] Implementar Row Level Security (RLS) restritivo
- [ ] Encriptar face descriptors no banco
- [ ] Termo de consentimento LGPD
- [ ] Auditoria de acessos

### 🚀 Performance
- [ ] Implementar cache em memória (Redis)
- [ ] Usar pgvector para busca vetorial rápida
- [ ] Code splitting para face-api.js
- [ ] Compressão de imagens

### 💬 WhatsApp
- [ ] Integrar Twilio API ou Evolution
- [ ] Configurar templates de mensagens
- [ ] Retry automático em falhas
- [ ] Tracking de entrega

### 📊 Analytics
- [ ] Dashboard com gráficos de frequência
- [ ] Relatórios por turma
- [ ] Alertas de ausência
- [ ] Integração com sistemas escolares

### 🏗️ Escalabilidade
- [ ] Multi-tenant (várias escolas)
- [ ] Múltiplos pontos de captura
- [ ] Fila de processamento
- [ ] Monitoramento com Sentry

---

## Troubleshooting

### ❌ "Nenhum rosto detectado"
- Aumentar iluminação
- Acertar ângulo da câmera
- Tirar óculos escuro/viseira
- Tentar com outra foto

### ❌ "Acesso negado ao storage"
- Verificar se bucket `school-photos` existe
- Verificar RLS policies do storage
- Limpar cache do navegador

### ❌ "Edge Function erro"
- Verificar logs no Supabase Dashboard
- Conferir variáveis de ambiente
- Testar com curl manualmente

### ❌ "Câmera não funciona"
- Permitir acesso à câmera no navegador
- Usar HTTPS (não funciona com HTTP em produção)
- Tentar em navegador diferente

---

## Custos Estimados (Mensais)

| Serviço | Uso | Custo |
|---------|-----|-------|
| Supabase Pro | 100 alunos | $25 |
| Storage | 100 fotos | $1 |
| Edge Functions | 5000 chamadas | $1 |
| Bandwidth | 10GB | $1 |
| **Total** | | **~$30/mês** |

---

## Referências

- [Face-API.js Docs](https://github.com/vladmandic/face-api)
- [Supabase Docs](https://supabase.com/docs)
- [Cosine Similarity](https://en.wikipedia.org/wiki/Cosine_similarity)
- [LGPD Brasil](https://www.gov.br/cidadania/pt-br/acesso-a-informacao/lgpd)

---

## Licença

MIT - Use livremente, mas considere a LGPD ao coletar dados de menores!
