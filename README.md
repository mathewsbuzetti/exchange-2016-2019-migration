# 🔄 Migração do Exchange Server 2016 para Exchange Server 2019
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Mathews_Buzetti-blue)](https://www.linkedin.com/in/mathewsbuzetti)
![Exchange](https://img.shields.io/badge/Exchange-2019-orange?style=flat-square)
[![Status](https://img.shields.io/badge/Status-Production-green)]()
![Documentation](https://img.shields.io/badge/Documentation-Technical-blue?style=flat-square)

## 📋 Metadados
Metadado | Descrição
---------|------------
| **Título** | Migração do Exchange Server 2016 para Exchange Server 2019
| **Assunto** | Microsoft Exchange Server
| **Versão** | 1.0.0 |
| **Data** | 21/03/2025
| **Autor** | Mathews Buzetti
| **Tags** | `exchange-server`, `email-migration`, `windows-server`, `active-directory`, `infrastructure-migration`

## 📋 Índice

1. [Metadados](#-metadados)
2. [Objetivo](#-objetivo)
3. [Avisos Importantes Antes de Iniciar](#-avisos-importantes-antes-de-iniciar)
4. [Pré-requisitos](#-pré-requisitos)
5. [Instalação de Recursos no Windows Server](#-instalação-de-recursos-no-windows-server)
6. [Download do Exchange](#-download-do-exchange)
7. [Preparação do Active Directory](#-preparação-do-active-directory)
8. [Instalação do Exchange](#-instalação-do-exchange)
9. [Verificação de Avisos MAPI](#-verificação-de-avisos-mapi)
10. [Configuração de URLs](#-configuração-de-urls)
11. [Certificados SSL](#-certificados-ssl)
12. [Configuração de Armazenamento](#-configuração-de-armazenamento)
13. [Configuração DNS](#-configuração-dns)
14. [Migração de Caixas de Correio Exchange](#-migração-de-caixas-de-correio-exchange)
15. [Configuração de Conectores e Verificação de Filas](#-configuração-de-conectores-e-verificação-de-filas)
16. [Testes Finais](#-testes-finais)
17. [Versionamento](#-versionamento)

## 🎯 Objetivo
Este guia fornece um roteiro detalhado para migração do Microsoft Exchange Server 2016 para o Exchange Server 2019. O processo de migração é crítico e requer planejamento cuidadoso para garantir:
- Mínima interrupção dos serviços de email
- Preservação integral dos dados dos usuários
- Transição suave entre versões do Exchange

## 🚨 Avisos Importantes Antes de Iniciar
> [!WARNING]
> - Sempre faça backup completo de todo o ambiente antes de iniciar
> - Realize a migração em horário de baixo impacto
> - Tenha um plano de rollback detalhado
> - Comunique antecipadamente todos os usuários sobre a migração

## 💻 Pré-requisitos

### 📥 1.1. Instalação do .NET Framework 4.8
**Descrição**: O .NET Framework 4.8 é um requisito fundamental para o Exchange Server 2019. Ele fornece a base de runtime necessária para execução do Exchange.

**Procedimento**:
1. Download: [.NET Framework 4.8](https://dotnet.microsoft.com/pt-br/download/dotnet-framework/thank-you/net48-web-installer)
2. Execute o instalador com privilégios administrativos
3. Siga as instruções na tela
4. Reinicie o servidor após a instalação

**Verificação**:
- Abra o PowerShell e execute: `Get-WindowsFeature Net-Framework-45-Features`
- Confirme se está instalado e habilitado

### 📥 1.2. Visual C++ 2012 Redistributable
**Descrição**: Este componente fornece bibliotecas runtime necessárias para aplicações compiladas com Visual Studio 2012.

**Procedimento**:
1. Download: [VC++ 2012](https://www.microsoft.com/en-us/download/details.aspx?id=30679)
2. Execute o instalador
3. Aceite os termos e continue a instalação
4. Aguarde a conclusão

### 📥 1.3. Visual C++ 2013 Redistributable
**Descrição**: Similar ao anterior, este componente fornece bibliotecas runtime para aplicações mais recentes.

**Procedimento**:
1. Download: [VC++ 2013](https://aka.ms/highdpimfc2013x64ptb)
2. Execute o instalador
3. Siga o processo de instalação
4. Aguarde a conclusão

### 📥 1.4. Unified Communications Managed API 4.0
**Descrição**: O UCMA 4.0 é necessário para funcionalidades de comunicação unificada do Exchange.

**Procedimento**:
1. Download: [UCMA 4.0](https://www.microsoft.com/en-us/download/details.aspx?id=34992)
2. Execute o instalador como administrador
3. Siga as instruções de instalação
4. Verifique a conclusão no Painel de Controle

### 📥 1.5. IIS URL Rewrite Module
**Descrição**: Este módulo é crucial para o funcionamento do Exchange Server 2019, permitindo o redirecionamento correto de URLs.

**Procedimento**:
1. Download: [IIS URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
2. Execute a instalação como administrador
3. Aguarde a conclusão

> [!WARNING]
> Reinicie o servidor após a instalação

## 🛠️ Instalação de Recursos no Windows Server

### 🔧 2.1. Instalar RSAT-ADDS
**Descrição**: As Remote Server Administration Tools (RSAT) são necessárias para gerenciar o Active Directory.

**Procedimento**:
```powershell
Install-WindowsFeature RSAT-ADDS
```

**Verificação**:
```powershell
Get-WindowsFeature RSAT-ADDS | Select Name, Installed
```

### 🔧 2.2. Instalar Recursos Necessários
**Descrição**: Estes são os componentes fundamentais do Windows Server necessários para o Exchange.

**Procedimento**:
```powershell
Install-WindowsFeature NET-Framework-45-Features, RPC-over-HTTP-proxy, RSAT-Clustering, RSAT-Clustering-CmdInterface, RSAT-Clustering-Mgmt, RSAT-Clustering-PowerShell, Web-Mgmt-Console, WAS-Process-Model, Web-Asp-Net45, Web-Basic-Auth, Web-Client-Auth, Web-Digest-Auth, Web-Dir-Browsing, Web-Dyn-Compression, Web-Http-Errors, Web-Http-Logging, Web-Http-Redirect, Web-Http-Tracing, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Lgcy-Mgmt-Console, Web-Metabase, Web-Mgmt-Console, Web-Mgmt-Service, Web-Net-Ext45, Web-Request-Monitor, Web-Server, Web-Stat-Compression, Web-Static-Content, Web-Windows-Auth, Web-WMI, Windows-Identity-Foundation
```

**Verificação**:
```powershell
Get-WindowsFeature | Where-Object {$_.Installed -eq $true} | Format-Table Name,InstallState
```
## 📦 Download do Exchange
**Descrição**: O Exchange Server 2019 CU14 é a versão que será utilizada nesta instalação.

**Procedimento**:
1. Download: [Exchange 2019 CU14](https://www.microsoft.com/pt-br/download/details.aspx?id=105878)
2. Extraia o arquivo ISO baixado
3. Mantenha os arquivos em um local de fácil acesso

> [!WARNING]\
> Certifique-se de baixar a versão mais recente disponível no momento da instalação.

## 🔄 Preparação do Active Directory

### 🛠️ 4.1. Preparar Schema
**Descrição**: Esta etapa estende o schema do Active Directory para incluir os atributos necessários para o Exchange 2019.

**Procedimento**:
```cmd
Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareSchema
```

**Verificação**:
- Verifique os logs em: `%ExchangeInstallPath%\Logging\ExchangeSetup`
- Confirme que não há erros no Event Viewer

### 🛠️ 4.2. Preparar Active Directory
**Descrição**: Esta etapa prepara o Active Directory para o Exchange 2019, criando os containers e grupos necessários.

**Para primeira instalação**:
```cmd
Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAD /OrganizationName:"SuaOrganizacao"
```

**Para ambiente existente**:
```cmd
Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAD
```

> [!WARNING]
> - O nome da organização não pode ser alterado depois de definido
> - Use um nome sem espaços

### 🛠️ 4.3. Preparar Todos os Domínios
**Descrição**: Prepara todos os domínios da floresta para o Exchange 2019.

**Procedimento**:
```cmd
Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAllDomains
```

> [!NOTE]
> **Verificação**:
> - Aguarde a replicação do AD (pode levar várias horas)
> - Verifique os logs de instalação
> - Confirme as alterações no AD Sites and Services

## 📝 Instalação do Exchange

### 🔧 5.1. Instalação Inicial
**Descrição**: Instalação do Exchange Server 2019 no novo servidor.

**Procedimento**:
1. Execute Setup.exe como administrador
2. Selecione "Next" na tela inicial
3. Aceite os termos de licença
4. Escolha se deseja participar do programa de experiência do cliente
5. Selecione a instalação recomendada
6. Especifique o local de instalação
7. Aguarde a conclusão da instalação

> [!NOTE]
> **Verificações Pós-Instalação**:
> - Verifique o Event Viewer
> - Teste o acesso ao ECP (Exchange Control Panel)
> - Verifique os serviços do Exchange

## 🔧  Verificação de Avisos MAPI
Se você encontrar avisos relacionados ao MAPI over HTTP durante a instalação, siga estas etapas no servidor Exchange antigo 2016:

### 1.1. Preparação
Abra o Exchange Management Shell com privilégios de administrador no servidor Exchange antigo (por exemplo, Exchange 2016).

### 1.2. Verificação do Ambiente
Execute os seguintes comandos na ordem especificada:

**Mostrar informações de configuração do ambiente**:
```powershell
Get-OutlookAnywhere | Format-List
```

**Verificar se o MAPI está habilitado**:
```powershell
Get-OrganizationConfig | Format-List *mapihttpenabled
```

### 1.3. Configuração do Diretório Virtual
Por padrão, o Exchange cria um diretório virtual para MAPI sobre HTTP. Use o cmdlet Set-MapiVirtualDirectory para configurá-lo:

```powershell
Set-MapiVirtualDirectory -Identity "<NomeDoSeuServidor>\mapi (Default Web Site)" -InternalUrl https://<SeuDominio.com>/mapi -IISAuthenticationMethods Negotiate
```

**Observações sobre o comando**:
- Substitua `<NomeDoSeuServidor>` pelo nome do seu servidor Exchange
- Substitua `<SeuDominio.com>` pelo nome de domínio completo (FQDN) do seu servidor

### 1.4. Habilitação do MAPI
Execute o comando para habilitar conexões MAPI sobre HTTP para toda a organização:

```powershell
Set-OrganizationConfig -MapiHttpEnabled $true
```

### 1.5. Verificação Final
Confirme se o MAPI foi ativado corretamente:

```powershell
Get-OrganizationConfig | Format-List *mapihttpenabled
```

## 2. Próximos Passos
Após realizar todas as configurações no servidor Exchange antigo:
1. Retorne ao novo servidor
2. Continue com a instalação do Exchange Server 2019
3. Monitore os logs de instalação

## 🌐 Configuração de URLs

### 🔗 6.1. Configurar URLs Internas e Externas
**Descrição**: Configuração dos endpoints de acesso para os diferentes serviços do Exchange.

> [!WARNING]
> - Substitua "EX01" pelo nome real do seu servidor
> - Substitua "mail.techijack.live" pelo FQDN real do seu ambiente
> - Certifique-se de que o certificado SSL cobre todos os FQDNs configurados

**Procedimento**:
```powershell
$Server_name = "EX01"
$FQDN = "mail.techijack.live"

# Configurar OWA (Outlook Web App)
Get-OWAVirtualDirectory -Server $Server_name | Set-OWAVirtualDirectory -InternalURL "https://$($FQDN)/owa" -ExternalURL "https://$($FQDN)/owa"

# Configurar ECP (Exchange Control Panel)
Get-ECPVirtualDirectory -Server $Server_name | Set-ECPVirtualDirectory -InternalURL "https://$($FQDN)/ecp" -ExternalURL "https://$($FQDN)/ecp"

# Configurar OAB (Offline Address Book)
Get-OABVirtualDirectory -Server $Server_name | Set-OABVirtualDirectory -InternalURL "https://$($FQDN)/oab" -ExternalURL "https://$($FQDN)/oab"

# Configurar ActiveSync
Get-ActiveSyncVirtualDirectory -Server $Server_name | Set-ActiveSyncVirtualDirectory -InternalURL "https://$($FQDN)/Microsoft-Server-ActiveSync" -ExternalURL "https://$($FQDN)/Microsoft-Server-ActiveSync"

# Configurar EWS (Exchange Web Services)
Get-WebServicesVirtualDirectory -Server $Server_name | Set-WebServicesVirtualDirectory -InternalURL "https://$($FQDN)/EWS/Exchange.asmx" -ExternalURL "https://$($FQDN)/EWS/Exchange.asmx"

# Configurar MAPI
Get-MapiVirtualDirectory -Server $Server_name | Set-MapiVirtualDirectory -InternalURL "https://$($FQDN)/mapi" -ExternalURL "https://$($FQDN)/mapi"

# Configurar Autodiscover
Set-ClientAccessService -Identity $Server_name -AutodiscoverServiceInternalURI "https://$FQDN/Autodiscover/Autodiscover.xml"
```

**Verificação**:
```powershell
# Verificar todas as configurações
$OWA = Get-OWAVirtualDirectory -Server $Server_name -AdPropertiesOnly | Select InternalURL,ExternalURL
$ECP = Get-ECPVirtualDirectory -Server $Server_name -AdPropertiesOnly | Select InternalURL,ExternalURL
$OAB = Get-OABVirtualDirectory -Server $Server_name -AdPropertiesOnly | Select InternalURL,ExternalURL
$EAS = Get-ActiveSyncVirtualDirectory -Server $Server_name -AdPropertiesOnly | Select InternalURL,ExternalURL
$MAPI = Get-MAPIVirtualDirectory -Server $Server_name -AdPropertiesOnly | Select InternalURL,ExternalURL
$Auto_Discover = Get-ClientAccessServer $Server_name | Select AutoDiscoverServiceInternalUri

$OWA,$ECP,$OAB,$EAS,$MAPI | Format-Table
$Auto_Discover
```

## 🔒 Certificados SSL

### 📜 7.1. Exportar Certificado
**Descrição**: Processo de exportação do certificado SSL do servidor Exchange antigo.

**Procedimento**:
1. Abra o Exchange Admin Center (EAC)
2. Navegue até Servidores > Certificados
3. Selecione o certificado em uso
4. Clique em "..." e escolha "Exportar"
5. Escolha um local seguro e defina uma senha forte
6. Salve o arquivo .pfx

> [!WARNING]
> - Mantenha a senha em local seguro
> - Certifique-se de que o certificado não está expirado

### 📜 7.2. Importar Certificado
**Descrição**: Processo de importação do certificado no novo servidor Exchange 2019.

**Procedimento**:
1. Copie o arquivo .pfx para o novo servidor
2. Abra o EAC no Exchange 2019
3. Navegue até Servidores > Certificados
4. Clique em "+" para adicionar
5. Escolha "Importar certificado"
6. Selecione o arquivo .pfx
7. Digite a senha de exportação
8. Atribua os serviços:
   - IIS (para OWA, ECP, ActiveSync, etc.)
   - SMTP (para envio/recebimento de emails)
   - IMAP (se utilizado)
   - POP (se utilizado)

> [!NOTE]
> **Verificação**:
> - Teste o acesso HTTPS aos serviços
> - Verifique o certificado no navegador
> - Confirme que não há alertas de certificado

## 💾 Configuração de Armazenamento

### 📊 8.1. Verificar Bancos de Dados
**Descrição**: Levantamento dos bancos de dados existentes e suas configurações.

**Procedimento**:
```powershell
# Abrir o PowerShell do Exchange e Importar esse Módulo
Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn

# Verificar bancos atuais
Get-MailboxDatabase | Format-List Name, EdbFilePath, LogFolderPath
```
### 📊 8.2. Renomear Banco de Dados
**Descrição**: Padronização do nome do banco de dados para melhor gerenciamento.

**Procedimento**:
```powershell
# Exemplo de renomeação para cada banco
Set-MailboxDatabase "DB-EX16-01" –Name "DB-EX19-PROD"
Set-MailboxDatabase "DB-EX16-RH" –Name "DB-EX19-RH"
Set-MailboxDatabase "DB-EX16-ADM" –Name "DB-EX19-ADM"
```

> [!WARNING]
> - Substitua "DB-EX16-01", "DB-EX16-RH", "DB-EX16-ADM" pelos nomes reais dos seus bancos
> - O novo padrão de nomenclatura usa:
>     - DB: Database
>     - EX19: Exchange 2019
>     - PROD/RH/ADM: Identificador do ambiente/departamento

### 📊 8.3. Mover Caminhos
**Descrição**: Configuração dos caminhos para arquivos de banco de dados e logs.

**Procedimento**:
```powershell
# Exemplo para cada banco
Move-DatabasePath DB-EX19-PROD -EdbFilePath E:\DB-EX19-PROD\DB-EX19-PROD.edb –LogFolderPath F:\LOGS\DB-EX19-PROD
Move-DatabasePath DB-EX19-RH -EdbFilePath E:\DB-EX19-RH\DB-EX19-RH.edb –LogFolderPath F:\LOGS\DB-EX19-RH
Move-DatabasePath DB-EX19-ADM -EdbFilePath E:\DB-EX19-ADM\DB-EX19-ADM.edb –LogFolderPath F:\LOGS\DB-EX19-ADM
```

> [!WARNING]
> 1. Estrutura dos diretórios:
>      - Arquivos .edb:
>        - `E:\DB-EX19-PROD\DB-EX19-PROD.edb`
>        - `E:\DB-EX19-RH\DB-EX19-RH.edb`
>        - `E:\DB-EX19-ADM\DB-EX19-ADM.edb`
>      - Arquivos de Log:
>        - `F:\LOGS\DB-EX19-PROD`
>        - `F:\LOGS\DB-EX19-RH`
>        - `F:\LOGS\DB-EX19-ADM`
> 2. Requisitos:
>      - Use discos dedicados para .edb
>      - Mantenha logs em disco separado
>      - Monitore espaço em ambos volumes

### 📊 8.4. Verificar Alterações
**Descrição**: Confirmação das alterações realizadas.

**Procedimento**:
```powershell
Get-MailboxDatabase | Format-List Name, EdbFilePath, LogFolderPath
```

## 🌐 Configuração DNS

**Descrição**: Configuração dos registros DNS necessários para o funcionamento do Exchange 2019.

**Procedimento**:
1. Abra o DNS Manager
2. Adicione registros Host A:
   - "Mail" apontando para o IP do servidor EX2019
   - "Autodiscover" apontando para o IP do servidor EX2019
   - "Outlook" apontando para o IP do servidor EX2019
3. Verifique/Atualize TTL dos registros

**Verificação**:
- Use nslookup para testar os registros
- Verifique propagação do DNS

## 📦 Migração de Caixas de Correio Exchange

> [!NOTE]
> **Exemplo de Bancos de Dados**
>  - No exemplo abaixo, estamos migrando de:
>    - `DB-EX16-01` (Banco 1 do Exchange 2016)
>    - `DB-EX16-RH` (Banco 2 do Exchange 2016)
>    - `DB-EX16-ADM` (Banco 3 do Exchange 2016)
>  - Para:
>    - `DB-EX19-PROD` (Novo banco no Exchange 2019)
>    - `DB-EX19-RH` (Novo banco RH no Exchange 2019)
>    - `DB-EX19-ADM` (Novo banco ADM no Exchange 2019)

### 📋 Processo de Migração

#### 1️⃣ Migração das Caixas de Correio do Sistema

##### 1.1. Verificação Inicial
```powershell
# Verificar caixas de arbitragem em cada banco
Get-Mailbox -Database "DB-EX16-01" -Arbitration     # Banco Principal
Get-Mailbox -Database "DB-EX16-RH" -Arbitration     # Banco RH
Get-Mailbox -Database "DB-EX16-ADM" -Arbitration    # Banco Administrativo
```

##### 1.2. Migração de Arbitragem
```powershell
# Mover caixas de arbitragem para novos bancos
Get-Mailbox -Database "DB-EX16-01" -Arbitration | 
    New-MoveRequest -TargetDatabase "DB-EX19-PROD" -BatchName "Migração Arbitragem PROD"

Get-Mailbox -Database "DB-EX16-RH" -Arbitration | 
    New-MoveRequest -TargetDatabase "DB-EX19-RH" -BatchName "Migração Arbitragem RH"

Get-Mailbox -Database "DB-EX16-ADM" -Arbitration | 
    New-MoveRequest -TargetDatabase "DB-EX19-ADM" -BatchName "Migração Arbitragem ADM"
```

#### 2️⃣ Migração das Caixas de Correio de Usuários

> [!WARNING]
> 1. **Ordem de Migração**:
>      - Primeiro: Caixas de arbitragem
>      - Segundo: Caixas de usuários em lotes
>      - Terceiro: Caixas com problemas (usando BadItemLimit)

##### 2.1. Migração Padrão
```powershell
# Migrar caixas de usuários
Get-Mailbox -Database "DB-EX16-01" -RecipientTypeDetails UserMailbox | 
    New-MoveRequest -TargetDatabase "DB-EX19-PROD" -BatchName "Migração Usuários PROD"

Get-Mailbox -Database "DB-EX16-RH" -RecipientTypeDetails UserMailbox | 
    New-MoveRequest -TargetDatabase "DB-EX19-RH" -BatchName "Migração Usuários RH"

Get-Mailbox -Database "DB-EX16-ADM" -RecipientTypeDetails UserMailbox | 
    New-MoveRequest -TargetDatabase "DB-EX19-ADM" -BatchName "Migração Usuários ADM"
```

##### 2.2. Migração com BadItemLimit
```powershell
# Para caixas com itens corrompidos
Get-Mailbox -Database "DB-EX16-01" -RecipientTypeDetails UserMailbox | 
    New-MoveRequest -TargetDatabase "DB-EX19-PROD" -BatchName "Migração Usuários PROD BadItem" -BadItemLimit 50

Get-Mailbox -Database "DB-EX16-RH" -RecipientTypeDetails UserMailbox | 
    New-MoveRequest -TargetDatabase "DB-EX19-RH" -BatchName "Migração Usuários RH BadItem" -BadItemLimit 50

Get-Mailbox -Database "DB-EX16-ADM" -RecipientTypeDetails UserMailbox | 
    New-MoveRequest -TargetDatabase "DB-EX19-ADM" -BatchName "Migração Usuários ADM BadItem" -BadItemLimit 50
```

#### 3️⃣ Monitoramento e Verificação

##### 3.1. Status das Migrações
```powershell
# Verificar progresso
Get-MoveRequest | Get-MoveRequestStatistics

# Limpar solicitações concluídas
Get-MoveRequest | Where-Object {$_.Status -eq "Completed"} | Remove-MoveRequest
```

##### 3.2. Verificação Final
```powershell
# Verificar bancos originais
Get-Mailbox -Database "DB-EX16-01"
Get-Mailbox -Database "DB-EX16-RH"
Get-Mailbox -Database "DB-EX16-ADM"

# Verificar novos bancos
Get-MailboxDatabase "DB-EX19-PROD" | Format-List Name, ServerName, EdbFilePath, LogFolderPath
Get-MailboxDatabase "DB-EX19-RH" | Format-List Name, ServerName, EdbFilePath, LogFolderPath
Get-MailboxDatabase "DB-EX19-ADM" | Format-List Name, ServerName, EdbFilePath, LogFolderPath
```

## 🔄 Configuração de Conectores

### 📨 Migração de Conectores de Envio
1. Acessar Exchange Admin Center (EAC)
2. Navegar até Fluxo de Email > Conectores de Envio
3. Para cada conector:
   - Editar o conector existente
   - Adicionar novo servidor Exchange 2019
   - Remover servidor Exchange 2016
   - Salvar alterações
4. Testar fluxo de email após cada alteração

### 📥 Configuração de Conectores de Recebimento

#### 1. Identificar Conectores no Exchange 2016
```powershell
# Listar conectores existentes
Get-ReceiveConnector | Format-List Identity, Name, Bindings, RemoteIPRanges, AuthMechanism

# Exportar configurações
Get-ReceiveConnector | Export-CliXml C:\ConnectorSettings.xml
```

#### 2. Criar Conectores no Exchange 2019

1. **Criar Conector Anônimo**:
```powershell
New-ReceiveConnector -Name "Anonymous Connector" -Usage Custom -Bindings '0.0.0.0:25' -Server EX19-SERVER -RemoteIPRanges "0.0.0.0-255.255.255.255"
```

> [!WARNING]
> **Substituições necessárias:**
> - `"Anonymous Connector"` → Nome desejado para o conector (Ex: "Conector Anônimo SMTP")
> - `'0.0.0.0:25'` → IP e porta do seu servidor
> - `EX19-SERVER` → Nome do seu servidor Exchange 2019
> - `"0.0.0.0-255.255.255.255"` → Range de IPs da sua rede

2. **Configurar Permissões Anônimas**:
```powershell
Set-ReceiveConnector "Anonymous Connector" -PermissionGroups AnonymousUsers
```

> [!WARNING]
> **Substituição necessária:**
> - `"Anonymous Connector"` → Use o mesmo nome definido no comando anterior

3. **Criar Conector Autenticado**:
```powershell
New-ReceiveConnector -Name "Authenticated Connector" -Usage Custom -Bindings '0.0.0.0:587' -Server EX19-SERVER -RemoteIPRanges "0.0.0.0-255.255.255.255"
```

> [!WARNING]
> **Substituições necessárias:**
> - `"Authenticated Connector"` → Nome desejado para o conector (Ex: "Conector Autenticado SMTP")
> - `'0.0.0.0:587'` → IP e porta do seu servidor
> - `EX19-SERVER` → Nome do seu servidor Exchange 2019
> - `"0.0.0.0-255.255.255.255"` → Range de IPs da sua rede

4. **Configurar Autenticação**:
```powershell
Set-ReceiveConnector "Authenticated Connector" -AuthMechanism Tls,Basic,Integrated -PermissionGroups ExchangeUsers
```

> [!WARNING]
> **Substituição necessária:**
> - `"Authenticated Connector"` → Use o mesmo nome definido no comando anterior

5. **Adicionar Servidor Confiável**:
```powershell
Get-ExchangeServer | Add-ADPermission -User "NT AUTHORITY\ANONYMOUS LOGON" -ExtendedRights MS-Exc-Store-Admin
```

6. **Configurar Permissões de Conector**:
```powershell
Get-ReceiveConnector | Add-ADPermission -User "NT AUTHORITY\ANONYMOUS LOGON" -ExtendedRights MS-Exc-Accept-Headers-Routing,MS-Exc-Accept-Headers-Forest,MS-Exc-Accept-Headers-Organization
```

7. **Estabelecer Confiança**:
```powershell
Set-ADSiteLink -Identity "Default-First-Site-Link" -ReplicationInterval 15
```

> [!WARNING]
> **Substituição necessária:**
> - `"Default-First-Site-Link"` → Nome do seu Site Link do Active Directory

8. **Reiniciar Serviço**:
```powershell
Restart-Service MSExchangeTransport
```

9. **Verificar Status**:
```powershell
Get-Service MSExchangeTransport | Format-List Name, Status, DisplayName
```

### 📩 Testes de Conectores via PowerShell

1. **Teste com Autenticação**:
```powershell
Send-MailMessage -From 'remetente@seudominio.com.br' -To 'destinatario@seudominio.com.br' -Subject 'Teste Auth' -smtpserver IP-DO-SERVIDOR -port 25 -usessl -credential pscredential
```

> [!WARNING]
> **Substituições necessárias:**
> - `remetente@seudominio.com.br` → Email do remetente em seu domínio
> - `destinatario@seudominio.com.br` → Email do destinatário
> - `IP-DO-SERVIDOR` → IP do seu servidor Exchange
> - `port 25` → Porta configurada (25 ou 587)
>
> **Configurações importantes:**
> - Use endereços de email válidos no seu domínio
> - Configure credenciais antes do teste autenticado:
>   - `$cred = Get-Credential`

2. **Teste sem Autenticação**:
```powershell
Send-MailMessage -From 'relay@seudominio.com.br' -To 'destinatario@seudominio.com.br' -Subject 'Teste Relay' -body 'Mensagem de Teste - Favor desconsiderar' -smtpserver IP-DO-SERVIDOR
```

> [!WARNING]
> **Substituições necessárias:**
> - `relay@seudominio.com.br` → Email usado para relay
> - `destinatario@seudominio.com.br` → Email do destinatário
> - `IP-DO-SERVIDOR` → IP do seu servidor Exchange

### 📧 Verificação de Filas de Email

#### 1️⃣ Verificar Todas as Filas
```powershell
# Listar todas as filas
Get-Queue

# Verificar filas com detalhes
Get-Queue | Format-List Name,MessageCount,Status,NextHopDomain
```

#### 3️⃣ Verificar Filas com Problemas
```powershell
# Verificar filas com status de erro
Get-Queue | Where {$_.Status -eq "Retry"} | Format-List

# Verificar filas paradas
Get-Queue | Where {$_.Status -eq "Suspended"}
```

## 🗑️ Desinstalação do Exchange Server

Após a migração completa para o Exchange 2019 e a verificação adequada de todas as filas de email, o próximo passo é a remoção segura do Exchange Server antigo.

> [!WARNING]
> - Certifique-se de que todas as caixas de correio foram migradas
> - Verifique se não há dependências no servidor antigo

### 📋 1. Verificação Inicial de Bancos de Dados

#### 1.1. Listar Bancos de Dados
**Descrição**: Identificar todos os bancos de dados existentes no servidor Exchange antigo.

**Procedimento**:
```powershell
# Listar todos os bancos de dados
Get-MailboxDatabase | Format-Table Name, Server, MountStatus
```

### 💾 2. Desmontagem e Remoção de Bancos de Dados

#### 2.1. Desmontar Bancos de Dados
**Descrição**: Desmontar todos os bancos de dados antes da remoção.

**Procedimento**:
```powershell
# Desmontar banco de dados específico
Dismount-Database "NomeDoBanco" -Confirm:$false

# Desmontar todos os bancos de dados do servidor
Get-MailboxDatabase -Server "NomeDoServidor" | Dismount-Database -Confirm:$false
```

#### 2.2. Remover Bancos de Dados
**Descrição**: Remover os bancos de dados do servidor Exchange.

**Procedimento**:
```powershell
# Remover banco de dados específico
Remove-MailboxDatabase "NomeDoBanco" -Confirm:$false

# Remover todos os bancos de dados do servidor
Get-MailboxDatabase -Server "NomeDoServidor" | Remove-MailboxDatabase -Confirm:$false
```

> [!WARNING]
> Em caso de erros na remoção de bancos de dados:
> - Verifique se não há caixas de correio ativas
> - Verifique se o banco está realmente desmontado
> - Se o banco estiver corrompido, considere usar o parâmetro `-RemoveBrokenDatabaseCopies` (se disponível na sua versão)
> - Como último recurso, use o ADSIEdit para remover referências a bancos de dados problemáticos

### 🧹 3. Remover Banco Manual usando ADSIEdit (se necessário)

Se houver problemas na desinstalação ou referências remanescentes no Active Directory, pode ser necessário usar o ADSIEdit para limpeza manual.

#### 3.1. Remover Manualmente Bancos de Dados Problemáticos
**Descrição**: Usar o ADSIEdit para remover referências a bancos de dados que não podem ser excluídos pelos comandos normais.

**Procedimento**:
1. Abra o ADSIEdit (Execute `adsiedit.msc` no prompt de comando)
2. Conecte-se a "Configuração"
3. Navegue para: 
   ```
   CN=Configuration > CN=Services > CN=Microsoft Exchange > CN=SuaOrganização > CN=Databases
   ```
4. Localize o objeto CN=Mailbox Database (ex: CN=DB01)
5. Clique com o botão direito e selecione Excluir

### 🖥️ 4. Desinstalar o Exchange Server
**Descrição**: Desinstalar o software Exchange Server através do Painel de Controle.

**Procedimento**:
1. Abra o Painel de Controle > Programas e Recursos
2. Localize "Microsoft Exchange Server 2016"
3. Selecione e clique em "Desinstalar"
4. Siga as instruções do assistente de desinstalação
5. Reinicie o servidor após a conclusão

![Desinstalação do Exchange Server via Painel de Controle](https://github.com/user-attachments/assets/15462d67-2b43-425d-80ce-885df00017a0)

> [!NOTE]
> A imagem acima mostra a tela de desinstalação do Exchange Server 2016 através do Painel de Controle do Windows.

### 🔄 5. Remover o Servidor do Domínio
**Descrição**: Após a desinstalação do Exchange 2016, remover o servidor do domínio para completar a limpeza.

**Procedimento**:
1. Abra as Propriedades do Sistema:
   - Clique com o botão direito em "Este Computador"
   - Selecione "Propriedades"
   - Clique em "Alterar configurações" na seção de nome do computador
   
2. Na janela de Propriedades do Sistema:
   - Clique na aba "Nome do Computador"
   - Clique no botão "Alterar..."

3. Na janela "Alterações de Nome/Domínio do Computador":
   - Na seção "Membro de", selecione a opção "Grupo de Trabalho"
   - Digite um nome para o grupo de trabalho (exemplo: "WORKGROUP")
   - Clique em "OK"

4. Forneça as credenciais de administrador do domínio quando solicitado

5. Confirme a remoção do domínio e reinicie o servidor quando solicitado

## 🔄 Versionamento

- Versão: 1.0.0
- Última atualização: 21/03/2025
