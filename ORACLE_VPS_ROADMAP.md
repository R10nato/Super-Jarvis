# Roadmap: Super Jarvis + Ollama na Oracle Cloud VPS

## 🎯 Objetivo
Configurar o Super Jarvis (OpenJarvis + Hermes Agent fusionado) com modelo Ollama em uma VPS Oracle Cloud gratuita.

### 🇧🇷 **Vantagem da Região São Paulo:**
A VPS está na região `sa-saopaulo-1`, o que oferece:
- **Menor latência** para usuários no Brasil
- **Respostas mais rápidas** em comparação com regiões internacionais
- **Conformidade com LGPD** (dados hospedados no Brasil)
- **Performance otimizada** para o público brasileiro

## 🤖 **Modelo Escolhido: Qwen2.5:7b**

**Por que Qwen2.5:7b é ideal para o Super Jarvis:**

### ✅ **Excelente para Automações**
- Processamento avançado de comandos complexos
- Melhor compreensão de instruções multi-etapas
- Execução confiável de automações programadas
- Suporte avançado para scripts e APIs

### ✅ **Superior Comunicação**
- Diálogos naturais e fluentes em português
- Capacidade de contexto mais longo (32K tokens)
- Respostas mais coesas e inteligentes
- Melhor compreensão de nuances e contexto

### ✅ **Otimizado para x86**
- Eficiente em arquitetura x86 da Oracle Cloud
- 32GB RAM oferece excelente performance
- 2 OCPU suficiente para operações do Super Jarvis
- Região São Paulo oferece menor latência para o Brasil
- Aproveita bem os 4 OCPU disponíveis

### ✅ **Capacidades Avançadas**
- 7B parâmetros - equilíbrio ideal entre capacidade e performance
- Suporte a múltiplos idiomas (excelente em português)
- Excelente em raciocínio e análise complexa
- Perfeito para Super Jarvis com skills avançadas de Hermes
- Contexto longo ideal para context references e memória

### ✅ **Recursos da VPS Oracle x86**
- 32GB RAM = mais que suficiente para Qwen2.5:7b (~4-5GB) + qwen2.5:3b backup (~2-3GB) + Super Jarvis (~2-3GB)
- 2 OCPU = bom desempenho para o modelo (32GB RAM compensa menos OCPU)
- SSD Oracle Cloud = acesso rápido ao modelo
- Total usado: ~8-11GB RAM, deixando 21-24GB livres para cache e sistema
- Região São Paulo = menor latência para o Brasil

### ✅ **Modelo Backup: Qwen2.5:3b**
- Versão mais leve do mesmo modelo
- 3B parâmetros para operações rápidas
- Mantém compatibilidade com Qwen2.5:7b
- Ideal para comandos simples e respostas rápidas

## 📋 Pré-requisitos
- Conta Oracle Cloud gratuita (VM.Standard3.Flex - 2 OCPU, 32GB RAM)
- Conhecimento básico de Linux
- Acesso SSH à VPS
- Chaves de API configuradas (opcional)
- Requisitos para Qwen2.5:7b:
  - ~4-5GB RAM para o modelo
  - ~2-3GB RAM para o Super Jarvis
  - ~4-5GB espaço em disco
  - 2 OCPU suficiente para boa performance (32GB RAM compensa)

### 📊 **VPS Oracle Cloud Configurada:**
- **IP Público**: 163.176.147.5
- **IP Privado**: 10.0.0.136
- **Subnet**: public subnet-jarvis
- **Shape**: VM.Standard3.Flex (2 OCPU, 32GB RAM)
- **Sistema**: Ubuntu 20.04 Minimal
- **Região**: sa-saopaulo-1 (São Paulo)
- **Disco**: 90GB
- **Usuário**: ubuntu

---

## 🗺️ **FASE 1: Configuração da VPS Oracle Cloud**

### 1.1 Criar Conta Oracle Cloud
1. Acesse: https://signup.cloud.oracle.com/
2. Clique em "Sign Up" ou "Try for Free"
3. Preencha as informações (email, senha, país)
4. Verifique o email e confirme a conta

