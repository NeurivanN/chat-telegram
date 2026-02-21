# Chatbot Telegram - Temperatura de Cidades

Chatbot no Telegram que informa a temperatura atual de qualquer cidade do mundo utilizando a API OpenWeatherMap via HTTP Request no N8N.

## Funcionalidades

- Recebe o nome da cidade via mensagem no Telegram
- Consulta a temperatura atual usando o node OpenWeatherMap
- Retorna uma mensagem amigável com a temperatura em graus Celsius
- Tratamento de erros para cidades não encontradas

## Arquitetura do Workflow

```
Telegram Trigger → Verificar Mensagem ──(inválida/saudação)──→ Enviar Ajuda
                          │
                    (cidade válida)
                          ↓
                   Formatar Entrada → OpenWeather API → Validar Resposta
                                                               │
                               ┌───────────────────────────────┴──────────────────┐
                               ↓ (TRUE - cod=200)                                 ↓ (FALSE)
                         Code Fallback → Enviar Temperatura              Enviar Erro
```

## Pré-requisitos

1. **Instância N8N** (v1.0 ou superior)
2. **Bot do Telegram** (criado via @BotFather)
3. **Conta OpenWeather** (API gratuita)

## Importação do Workflow

### Passo 1: Importar o arquivo JSON

1. Acesse sua instância N8N
2. Clique em **"Add Workflow"** (ou **"+"**)
3. Selecione **"Import from File"**
4. Escolha o arquivo `workflow-chatbot-telegram.json`
5. Clique em **"Import"**

### Passo 2: Configurar Credenciais

#### Telegram Bot (Obrigatório)

1. No Telegram, abra conversa com **@BotFather**
2. Envie `/newbot` e siga as instruções
3. Copie o **Bot Token** gerado
4. No N8N, vá em **Credentials** → **Add Credential**
5. Selecione **"Telegram"**
6. Cole o **Bot Token** no campo apropriado
7. Salve como **"Telegram account"**

**Associar credencial aos nodes:**
- Clique no node **"Telegram Trigger"** → selecione a credencial
- Clique no node **"Enviar Temperatura"** → selecione a credencial
- Clique no node **"Enviar Erro"** → selecione a credencial

#### OpenWeatherMap API (Credencial Nativa do N8N)

Este workflow utiliza o sistema de credenciais nativo do N8N para armazenar a API Key com segurança.

1. Acesse [openweathermap.org](https://openweathermap.org/) e crie uma conta gratuita
2. Copie sua **API Key**
3. No N8N, vá em **Credentials → New Credential**
4. Pesquise por **"Query Auth"**
5. Configure:
   - **Name:** `OpenWeather API Key`
   - **Name** (campo do parâmetro): `appid`
   - **Value:** `sua_chave_openweather`
6. Salve e vincule ao node **"OpenWeather API"** no workflow

> **Nota:** Chaves novas da OpenWeather podem demorar até 2 horas para ativar após o cadastro.

## Ativação do Workflow

1. Após configurar todas as credenciais, clique em **"Activate"** (toggle no canto superior direito)
2. O workflow ficará escutando mensagens do Telegram

## Como Testar

### Teste 1: Cidade válida
Envie uma mensagem para seu bot no Telegram:
```
São Paulo, SP
```

**Resposta esperada:**
```
☀️ Clima em São Paulo, BR

🌡 Temperatura: 25°C
Max: 28°C   Min: 22°C
Umidade: 65%
Sensação térmica: 26°C

Céu limpo
```

### Teste 2: Outras cidades
```
Rio de Janeiro, RJ
Belo Horizonte, MG
Curitiba, PR
Salvador, BA
```

Ou apenas o nome da cidade:
```
São Paulo
Rio de Janeiro
Brasília
```

### Teste 3: Cidade inexistente
```
CidadeQueNaoExiste
```

**Resposta esperada:**
```
Por favor, informe o nome de uma cidade para consultar o clima.

Exemplo: São Paulo, SP ou Rio de Janeiro, RJ
```

## Estrutura dos Nodes

| Node | Função |
|------|--------|
| **Telegram Trigger** | Recebe mensagens do usuário |
| **Verificar Mensagem** | Valida se o texto parece um nome de cidade (filtra saudações e comandos) |
| **Formatar Entrada** | Extrai nome da cidade, converte para minúsculas e adiciona `,BR` |
| **OpenWeather API** | HTTP Request que consulta a API OpenWeather (GET) com credencial Query Auth |
| **Validar Resposta** | Verifica se `cod == 200` (sucesso) |
| **Code Fallback** | Formata a mensagem com temperatura, emoji dinâmico e dados completos |
| **Enviar Temperatura** | Envia resposta de sucesso via Telegram |
| **Enviar Ajuda** | Envia instrução de uso quando a mensagem não é uma cidade |
| **Enviar Erro** | Envia mensagem de instrução quando a cidade não é encontrada |

## Detalhes Técnicos

### Formatação da Entrada

O node **Formatar Entrada** processa o texto do usuário:

```javascript
{{ $json.message.text.trim().split(',')[0].trim() }},BR
```

**Transformações:**
- `São Paulo, SP` → `São Paulo,BR`
- `Rio de Janeiro` → `Rio de Janeiro,BR`
- `  Curitiba  ` → `Curitiba,BR`

### Code Fallback

Gera a mensagem formatada com a temperatura:

```javascript
const cidade = $json.name;
const temp = Math.round($json.main.temp);
const mensagemFallback = `🌤️ A temperatura em ${cidade} é de ${temp}°C.`;
```

## Variáveis e Credenciais Esperadas

### Variáveis de Ambiente

| Variável | Descrição | Onde Obter |
|----------|-----------|------------|
| `OPENWEATHER_API_KEY` | Chave da API OpenWeather | [openweathermap.org/api](https://openweathermap.org/api) |
| `TELEGRAM_BOT_TOKEN` | Token do bot do Telegram | [@BotFather](https://t.me/BotFather) no Telegram |

### Credenciais N8N

| Nome | Tipo | Obrigatório |
|------|------|-------------|
| `Telegram account` | `telegramApi` | Sim |
| `OpenWeather API Key` | `httpQueryAuth` (Query Auth) | Sim |

> **Importante:** Configure a credencial **Query Auth** com o parâmetro `appid` apontando para sua API Key da OpenWeather.

## Solução de Problemas

### Erro: "Cidade não encontrada" para cidades válidas
- Verifique se está digitando o nome corretamente
- Tente apenas o nome da cidade sem o estado (ex.: `São Paulo`)
- Verifique se a credencial OpenWeatherMap está configurada

### Erro: Bot não responde
- Verifique se o workflow está ativado
- Verifique se as credenciais do Telegram estão configuradas em todos os nodes
- Verifique os logs de execução no N8N

### Erro: "Invalid API key"
- Verifique se a chave OpenWeather está correta na credencial
- Chaves novas podem demorar até 2 horas para ativar

## Docker Compose (Opcional)

Se você usa Docker para rodar o N8N localmente:

```yaml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=senha_segura
      - WEBHOOK_URL=https://seu-dominio.com/
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

## Licença

Este projeto é disponibilizado para fins educacionais.

---

**Desenvolvido com N8N**
