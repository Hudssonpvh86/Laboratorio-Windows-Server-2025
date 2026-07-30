# 📑 IMPLEMENTAÇÃO DE INFRAESTRUTURA DE REDE LOCAL E DIRETÓRIO CENTRALIZADO (AD DS)

## 👤 Sobre o Autor
* **Perfil:** Tecnólogo em Análise e Desenvolvimento de Sistemas
* **Certificações:** ITIL® 4 Foundation (ITSM) | Suporte Técnico (Google/Coursera)
* **Especialidade:** Montagem, Manutenção de Desktops e Infraestrutura de TI

---

## 🎯 1. Escopo do Projeto e Cenário Corporativo
Este projeto documenta o provisionamento de um ambiente de laboratório virtualizado (Oracle VM VirtualBox) rodando **Windows Server 2025**. O objetivo foi centralizar a governança de identidades, aplicar políticas rígidas de segurança (Hardening) e gerenciar ativos e dados em uma rede empresarial simulada de médio porte.

---

## 🛠️ 2. Diagnóstico de Incidentes, Lentidão e Erros de Perfil
Durante o ciclo de testes pós-congelamento da máquina virtual, o console do *Server Manager* apresentou corrupção nos arquivos de cache XML (`user.config` e `ServerList.xml`), travando os serviços de inventário (WMI) com o status preso em *"In progress"*.

* **Resolução Técnica:** Forçado o encerramento do processo via Gerenciador de Tarefas. O cache corrompido foi expurgado manualmente através do diretório oculto da aplicação:
  ```bash
  %localappdata%\Microsoft_Corporation
  ```
* **Resultado:** O Windows Server recriou a estrutura de profile de forma limpa, normalizando o status do painel para: *"Online - Performance counters not started"*.

---

## 📡 3. Camada de Rede e Testes de Conectividade via CLI
Para garantir a estabilidade do ecossistema, desativamos o endereçamento dinâmico (DHCP) e aplicamos uma topologia de IP estático (fixo).

* **Configurações Locais:** IP `10.0.2.15` | Máscara `255.255.255.0` | Gateway `10.0.2.2`
* **DNS Preferencial:** `127.0.0.1` *(Loopback essencial para a resolução de nomes do Active Directory)*.

### Comandos de Auditoria Executados no Prompt (CMD):
* `ipconfig /all`: Validação do status estático da placa e integridade das propriedades de rede.
* `ping 8.8.8.8`: Teste de ICMP apontando estabilidade total para a saída de internet (0% de perda de pacotes).
* `tracert google.com`: Rastreamento de rota isolando saltos de rede até o servidor de destino em Guarulhos/SP (`gru...`).
* `nslookup google.com`: Validação da capacidade de resposta do servidor de nomes.

---

## ⚙️ 4. Gerenciamento de Serviços e Atualizações (`services.msc`)
Tratamento de chamado técnico simulado para mitigar falhas crônicas de downloads em segundo plano do Windows Update:
* **Identificação:** O serviço **BITS** (*Background Intelligent Transfer Service*) encontrava-se em estado *Stopped* com inicialização *Manual*.
* **Remediação:** Alterado o tipo de inicialização para **`Automatic`** e disparada a execução do serviço em tempo real (*Running*).

---

## 👑 5. Serviços de Diretório (Active Directory) e Governança
Instalação do escopo de funções do **AD DS** e promoção da máquina a Controlador de Domínio raiz da floresta **`suporte.local`** (NetBIOS: `SUPORTE`).

* **Unidade Organizacional (OU):** Criação da pasta estrutural `TI` para segregação de privilégios.
* **Provisionamento de Usuários:** Contas criadas para os colaboradores `maria.silva` e `carlos.mendonca` (aplicando a boa prática de omitir caracteres especiais ou cedilhas em nomes de logon).
* **Segurança de Acesso:** Criação do Grupo Global `GG_TI` para gerenciamento centralizado de acessos, aplicação de regras de complexidade de senhas e imposição do parâmetro *User must change password at next logon*.

---

## 🔒 6. Políticas de Grupo Aplicadas (GPO)
Para garantir o *Hardening* das estações de trabalho, criamos e vinculamos diretivas rigorosas na OU `TI`:
1. **`Bloqueio_Painel_Controle`:** Restrição total de acesso ao painel e configurações globais do sistema operacional.
2. **`Bloqueio_Cmd`:** Ativação da diretiva *Prevent access to the command prompt* para mitigar a execução de scripts e comandos por usuários comuns do setor.

> 🚀 **Comando de Aplicação Imediata:** Para espalhar as regras sem aguardar o ciclo padrão de 90 minutos do sistema, forçamos a atualização da diretiva via terminal:
> ```bash
> gpupdate /force
> ```

---

## 📂 7. Servidor de Arquivos e Permissões NTFS
Criação de repositório centralizado no volume local (`C:\Arquivos_Empresa`) mitigando vazamento de escopo de dados:
* **Compartilhamento SMB:** Permissão de controle total (`Full Control`) para o grupo genérico *Everyone* (Todos).
* **Segurança NTFS (Filtro Real):** Inclusão explícita do grupo **`GG_TI`** com o nível de permissão **`Modify`** (Modificar). Qualquer conta fora deste grupo que tentar mapear ou acessar o diretório receberá a mensagem de acesso negado.

---

### 📈 Conclusão e Resultados
A arquitetura deste laboratório consolida conhecimentos práticos sólidos de administração de servidores Windows Server, segurança cibernética corporativa e aplicação de frameworks de ITSM (ITIL 4) no gerenciamento e tratamento de incidentes.