### 1.2 Criar Instância Gratuita
1. Faça login no Oracle Cloud Console
2. Navegue para: **Compute** → **Instances**
3. Clique em **"Create Instance"**
4. Configure:
   - **Name**: `super-jarvis-vps`
   - **Compartment**: Selecione ou crie um compartment
   - **Shape**: `VM.Standard3.Flex` (x86 - 2 OCPU, 32GB RAM)
   - **Image**: **Oracle Linux** ou **Ubuntu**
   - **SSH Key**: Crie ou importe uma chave SSH
   - **VNIC**: Padrão (com IP público)
5. Clique em **"Create"**

### 1.3 Aguardar Criação
- Aguarde ~5-10 minutos para a instância ser criada
- Verifique o status mudar para "Running"
- Anote o **IP Público** da instância

### 1.4 Configurar Firewall e Security List
```bash
# No Oracle Cloud Console:
1. Vá para: Networking → Virtual Cloud Networks → jarvis
2. Clique em "Security Lists"
3. Adicione regras de entrada (Ingress Rules):

# Regra 1: SSH (já pode existir)
- Source: 0.0.0.0/0
- IP Protocol: TCP
- Destination Port: 22
- Description: SSH Access

# Regra 2: Super Jarvis API
- Source: 0.0.0.0/0
- IP Protocol: TCP
- Destination Port: 8000
- Description: Super Jarvis HTTP API

# Regra 3: Ollama
- Source: 0.0.0.0/0
- IP Protocol: TCP
- Destination Port: 11434
- Description: Ollama API

# Regra 4: 9router (se usar)
- Source: 0.0.0.0/0
- IP Protocol: TCP
- Destination Port: 20128
- Description: 9router API
```



---

## 🔧 **FASE 2: Acesso e Configuração Inicial da VPS**

### 2.1 Conectar via SSH
```bash
# No seu terminal local (Windows PowerShell ou CMD)
ssh -i /caminho/para/sua/chave.pem ubuntu@163.176.147.5

# Se estiver usando o Windows com PowerShell e sua chave em .ppk (PuTTY):
# Use PuTTY ou converta a chave para formato OpenSSH
```

### 2.2 Atualizar Sistema
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget python3 python3-pip python3-venv
```

### 2.3 Instalar uv (Gerenciador de Pacotes)
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.local/bin/activate  # Ou reinicie o terminal
```

### 2.4 Verificar Instalação
```bash
uv --version
python3 --version
```

---

## 🐍 **FASE 3: Instalação do Ollama**

### 3.1 Instalar Ollama
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### 3.2 Iniciar Ollama como Serviço
```bash
# Iniciar manualmente
ollama serve

# Ou criar serviço systemd (recomendado)
sudo systemctl enable ollama
sudo systemctl start ollama
```

### 3.3 Baixar Modelo Qwen2.5:7b (Escolha do Usuário)
```bash
# Qwen2.5:7b - Excelente para automações e comunicação
# Modelo mais capaz com 7B parâmetros, adequado para VPS ARM
# Otimizado para tarefas complexas, automação e diálogos naturais
ollama pull qwen2.5:7b

# Modelo backup mais leve para operações rápidas
ollama pull qwen2.5:3b
```

**Por que Qwen2.5:7b?**
- Excelente desempenho em automações e tarefas complexas
- Comunicação natural e fluente em múltiplos idiomas
- 7B parâmetros - equilíbrio ideal entre capacidade e performance
- Otimizado para arquitetura ARM
- Suporta contextos mais longos (até 32K tokens)
- Ideal para o Super Jarvis com capacidades avançadas

### 3.4 Verificar Ollama e Recursos
```bash
# Testar localmente
curl http://localhost:11434/api/tags
ollama list

# Verificar uso de memória do modelo
# Qwen2.5:7b requer aproximadamente 4-5GB RAM para rodar
# A VPS Oracle x86 tem 32GB RAM, então há espaço generoso
free -h
```

