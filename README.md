# 🔄 Manual de Instalação, Configuração e Migração do Exchange Server 2019
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Mathews_Buzetti-blue)](https://www.linkedin.com/in/mathewsbuzetti)
[![Azure](https://img.shields.io/badge/Microsoft-Exchange_Server-orange)]()
[![Status](https://img.shields.io/badge/Status-Production-green)]()

## 🎯 Objetivo
Este guia fornece um roteiro detalhado para migração do Microsoft Exchange Server 2016 para o Exchange Server 2019. O processo de migração é crítico e requer planejamento cuidadoso para garantir:
- Mínima interrupção dos serviços de email
- Preservação integral dos dados dos usuários
- Transição suave entre versões do Exchange

## 🚨 Avisos Importantes Antes de Iniciar
> **⚠️ ATENÇÃO**: 
> - Sempre faça backup completo de todo o ambiente antes de iniciar
> - Realize a migração em horário de baixo impacto
> - Tenha um plano de rollback detalhado
> - Comunique antecipadamente todos os usuários sobre a migração

## 📋 Índice
- [Pré-requisitos](#-pré-requisitos)
- [Instalação do Windows Server](#-instalação-do-windows-server)
- [Download do Exchange](#-download-do-exchange)
- [Preparação do Active Directory](#-preparação-do-active-directory)
- [Instalação do Exchange](#-instalação-do-exchange)
- [Configuração de URLs](#-configuração-de-urls)
- [Certificados SSL](#-certificados-ssl)
- [Configuração de Armazenamento](#-configuração-de-armazenamento)
- [Configuração DNS](#-configuração-dns)
- [Migração](#-migração)
- [Pós-Migração](#-pós-migração)

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
4. ⚠️ **IMPORTANTE**: Reinicie o servidor após a instalação

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

⚠️ **IMPORTANTE**: Certifique-se de baixar a versão mais recente disponível no momento da instalação.

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

⚠️ **IMPORTANTE**: 
- O nome da organização não pode ser alterado depois de definido
- Use um nome sem espaços

### 🛠️ 4.3. Preparar Todos os Domínios
**Descrição**: Prepara todos os domínios da floresta para o Exchange 2019.

**Procedimento**:
```cmd
Setup.exe /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAllDomains
```

**Verificação**:
- Aguarde a replicação do AD (pode levar várias horas)
- Verifique os logs de instalação
- Confirme as alterações no AD Sites and Services

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

**Verificações Pós-Instalação**:
- Verifique o Event Viewer
- Teste o acesso ao ECP (Exchange Control Panel)
- Verifique os serviços do Exchange

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

⚠️ **IMPORTANTE**: 
- Substitua "EX01" pelo nome real do seu servidor
- Substitua "mail.techijack.live" pelo FQDN real do seu ambiente
- Certifique-se de que o certificado SSL cobre todos os FQDNs configurados

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

⚠️ **IMPORTANTE**: 
- Mantenha a senha em local seguro
- Certifique-se de que o certificado não está expirado
- Verifique se todos os nomes alternativos (SANs) necessários estão incluídos

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

**Verificação**:
- Teste o acesso HTTPS aos serviços
- Verifique o certificado no navegador
- Confirme que não há alertas de certificado

## 💾 Configuração de Armazenamento

### 📊 8.1. Verificar Bancos de Dados
**Descrição**: Levantamento dos bancos de dados existentes e suas configurações.

**Procedimento**:
```powershell
# Carregar PowerShell Exchange
Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn

# Verificar bancos atuais
Get-MailboxDatabase | Format-List Name, EdbFilePath, LogFolderPath
```

### 📊 8.2. Renomear Banco de Dados
**Descrição**: Padronização do nome do banco de dados para melhor gerenciamento.

**Procedimento**:
```powershell
Set-MailboxDatabase "Mailbox Database 0582773279" –Name "DB01-2019"
```

      > **📝 NOTA IMPORTANTE**: 
      > - O nome "Mailbox Database 0582773279" é APENAS UM EXEMPLO.
      > - SUBSTITUA pelo nome real do banco de dados identificado no passo de verificação (Get-MailboxDatabase).
      > - "DB01-2019" é um nome sugerido. Escolha um nome significativo para sua organização.

### 📊 8.3. Mover Caminhos
**Descrição**: Configuração dos caminhos otimizados para arquivos de banco de dados e logs.

**Procedimento**:
```powershell
Move-DatabasePath DB01-2019 -EdbFilePath E:\DB01-2019\DB01-2019.edb –LogFolderPath F:\LOGS\DB01-2019
```

**Melhores Práticas**:
- Use discos separados para DB e logs
- Configure RAID apropriado
- Monitore espaço em disco
- Mantenha backup atualizado

### 📊 8.4. Verificar Alterações
**Descrição**: Confirmação das alterações realizadas.

**Procedimento**:
```powershell
Get-MailboxDatabase | Format-List Name, EdbFilePath, LogFolderPath
```

## 🌐 Configuração DNS

**Descrição**: Configuração dos registros DNS necessários para o funcionamento do Exchange.

**Procedimento**:
1. Abra o DNS Manager
2. Adicione registros Host A:
   - "Mail" apontando para o IP do servidor
   - "Autodiscover" apontando para o IP do servidor
3. Verifique/Atualize TTL dos registros

**Verificação**:
- Use nslookup para testar os registros
- Verifique propagação do DNS
- Teste Autodiscover com clientes Outlook

## 📦 Migração

### 📨 10.1. Migração das Caixas de Correio do Sistema
**Descrição**: Migração das caixas de correio de arbitragem e sistema.

**Procedimento**:
```powershell
# Verificar caixas de arbitragem
Get-Mailbox -Database "Mailbox Database 1398699602" -Arbitration
Get-Mailbox -Database "dtb1" -Arbitration
Get-Mailbox -Database "Mailbox Database 1366087898" -Arbitration

# Mover caixas de arbitragem
Get-Mailbox -Database "Mailbox Database 1398699602" -Arbitration | 
    New-MoveRequest -TargetDatabase DB01-2019 -BatchName "Migração Arbitragem 1398699602"
```

⚠️ **IMPORTANTE**: 
- Faça a migração em lotes pequenos
- Monitore o progresso
- Mantenha logs detalhados

### 👥 10.2. Migração das Caixas de Correio de Usuários
**Descrição**: Migração das caixas de correio dos usuários finais.

**Procedimento**:
```powershell
# Migração padrão
Get-Mailbox -Database "Mailbox Database 1398699602" -RecipientTypeDetails UserMailbox | 
    New-MoveRequest -TargetDatabase DB01-2019 -BatchName "Migração Usuários 1398699602"

# Com tratamento de itens corrompidos
Get-Mailbox -Database "Mailbox Database 1398699602" -RecipientTypeDetails UserMailbox | 
    New-MoveRequest -TargetDatabase DB01-2019 -BatchName "Migração Usuários 1398699602 com BadItem" -BadItemLimit 50
```

**Melhores Práticas**:
- Comunique os usuários
- Planeje janelas de manutenção
- Prepare plano de rollback
- Monitore performance do servidor

### 📊 10.3. Monitoramento da Migração
**Descrição**: Acompanhamento do progresso e status das migrações.

**Procedimento**:
```powershell
# Verificar status geral
Get-MoveRequest | Get-MoveRequestStatistics

# Limpar solicitações concluídas
Get-MoveRequest | Where-Object {$_.Status -eq "Completed"} | Remove-MoveRequest
```

### ✅ 10.4. Verificação Final
**Descrição**: Confirmação da conclusão da migração.

**Procedimento**:
```powershell
# Verificar caixas restantes em cada banco
Get-Mailbox -Database "Mailbox Database 1398

