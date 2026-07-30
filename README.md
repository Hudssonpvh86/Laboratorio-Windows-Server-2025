DOCUMENTAÇÃO TÉCNICA: IMPLEMENTAÇÃO DE INFRAESTRUTURA WINDOWS SERVER E GESTÃO DE IDENTIDADES (IAM)

 1. Diagnóstico de Lentidão, Travamento e Erros do Painel
Após religar a máquina virtual a partir do estado salvo, o console do Server Manager congelou e apresentou erros críticos de arquivos XML corrompidos (user.config e ServerList.xml) devido a encerramentos forçados no VirtualBox, impedindo o gerenciamento do servidor.

* Análise de Causa Raiz: O congelamento da VM travou os serviços de inventário do sistema em segundo plano (WMI e TrustedInstaller), deixando o status do painel preso em "In progress".
* Resolução Técnica: Forçamos o encerramento do processo travado pelo Gerenciador de Tarefas (Ctrl + Shift + Esc). Em seguida, limpamos o cache corrompido de perfil do sistema navegando na pasta oculta através do comando Executar (Windows + R):

%localappdata%\Microsoft_Corporation

Deletamos a pasta corrompida do Server Manager. O Windows recriou o cache do zero e o painel voltou a carregar de forma 100% saudável, exibindo o status correto: "Online - Performance counters not started".

------------------------------
## 📡 2. Testes Avançados de Rede no CMD (Linha de Comando)
Com o sistema destravado, abrimos o Prompt de Comando (CMD) para validar a topologia de rede local e testar a conectividade externa:

* ipconfig /all: Comando utilizado para detalhar a placa de rede. Validamos que o DHCP estava desativado (DHCP Enabled: No), confirmando o uso de IP Fixo (Estático) 10.0.2.15, Máscara 255.255.255.0, Gateway 10.0.2.2 e DNS local apontando para 127.0.0.1 (localhost). [1] 
* ping 8.8.8.8: Comando de teste de conectividade direta enviando pacotes ao DNS do Google. O resultado mostrou 0% de perda de pacotes e tempo médio de resposta de 83ms, provando que a VM tinha acesso estável à internet. [2] 
* tracert google.com: Comando de rastreamento de rota (Trace Route). Analisamos o trajeto dos pacotes e identificamos que o tráfego foi resolvido em apenas 1 salto direto através do servidor do Google localizado fisicamente em Guarulhos/SP (gru...), demonstrando a topologia de rede virtualizada.
* nslookup google.com: Comando de consulta de DNS utilizado para testar a resolução de nomes na rede antes da ativação completa do servidor.

------------------------------
## ⚙️ 3. Gerenciamento de Serviços e Atualizações (services.msc)
Para resolver um cenário clássico onde o servidor apresenta falhas em downloads de atualizações automáticas ou transferências de arquivos em segundo plano, realizamos a auditoria e correção de serviços do sistema:

* Ferramenta: Console de Serviços (Windows + R > services.msc).
* O Problema: Identificamos que o serviço BITS (Background Intelligent Transfer Service) — responsável por gerenciar downloads em segundo plano do Windows Update — estava com o status zerado (Stopped) e configurado para inicialização Manual. [3] 
* A Correção: Abrimos as propriedades do BITS, alteramos o tipo de inicialização (Startup type) para Automatic (Garantindo que ele suba sozinho com o Windows) e clicamos no botão Start para forçar a execução imediata dele em tempo real (Running).

------------------------------
## 👑 4. Instalação e Promoção do Active Directory (AD DS)

* Instalação: Ativamos a função Active Directory Domain Services através do assistente do Server Manager, instalando os binários necessários no sistema.
* Promoção: Iniciamos o assistente definitivo pelas notificações do painel, selecionamos a opção de criar uma nova floresta corporativa do zero com o domínio suporte.local (NetBIOS: SUPORTE).
* Segurança: Definimos a senha do modo de restauração (DSRM) e avançamos ignorando o aviso padrão de delegação DNS. Após a validação dos pré-requisitos, o sistema realizou o reboot automático e subiu a tela de login corporativo exibindo o formato de rede: SUPORTE\Administrator.

------------------------------
## 🗂️ 5. Governança de Usuários e Grupos (ADUC)
Abrimos a ferramenta Active Directory Users and Computers para estruturar a hierarquia da empresa de forma organizada:

* Unidade Organizacional (OU): Criamos a pasta corporativa chamada TI para isolar as contas dos funcionários das pastas nativas.
* Usuários Provisionados: Criamos as contas de maria.silva e carlos.mendonca seguindo as boas práticas internacionais de nunca utilizar caracteres especiais, acentos ou cedilha no nome de logon (mudando de mendonça para mendonca).
* Grupo de Segurança: Criamos o Grupo Global GG_TI e associamos a Maria e o Carlos a ele, centralizando a gestão de acessos de rede.
* Políticas de Senha: Aplicamos regras de senhas complexas (mínimo de 7 dígitos misturando maiúsculas, minúsculas e números) para corrigir o erro de violação de política do AD. Simulamos um atendimento real de help desk rodando um reset de senha e desbloqueio manual de conta.

------------------------------
## 🔒 6. Políticas de Segurança Avançadas (GPO)
Abrimos o console de Group Policy Management para travar privilégios de usuários comuns:

* GPO 1 (Bloqueio_Painel_Controle): Ativamos a diretiva de proibir acesso ao Painel de Controle e configurações do PC, vinculando-a na OU TI.
* GPO 2 (Bloqueio_Cmd): Editamos a política dentro das configurações do usuário (User Configuration > Administrative Templates > System), ativando a diretiva Prevent access to the command prompt.
* gpupdate /force: Comando de Ouro do Analista. Executamos via terminal para forçar o servidor a ler e aplicar todas as novas regras de segurança imediatamente na rede, sem precisar aguardar o ciclo padrão de 90 minutos do sistema.

------------------------------
## 📂 7. Servidor de Arquivos (Compartilhamento e NTFS)
Finalizamos o dia montando um repositório centralizado e altamente seguro para os dados da empresa:

* Criação: Criamos a pasta física Arquivos_Empresa diretamente no diretório raiz C:\.
* Compartilhamento de Rede: Compartilhamos a pasta na rede sob o protocolo SMB, concedendo permissão de controle total (Full Control) para o grupo genérico Everyone (Todos).
* Segurança NTFS (Filtro Real): Entramos na aba Security da pasta, adicionamos o grupo GG_TI e concedemos o nível de permissão Modify (Modificar). Com isso, apenas a Maria e o Carlos (que pertencem a esse grupo) conseguem abrir, ler, salvar ou alterar arquivos ali dentro; qualquer outro usuário que tentar acessar a pasta pela rede terá o acesso negado.

------------------------------
## 🕵️‍♂️ 8. Auditoria de Desligamentos Inesperados (Event Viewer)

* Ferramenta: Visualizador de Eventos corporativo (Windows + R > eventvwr.msc).
* Investigação Prática: Acessamos os logs de System e isolamos o Event ID 1076 (Source: User32). Através da aba Details (Friendly View), coletamos as evidências do encerramento inesperado do servidor, auditando o horário exato da queda, o nome do processo responsável (taskhostw.exe), o tempo de duração do encerramento (ShutdownDuration de 5 segundos), o usuário que assumiu o erro (WIN-D35.../Administrator) e a justificativa preenchida no sistema ("teste").