**Considerações de Recursos para Qwen2.5:7b:**
- **Memória**: ~4-5GB RAM para o modelo + ~2-3GB para o Super Jarvis
- **CPU**: 2 OCPU da VPS x86 são suficientes para boa performance (32GB RAM compensa)
- **Armazenamento**: ~4-5GB de espaço em disco para o modelo
- **Total**: VPS Oracle x86 (32GB RAM) é excelente para este modelo
- **Vantagem**: 32GB RAM oferece mais espaço para cache e multitarefa

### 3.5 Configurar Acesso Remoto ao Ollama
```bash
# Editar configuração do Ollama para aceitar conexões externas
# No Ubuntu/Debian, o Ollama geralmente já aceita conexões externas por padrão
# Se necessário, configurar no /etc/systemd/system/ollama.service
```

---

## 🤖 **FASE 4: Instalação do Super Jarvis**

### 4.1 Clonar Repositório
```bash
cd ~
git clone https://github.com/open-jarvis/OpenJarvis.git
cd OpenJarvis
```

### 4.2 Instalar Dependências
```bash
uv sync --extra desktop --group desktop-native
uv sync --extra inference-cloud
uv sync --extra inference-google
```

### 4.3 Configurar Variáveis de Ambiente
```bash
# Criar arquivo .env
cat > ~/.openjarvis/.env << EOF
# Configurações do Super Jarvis
GOOGLE_API_KEY=sua_chave_gemini_aqui
ANTHROPIC_API_KEY=sua_chave_claude_aqui
OPENAI_API_KEY=sua_chave_openai_aqui
DEEPSEEK_API_KEY=sua_chave_deepseek_aqui
OLLAMA_API_KEY=fa0e1317bed14b528f5d60a91f19bf96
EOF
```

### 4.4 Copiar Configuração do Super Jarvis
```bash
# Se você tem a configuração local, pode copiar
# Ou criar a configuração na VPS diretamente
mkdir -p ~/.openjarvis
```

### 4.5 Criar Configuração na VPS
```bash
cat > ~/.openjarvis/config.toml << 'EOF'
# Super Jarvis Configuration para Oracle VPS

[intelligence]
default_model = "qwen2.5:7b"          # Modelo Qwen2.5:7b otimizado para automações e comunicação
fallback_model = "qwen2.5:3b"    # Modelo de fallback mais leve
preferred_engine = "ollama"        # Engine local
provider = "local"                 # Usando modelos locais

[agent]
default_agent = "orchestrator"
max_turns = 15
tools = "shell_exec,file_read,file_write,web_search,browser_navigate,code_interpreter,calculator,think,weather,browser_click,browser_type,browser_screenshot,skill_manage,memory_manage,user_profile_manage,retrieval,http_request"
objective = "Super Jarvis com capacidades self-improving rodando em Oracle VPS"
context_from_memory = true
enable_skills = true
enable_delegation = true
enable_checkpoints = true

[engine]
default = "ollama"

[engine.ollama]
host = "http://localhost:11434"
api_key = "fa0e1317bed14b528f5d60a91f19bf96"

[skills]
enabled = true
skills_dir = "~/.openjarvis/skills/"
active = "*"
auto_discover = true
auto_sync = true
max_depth = 5
sandbox_dangerous = true

[[skills.sources]]
source = "hermes"
filter = { category = ["research", "coding", "productivity", "system", "web"] }
auto_update = true

[learning]
enabled = true

[learning.skills]
enabled = true
auto_optimize = true
optimizer = "dspy"
min_traces_per_skill = 10
optimization_interval_seconds = 43200
overlay_dir = "~/.openjarvis/learning/skills/"

[super_jarvis]
context_references = true
max_context_size = 1000000
enable_git_references = true
enable_url_references = true
checkpoints_enabled = true
checkpoint_dir = "~/.openjarvis/checkpoints/"
max_checkpoints = 20
auto_checkpoint_before_writes = true
delegation_enabled = true
max_subagents = 5
subagent_timeout = 300
parallel_execution = true
scheduling_enabled = true
scheduler_backend = "cron"
task_history_retention_days = 30
auto_skill_creation = true
skill_creation_threshold = 3
procedural_memory_enabled = true
cross_session_learning = true

[server]
host = "0.0.0.0"  # Aceitar conexões externas
port = 8000
agent = "orchestrator"

[telemetry]
enabled = true
db_path = "~/.openjarvis/telemetry.db"
gpu_metrics = false  # VPS ARM não tem GPU Nvidia
energy_vendor = ""

[traces]
enabled = true
db_path = "~/.openjarvis/traces.db"
EOF
```

### 4.6 Criar Arquivos de Personalidade
```bash
# SOUL.md
cat > ~/.openjarvis/SOUL.md << 'EOF'
# Personalidade do Super Jarvis (Oracle VPS)

## Tom e Estilo
- Seja amigável, profissional e conciso
- Responda em português brasileiro
- Use emojis moderadamente
- Seja proativo em sugerir soluções
- Você é um Especialista e TI
- Infreestrturura, Redes e programação.

## Capacidades Cloud
- Rodando em Oracle Cloud VPS (ARM Ampere A1)
- Modelo Ollama: qwen2:1.5b otimizado para ARM
- Acesso 24/7 via API e SSH
- Recursos: 4 OCPU, 24GB RAM

## Capacidades Especiais
- Self-Improving Learning Loop
- Context References: @files, @folders, @git-diffs, @URLs
- Checkpoint System com rollback
- Subagent Delegation paralelo
- Scheduled Automation
- Multi-Platform Gateway

## Limitações de Ambiente
- Ambiente ARM (considerar ao sugerir comandos)
- Sem GPU Nvidia (processamento CPU only)
- Latência de rede pode afetar performance
EOF

# MEMORY.md
cat > ~/.openjarvis/MEMORY.md << 'EOF'
# Memória do Super Jarvis (Oracle VPS)

## Ambiente Cloud
- Plataforma: Oracle Cloud VPS
- Shape: VM.Standard.E4.Flex (ARM Ampere A1)
- Recursos: 4 OCPU, 24GB RAM
- IP Público: <PREENCHER_IP>
- Sistema: Oracle Linux / Ubuntu

## Configuração Ollama
- Modelo Principal: qwen2.5:7b (otimizado para automações e comunicação)
- Modelo Backup: qwen2.5:3b (leve para operações rápidas)
- API: http://localhost:11434
- Contexto: 32K tokens (ideal para Super Jarvis avançado)
- Modelo Principal: qwen2.5:7b (otimizado para automações e comunicação)
- Modelo Backup: qwen2.5:3b (leve para operações rápidas)
- API: http://localhost:11434
- Contexto: 32K tokens (ideal para Super Jarvis avançado)

## Projeto Super Jarvis
- Versão: Super Jarvis 1.0 (OpenJarvis + Hermes fusion)
- Localização: ~/OpenJarvis
- Skills: Integrados do Hermes Agent
- Scripts: 4 scripts de suporte avançado

## Objetivos Cloud
- Serviço 24/7 de assistente virtual
- Acesso via API HTTP na porta 8000
- Integração com 9router em http://localhost:20128
- Escalabilidade horizontal futura
EOF

# USER.md
cat > ~/.openjarvis/USER.md << 'EOF'
# Perfil do Super Jarvis (Oracle VPS)

## Quem Sou Eu
- Super Jarvis Cloud Edition rodando em Oracle VPS
- Assistente AI avançado com capacidades self-improving
- Acessível via API HTTP e SSH
- Otimizado para ambiente ARM

## Capacidades Cloud
- **Acesso 24/7**: Serviço contínuo sem interrupções
- **Multi-usuário**: Pode servir múltiplos usuários simultaneamente
- **API HTTP**: Interface RESTful para integrações
- **Monitoramento**: Telemetria e traces habilitados
- **Backup**: Dados persistentes em SSD Oracle Cloud

## Serviços Integrados
- **Ollama Local**: Modelos rodando localmente na VPS
- **9router**: Servidor de roteamento em http://localhost:20128
- **Skills System**: ~150 skills do Hermes + personalizados
- **Context References**: Sistema avançado de gerenciamento de contexto

## Metas Cloud
- Prover assistência virtual de alta qualidade
- Manter baixa latência para respostas rápidas
- Escalar horizontalmente conforme necessidade
- Integrar com serviços externos via API
EOF
```

### 4.7 Copiar Scripts Personalizados
```bash
# Criar diretório de scripts
mkdir -p ~/.openjarvis/scripts

# Criar scripts (usar o conteúdo dos scripts locais)
# Você pode copiar os scripts do seu Windows para a VPS usando scp
# scp ~/.openjarvis/scripts/*.py ubuntu@<IP>:/home/ubuntu/.openjarvis/scripts/
```

---

## 🚀 **FASE 5: Configuração de Serviço e Deploy**

### 5.1 Criar Serviço Systemd para Ollama
```bash
sudo tee /etc/systemd/system/ollama.service > /dev/null << 'EOF'
[Unit]
Description=Ollama Service
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/local/bin/ollama serve
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable ollama
sudo systemctl start ollama
```

### 5.2 Criar Serviço Systemd para Super Jarvis
```bash
sudo tee /etc/systemd/system/super-jarvis.service > /dev/null << 'EOF'
[Unit]
Description=Super Jarvis Service
After=network.target ollama.service

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/OpenJarvis
Environment="PATH=/home/ubuntu/.local/bin:/usr/local/bin:/usr/bin:/bin"
ExecStart=/home/ubuntu/.local/bin/uv run python -m openjarvis.cli serve
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable super-jarvis
# Não iniciar ainda, vamos testar primeiro
```

### 5.3 Testar Super Jarvis Manualmente
```bash
cd ~/OpenJarvis
export GOOGLE_API_KEY="sua_chave_gemini"
uv run python -c "from openjarvis import Jarvis; jarvis = Jarvis(); print(jarvis.ask('Olá! Sou o Super Jarvis na Oracle VPS. Como posso ajudar?'))"
```

### 5.4 Testar Ollama Remotamente
```bash
# Da sua máquina local
curl http://<IP-PUBLICO>:11434/api/tags
```

### 5.5 Iniciar Serviço Super Jarvis
```bash
sudo systemctl start super-jarvis
sudo systemctl status super-jarvis
```

---

## 🌐 **FASE 6: Acesso Remoto e Integração**

### 6.1 Configurar Acesso HTTP à API
```bash
# Testar acesso local na VPS
curl http://localhost:8000/health

# Testar acesso externo (do seu Windows)
curl http://<IP-PUBLICO>:8000/health
```

### 6.2 Configurar Cliente Local (Windows)
```bash
# No seu Windows, configurar o OpenJarvis local para usar a VPS
# Editar C:\Users\Renato Lopes\.openjarvis\config.toml

[engine.ollama]
host = "http://<IP-PUBLICO>:11434"
```

### 6.3 Testar Integração Local-VPS
```bash
# No Windows
cd C:\Projetos\OpenJarvis-main\OpenJarvis-main
.venv\Scripts\python.exe -c "from openjarvis import Jarvis; jarvis = Jarvis(); print(jarvis.ask('Testando conexão com Ollama na VPS'))"
```

---

## 🔐 **FASE 7: Segurança e Monitoramento**

### 7.1 Configurar Firewall
```bash
# No Oracle Cloud Console, adicionar regras se necessário
# Portas já configuradas: 22, 8000, 11434, 20128
```

### 7.2 Configurar SSH Keys
```bash
# Usar autenticação por chave SSH (já configurado na criação)
# Desabilitar login por senha
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 7.3 Monitorar Serviços
```bash
# Verificar status dos serviços
sudo systemctl status ollama
sudo systemctl status super-jarvis

# Ver logs
sudo journalctl -u ollama -f
sudo journalctl -u super-jarvis -f
```

### 7.4 Monitorar Recursos
```bash
# Uso de CPU e memória
htop
# Ou
top

# Uso de disco
df -h

# Memória disponível
free -h
```

---

## 🧪 **FASE 8: Testes e Validação**

### 8.1 Testar Básico do Super Jarvis
```bash
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Olá! Como você está funcionando?"}],
    "stream": false
  }'
```

### 8.2 Testar Capacidades de Automação do Qwen2.5:7b
```bash
# Testar comando de automação complexo
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Crie um script Python que monitore o uso de CPU e envie alerta se > 80%"}],
    "stream": false
  }'

# Testar agendamento de tarefa
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Agende um backup automático para todo dia às 2h da manhã"}],
    "stream": false
  }'
```

### 8.3 Testar Capacidades de Comunicação do Qwen2.5:7b
```bash
# Testar conversação natural em português
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Explique-me como funciona o sistema de checkpoints do Super Jarvis de forma simples e didática"}],
    "stream": false
  }'

# Testar raciocínio complexo
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Analise os logs do sistema e identifique possíveis problemas de performance no Super Jarvis"}],
    "stream": false
  }'
```

### 8.2 Testar Context References
```bash
python ~/.openjarvis/scripts/context_resolver.py @README.md
```

### 8.3 Testar Checkpoint System
```bash
python ~/.openjarvis/scripts/checkpoint_manager.py create "test.txt" "teste inicial"
python ~/.openjarvis/scripts/checkpoint_manager.py list
```

### 8.4 Testar Skill Discovery
```bash
python ~/.openjarvis/scripts/skill_optimizer.py discover --dry-run
```

### 8.5 Testar Subagent Delegation
```bash
python ~/.openjarvis/scripts/subagent_orchestrator.py delegate "Testar sistema" "shell_exec,web_search"
```

### 8.6 Testar Performance com Qwen2.5:7b
```bash
# Monitorar uso de recursos durante testes intensivos
htop

# Testar resposta com contexto longo (32K tokens do Qwen2.5:7b)
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Analise todo o projeto OpenJarvis e me dê um resumo detalhado da arquitetura, incluindo todos os componentes principais, suas interações e como o Qwen2.5:7b se integra neste sistema para fornecer capacidades avançadas de automação e comunicação"}],
    "stream": false
  }'

# Verificar latência e throughput
time curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Responda rápido: Qual é a capital do Brasil?"}],
    "stream": false
  }'
```

---

## 📊 **FASE 9: Otimização e Performance**

### 9.1 Otimizar Ollama para ARM com Qwen2.5:7b
```bash
# Ollama já é otimizado para ARM, mas podemos ajustar parâmetros específicos para Qwen2.5:7b
# Aumentar cache e buffers para melhor performance
# O Qwen2.5:7b suporta contexto de 32K tokens - podemos aproveitar isso

# Verificar parâmetros atuais do Ollama
ollama show qwen2.5:7b

# Ajustar parâmetros se necessário (via variáveis de ambiente)
export OLLAMA_NUM_THREAD=4  # Usar todos os 4 OCPU
export OLLAMA_MAX_LOADED_MODELS=2  # Carregar Qwen2.5:7b e qwen2.5:3b simultaneamente
```

### 9.2 Configurar Memória do Super Jarvis para Qwen2.5:7b
```bash
# A VPS tem 24GB RAM - distribuição ideal:
# - Qwen2.5:7b: ~4-5GB RAM
# - Qwen2.5:3b (backup): ~2-3GB RAM
# - Super Jarvis: ~2-3GB RAM
# - Sistema e cache: ~5-10GB RAM
# Total: ~13-21GB RAM (há espaço suficiente)

# Adicionar ao config.toml se necessário para otimizar uso de memória
# Considerar aumentar cache para aproveitar a RAM disponível
```

### 9.3 Monitorar Performance
```bash
# Monitorar uso de recursos durante uso intenso
htop
```

---

## 🔄 **FASE 10: Manutenção e Updates**

### 10.1 Atualizar Super Jarvis
```bash
cd ~/OpenJarvis
git pull
uv sync
sudo systemctl restart super-jarvis
```

### 10.2 Atualizar Ollama
```bash
# Atualizar Ollama
curl -fsSL https://ollama.com/install.sh | sh
sudo systemctl restart ollama

# Atualizar modelo
ollama pull qwen2.5:7b
ollama pull qwen2.5:3b
```

### 10.3 Backup de Configurações
```bash
# Backup das configurações
tar -czf ~/super-jarvis-backup-$(date +%Y%m%d).tar.gz ~/.openjarvis ~/OpenJarvis
```

---

## 🎯 **FASE 11: Integração 9router (Opcional)**

### 11.1 Configurar 9router na VPS
```bash
# Se você tiver o 9router, pode instalar na VPS
# Assumindo que você tenha o binário ou docker-compose
# Configure para rodar na porta 20128
```

### 11.2 Integrar via MCP
```bash
# Adicionar configuração MCP no config.toml
[tools.mcp]
enabled = true

[[tools.mcp.servers]]
name = "9router"
url = "http://localhost:20128"
```

---

## 📱 **FASE 12: Acesso Multi-Plataforma**

### 12.1 Configurar Telegram (Opcional)
```bash
# Seguir documentação do Hermes Agent para configurar Telegram
# Usar o gateway do Super Jarvis
```

### 12.2 Configurar Discord (Opcional)
```bash
# Similar ao Telegram, seguir documentação
```

---

## ✅ **CHECKLIST FINAL**

### Pré-Deploy
- [ ] Conta Oracle Cloud criada
- [ ] Instância ARM Ampere A1 criada
- [ ] Chave SSH configurada
- [ ] Firewall configurado (portas 22, 8000, 11434, 20128)

### Instalação
- [ ] Sistema atualizado
- [ ] uv instalado
- [ ] Python 3 instalado
- [ ] Git instalado
- [ ] Ollama instalado e rodando
- [ ] Modelo qwen2.5:7b baixado
- [ ] Modelo qwen2.5:3b baixado (backup)
- [ ] OpenJarvis clonado
- [ ] Dependências instaladas
- [ ] Configuração do Super Jarvis criada
- [ ] Scripts de suporte copiados

### Configuração
- [ ] Serviço Ollama systemd criado
- [ ] Serviço Super Jarvis systemd criado
- [ ] Variáveis de ambiente configuradas
- [ ] Firewall Oracle Cloud configurado
- [ ] SSH seguro configurado

### Testes
- [ ] Ollama acessível remotamente
- [ ] Super Jarvis testado manualmente
- [ ] Super Jarvis testado via systemd
- [ ] API HTTP acessível externamente
- [ ] Integração local-VPS testada
- [ ] Scripts de suporte testados

### Monitoramento
- [ ] Monitoramento de recursos configurado
- [ ] Logs configurados
- [ ] Backup automatizado configurado

---

## 🚀 **PRÓXIMOS PASSOS APÓS DEPLOY**

### 1. Testar Funcionalidades Específicas
- Testar context references
- Testar checkpoint system
- Testar subagent delegation
- Testar skill discovery

### 2. Configurar Dominínio (Opcional)
- Configurar domínio para acesso via HTTPS
- Configurar SSL/TLS
- Configurar DNS

### 3. Escalabilidade
- Monitorar performance sob carga
- Configurar load balancing se necessário
- Considerar múltiplas instâncias

### 4. Integrações Adicionais
- Configurar banco de dados externo
- Integrar com serviços de nuvem
- Configurar webhooks

---

## 📞 **SOLUÇÃO DE PROBLEMAS**

### Ollama não inicia
```bash
# Verificar logs
sudo journalctl -u ollama -f

# Verificar se porta está em uso
sudo netstat -tlnp | grep 11434
```

### Super Jarvis não inicia
```bash
# Verificar logs
sudo journalctl -u super-jarvis -f

# Verificar se Ollama está rodando
sudo systemctl status ollama
```

### Acesso externo bloqueado
```bash
# Verificar firewall Oracle Cloud
# Verificar security lists na VCN
# Verificar se serviços estão escutando em 0.0.0.0
```

### Performance lenta com Qwen2.5:7b
```bash
# Verificar uso de recursos
htop

# Verificar se há memória suficiente
free -h

# Considerar usar o modelo de backup qwen2.5:3b para operações rápidas
ollama pull qwen2.5:3b
# E mudar config.toml para usar qwen2.5:3b como fallback em operações simples

# Ajustar threads do Ollama para usar todos os OCPU
export OLLAMA_NUM_THREAD=4  # Usar todos os 4 OCPU disponíveis
```

---

## � **APROVEITANDO QWEN2.5:7B NO SUPER JARVIS**

### Capacidades Específicas Habilitadas

#### **1. Automações Complexas**
```bash
# Exemplo: Criar automação complexa
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Crie um sistema completo de monitoramento que: 1) Monitora CPU, RAM e disco a cada 5 minutos, 2) Gera logs estruturados, 3) Envia alertas via email se CPU > 80%, 4) Cria gráficos de tendência, 5) Mantém histórico de 30 dias"}],
    "stream": false
  }'
```

#### **2. Análise Avançada de Contexto**
```bash
# Exemplo: Análise de projeto completo com contexto longo
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Analise todo o código do Super Jarvis, identifique possíveis otimizações de performance, sugira melhorias de segurança e proponha novos recursos que adicionariam valor ao sistema"}],
    "stream": false
  }'
```

#### **3. Comunicação Multilíngue Natural**
```bash
# Exemplo: Diálogo natural em português
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Poderia me explicar de forma simples e didática como funciona o sistema de checkpoint do Super Jarvis, incluindo exemplos práticos de quando usar rollback e como configurar backups automáticos?"}],
    "stream": false
  }'
```

#### **4. Raciocínio Lógico Avançado**
```bash
# Exemplo: Resolução de problemas complexos
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Diagnostique por que o Super Jarvis está respondendo lentamente, considere: uso de CPU atual, latência de rede, carga do modelo Ollama, possíveis gargalos de I/O, e proponha uma solução sistemática para resolver cada problema identificado"}],
    "stream": false
  }'
```

#### **5. Geração de Código Complexo**
```bash
# Exemplo: Sistema completo de backup
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "messages": [{"role": "user", "content": "Crie um script Python completo que implemente um sistema de backup inteligente: backup incremental, compressão, criptografia, upload para nuvem, verificação de integridade, e sistema de notificação de sucesso/falha"}],
    "stream": false
  }'
```

### Estratégia de Uso dos Modelos

**Para Qwen2.5:7b (Principal):**
- Automações complexas e multi-etapas
- Análise profunda de código e arquitetura
- Diálogos naturais e detalhados
- Raciocínio lógico avançado
- Geração de código complexo
- Contexto longo (até 32K tokens)

**Para Qwen2.5:3b (Backup):**
- Comandos simples e diretos
- Respostas rápidas e triviais
- Operações de baixa latência
- Queries simples de informação
- Diálogos curtos

### Configuração de Roteamento Inteligente

```toml
# Adicionar ao config.toml para roteamento automático
[model_routing]
strategy = "complexity_based"
complexity_threshold = 0.7  # 0-1, acima disso usa qwen2.5:7b
simple_model = "qwen2.5:3b"
complex_model = "qwen2.5:7b"

[model_routing.indicators]
token_count_threshold = 1000
tool_count_threshold = 3
multi_step_threshold = true
```

---

## �🎓 **RECURSOS**

### Documentação Oficial
- OpenJarvis: https://open-jarvis.github.io/OpenJarvis/
- Ollama: https://ollama.ai/
- Oracle Cloud: https://docs.oracle.com/en-us/iaas/
- Hermes Agent: https://hermes-agent.nousresearch.com/

### Comunidade
- Discord OpenJarvis: https://discord.gg/CMVBmDQ5Fj
- GitHub Issues: https://github.com/open-jarvis/OpenJarvis/issues

---

## 🎉 **CONCLUSÃO**

Após seguir este roadmap, você terá:

✅ **Super Jarvis rodando em Oracle Cloud VPS**
✅ **Modelo Qwen2.5:7b otimizado para automações e comunicação**
✅ **Modelo backup Qwen2.5:3b para operações rápidas**
✅ **Capacidades self-imving avançadas**
✅ **Acesso 24/7 via API HTTP**
✅ **Integração completa com 9router**
✅ **Sistema de backup e monitoramento**
✅ **Performance otimizada para ARM Ampere**
✅ **Contexto de 32K tokens para capacidades avançadas**

O Super Jarvis estará pronto para servir como assistente virtual avançado com capacidade de aprendizado contínuo, automação inteligente e comunicação natural de alta qualidade, tudo rodando eficientemente na VPS Oracle Cloud ARM!