# AWS-Resource-Management-with-Tagging
Este repositório contém um laboratório completo para aprendizado de gerenciamento de recursos AWS usando tags (marcações)


[![AWS](https://img.shields.io/badge/AWS-EC2-orange?logo=amazon-aws)](https://aws.amazon.com/ec2/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![PHP](https://img.shields.io/badge/PHP-7.4+-777BB4?logo=php)](https://php.net)
[![Bash](https://img.shields.io/badge/Bash-5.0+-4EAA25?logo=gnu-bash)](https://www.gnu.org/software/bash/)

## 📌 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [O laboratório demonstra como](#-o-laboratório-demonstra-como)
- [Objetivos de Aprendizado](#-objetivos-de-aprendizado)
- [Arquitetura do Laboratório](#-arquitetura-do-laboratório)
- [Recursos do Ambiente](#recursos-do-ambiente)
- [Pré-requisitos](#-pré-requisitos)
  - [Software Necessário](#software-necessário)
  - [Acesso AWS](#acesso-aws)
- [Configuração do Ambiente](#-configuração-do-ambiente)
  - [1. Clone o repositório](#1-clone-o-repositório)
  - [2. Configure as permissões dos scripts](#2-configure-as-permissões-dos-scripts)
  - [3. Conecte-se à Command Host](#3-conecte-se-à-command-host-ambiente-de-laboratório)
- [Tarefas do Laboratório](#-tarefas-do-laboratório)
  - [Tarefa 1: Inspecionar e Alterar Tags](#tarefa-1-inspecionar-e-alterar-tags)
    - [1.1 Consultas Básicas com AWS CLI](#11-consultas-básicas-com-aws-cli)
    - [1.2 Consultas Avançadas com JMESPath](#12-consultas-avançadas-com-jmespath)
    - [1.3 Atualização em Lote de Tags](#13-atualização-em-lote-de-tags)
  - [Tarefa 2: Parar e Iniciar Instâncias por Tag](#tarefa-2-parar-e-iniciar-instâncias-por-tag)
    - [O Script Stopinator](#o-script-stopinator-scriptsstopinatorphp)
  - [Tarefa 3 (Desafio): Política Tag-Or-Terminate](#tarefa-3-desafio-política-tag-or-terminate)
    - [Cenário](#cenário)
    - [Solução Implementada](#solução-implementada)
- [Comandos Essenciais](#-comandos-essenciais)
  - [AWS CLI - Filtros por Tag](#aws-cli---filtros-por-tag)
  - [JMESPath - Consultas Avançadas](#jmespath---consultas-avançadas)
  - [Operações em Lote](#operações-em-lote)
- [Scripts Explicados](#-scripts-explicados)
  - [1. change-resource-tags.sh](#1-change-resource-tagssh)
  - [2. stopinator.php](#2-stopinatorphp)
  - [3. terminate-instances.php](#3-terminate-instancesphp)
- [Boas Práticas de Tagging](#-boas-práticas-de-tagging)
  - [Tags Recomendadas pela AWS](#tags-recomendadas-pela-aws)
  - [Estratégias de Governança](#estratégias-de-governança)
  - [Automação com Lambda + CloudWatch](#automação-com-lambda--cloudwatch)
- [Solução de Problemas](#-solução-de-problemas)
  - [Erro: "UnauthorizedOperation"](#erro-unauthorizedoperation)
  - [Erro: "InvalidInstanceID.NotFound"](#erro-invalidinstanceidnotfound)
  - [JMESPath não retorna resultados](#jmespath-não-retorna-resultados)
  - [Script PHP não executa](#script-php-não-executa)
- [Referências](#-referências)
  - [Documentação Oficial](#documentação-oficial)
- [Licença](#-licença)

---

## 🎯 O laboratório demonstra como:

- ✅ Aplicar tags a recursos existentes da AWS
- ✅ Encontrar recursos baseados em suas tags usando filtros e JMESPath
- ✅ Executar operações em lote (start/stop/terminate) baseadas em tags
- ✅ Implementar políticas automatizadas como "tag-or-terminate"

### 🎓 Objetivos de Aprendizado

Ao final deste laboratório, você será capaz de:

1. Usar a AWS CLI para consultar recursos por tags
2. Aplicar consultas avançadas com **JMESPath**
3. Automatizar operações em lote com scripts Bash e PHP
4. Implementar governança de recursos usando tagging

---

## 🏗️ Arquitetura do Laboratório
┌─────────────────────────────────────────────────────────────┐
│ AWS Cloud │
│ ┌──────────────────────────────────────────────────────┐ │
│ │ Lab VPC (vpc-xxxx) │ │
│ │ ┌──────────────────┐ ┌──────────────────┐ │ │
│ │ │ Public Subnet │ │ Private Subnet │ │ │
│ │ │ │ │ │ │ │
│ │ │ ┌──────────────┐ │ │ ┌──────────────┐ │ │ │
│ │ │ │Command Host │ │ │ │ App Server 1│ │ │ │
│ │ │ │ (EC2) │◄├────────┼─┤ (t3.micro) │ │ │ │
│ │ │ │ AWS CLI │ │ │ │ Project=ERP │ │ │ │
│ │ │ │ PHP SDK │ │ │ │ Env=dev │ │ │ │
│ │ │ └──────────────┘ │ │ └──────────────┘ │ │ │
│ │ │ │ │ │ │ │
│ │ │ │ │ ┌──────────────┐ │ │ │
│ │ │ │ │ │ App Server 2│ │ │ │
│ │ │ │ │ │ (t3.micro) │ │ │ │
│ │ │ │ │ │ Project=ERP │ │ │ │
│ │ │ │ │ │ Env=prod │ │ │ │
│ │ │ │ │ └──────────────┘ │ │ │
│ │ └──────────────────┘ └──────────────────┘ │ │
│ └──────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘


### Recursos do Ambiente

| Recurso | Quantidade | Descrição |
|---------|------------|-----------|
| Command Host | 1 | Instância com AWS CLI e PHP SDK pré-instalados |
| App Servers | 8 | Instâncias Linux com tags personalizadas |
| Tags utilizadas | 3 | Project, Environment, Version |

---

## 📋 Pré-requisitos

### Software Necessário

| Ferramenta | Versão | Instalação |
|------------|--------|-------------|
| AWS CLI | 2.x+ | `pip install awscli` |
| PHP | 7.4+ | `sudo apt install php` (Linux) |
| SSH Client | - | PuTTY (Win) / Terminal (Mac/Linux) |
| Git | 2.x+ | [git-scm.com](https://git-scm.com) |

### Acesso AWS

- Conta AWS com permissões para EC2
- Par de chaves SSH (.pem ou .ppk)
- Região configurada (ex: `us-west-2`)

---

```markdown
## ⚙️ Configuração do Ambiente

### 1. Clone o repositório

```bash
git clone https://github.com/DevAmeliaViana/AWS-Resource-Management-with-Tagging.git
cd aws-tagging-management-lab
```

### 2. Configure as permissões dos scripts

```bash
chmod +x scripts/*.sh
chmod +x scripts/*.php
```

### 3. Conecte-se à Command Host (ambiente de laboratório)

**Linux/Mac:**
```bash
ssh -i labsuser.pem ec2-user@<PUBLIC_IP>
```

**Windows (PuTTY):**
- Use o arquivo `labsuser.ppk`
- Host: `ec2-user@<PUBLIC_IP>`

## 🚀 Tarefas do Laboratório

### Tarefa 1: Inspecionar e Alterar Tags

#### 1.1 Consultas Básicas com AWS CLI

```bash
# Listar todas instâncias do projeto ERPSystem
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem"

# Retornar apenas IDs das instâncias
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].InstanceId'
```

#### 1.2 Consultas Avançadas com JMESPath

```bash
# Retornar ID + Zona de Disponibilidade
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].{ID:InstanceId, AZ:Placement.AvailabilityZone}'

# Incluir valor da tag Project
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].{
    ID:InstanceId,
    AZ:Placement.AvailabilityZone,
    Project:Tags[?Key==`Project`] | [0].Value
  }'

# Consulta completa com todas as tags
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].{
    ID:InstanceId,
    AZ:Placement.AvailabilityZone,
    Project:Tags[?Key==`Project`] | [0].Value,
    Environment:Tags[?Key==`Environment`] | [0].Value,
    Version:Tags[?Key==`Version`] | [0].Value
  }'
```

#### 1.3 Atualização em Lote de Tags

**Script:** `scripts/change-resource-tags.sh`

```bash
#!/bin/bash
# Altera a tag Version de 1.0 para 1.1 em todas instâncias de desenvolvimento

ids=$(aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
           "Name=tag:Environment,Values=development" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

aws ec2 create-tags --resources $ids --tags 'Key=Version,Value=1.1'

echo "Tags atualizadas para: $ids"
```

**Execução:**
```bash
./scripts/change-resource-tags.sh
```

### Tarefa 2: Parar e Iniciar Instâncias por Tag

#### O Script Stopinator (`scripts/stopinator.php`)

Este script usa o AWS SDK for PHP para gerenciar instâncias baseado em tags.

**Uso:**
```bash
# Parar instâncias
./scripts/stopinator.php -t"Project=ERPSystem;Environment=development"

# Iniciar instâncias
./scripts/stopinator.php -t"Project=ERPSystem;Environment=development" -s
```

**Explicação do código:**
```php
// Filtra instâncias por tags
$result = $ec2Client->describeInstances([
    'Filters' => [
        ['Name' => 'tag:Project', 'Values' => ['ERPSystem']],
        ['Name' => 'tag:Environment', 'Values' => ['development']]
    ]
]);

// Extrai IDs das instâncias
$instanceIds = [];
foreach ($result['Reservations'] as $reservation) {
    foreach ($reservation['Instances'] as $instance) {
        $instanceIds[] = $instance['InstanceId'];
    }
}

// Executa ação (stop ou start)
if ($startMode) {
    $ec2Client->startInstances(['InstanceIds' => $instanceIds]);
} else {
    $ec2Client->stopInstances(['InstanceIds' => $instanceIds]);
}
```

### Tarefa 3 (Desafio): Política Tag-Or-Terminate

#### Cenário

Sua empresa quer uma política automatizada que encerre qualquer instância que não possua a tag `Environment`.

#### Solução Implementada

**Script:** `scripts/terminate-instances.php`

```php
#!/usr/bin/env php
<?php
require 'vendor/autoload.php';

use Aws\Ec2\Ec2Client;

// Parâmetros
$region = $argv[1] ?? 'us-west-2';
$subnetId = $argv[2] ?? '';

// 1. Obter todas instâncias com tag Environment
$ec2 = new Ec2Client(['region' => $region, 'version' => 'latest']);

$taggedInstances = $ec2->describeInstances([
    'Filters' => [
        ['Name' => 'tag-key', 'Values' => ['Environment']],
        ['Name' => 'subnet-id', 'Values' => [$subnetId]]
    ]
]);

$taggedIds = [];
foreach ($taggedInstances['Reservations'] as $reservation) {
    foreach ($reservation['Instances'] as $instance) {
        $taggedIds[] = $instance['InstanceId'];
    }
}

// 2. Obter TODAS instâncias da sub-rede
$allInstances = $ec2->describeInstances([
    'Filters' => [
        ['Name' => 'subnet-id', 'Values' => [$subnetId]]
    ]
]);

// 3. Identificar instâncias sem tag
$toTerminate = [];
foreach ($allInstances['Reservations'] as $reservation) {
    foreach ($reservation['Instances'] as $instance) {
        if (!in_array($instance['InstanceId'], $taggedIds)) {
            $toTerminate[] = $instance['InstanceId'];
            echo "❌ Instância sem tag: {$instance['InstanceId']}\n";
        }
    }
}

// 4. Encerrar instâncias não conformes
if (!empty($toTerminate)) {
    echo "⚠️ Encerrando " . count($toTerminate) . " instância(s)...\n";
    $ec2->terminateInstances(['InstanceIds' => $toTerminate]);
    echo "✅ Instâncias encerradas.\n";
} else {
    echo "✅ Todas as instâncias estão em conformidade.\n";
}
```

**Execução:**
```bash
./scripts/terminate-instances.php us-west-2 subnet-01a07ff6fa3055b7d
```

## 📝 Comandos Essenciais

### AWS CLI - Filtros por Tag

```bash
# Filtrar por chave e valor
--filter "Name=tag:Environment,Values=production"

# Filtrar apenas pela existência da tag
--filter "Name=tag-key,Values=Environment"

# Múltiplos filtros
--filter "Name=tag:Project,Values=ERPSystem" \
         "Name=tag:Environment,Values=development"

# Filtrar por sub-rede
--filter "Name=subnet-id,Values=subnet-xxxxx"
```

### JMESPath - Consultas Avançadas

| Objetivo | Query |
|----------|-------|
| ID da instância | `Reservations[*].Instances[*].InstanceId` |
| ID + AZ | `Reservations[*].Instances[*].{ID:InstanceId, AZ:Placement.AvailabilityZone}` |
| Valor de tag específica | `Tags[?Key==\`Environment\`] \| [0].Value` |
| Múltiplas tags | `{Proj:Tags[?Key==\`Project\`]\|[0].Value, Env:Tags[?Key==\`Environment\`]\|[0].Value}` |

### Operações em Lote

```bash
# Parar instâncias
aws ec2 stop-instances --instance-ids i-xxx i-yyy

# Iniciar instâncias
aws ec2 start-instances --instance-ids i-xxx i-yyy

# Encerrar instâncias
aws ec2 terminate-instances --instance-ids i-xxx i-yyy

# Adicionar/Atualizar tag
aws ec2 create-tags --resources i-xxx --tags Key=Version,Value=2.0

# Remover tag
aws ec2 delete-tags --resources i-xxx --tags Key=Environment
```

## 📜 Scripts Explicados

### 1. `change-resource-tags.sh`

- **Função:** Atualiza tags em lote
- **Uso:** `./change-resource-tags.sh`
- **O que faz:**
  - Busca instâncias com `Project=ERPSystem` e `Environment=development`
  - Altera a tag `Version` de `1.0` para `1.1`

### 2. `stopinator.php`

- **Função:** Para ou inicia instâncias baseado em tags
- **Uso:** `./stopinator.php -t"tag1=val1;tag2=val2" [-s]`
- **Parâmetros:**
  - `-t`: Tags no formato `nome=valor;nome=valor`
  - `-s`: Modo start (sem ele, modo stop)

### 3. `terminate-instances.php`

- **Função:** Encerra instâncias sem tag `Environment`
- **Uso:** `./terminate-instances.php <region> <subnet-id>`
- **Parâmetros:**
  - `region`: Região AWS (ex: `us-west-2`)
  - `subnet-id`: ID da sub-rede a verificar

## 💡 Boas Práticas de Tagging

### Tags Recomendadas pela AWS

| Tag | Finalidade | Exemplo |
|-----|------------|---------|
| Name | Nome amigável do recurso | web-server-01 |
| Environment | Ambiente (dev/staging/prod) | production |
| Project | Projeto associado | ERPSystem |
| CostCenter | Centro de custo | financeiro |
| Owner | Responsável | team-aws@empresa.com |
| Backup | Política de backup | daily |
| Version | Versão do software | 2.1.0 |

### Estratégias de Governança

```bash
# Política: instâncias sem Owner são encerradas
aws ec2 describe-instances \
  --filter "Name=tag-key,Values=Owner" \
  --query 'Reservations[*].Instances[?!not_null(Tags[?Key==`Owner`].Value)].[InstanceId]'

# Política: instâncias production não podem ser paradas manualmente
aws ec2 describe-instances \
  --filter "Name=tag:Environment,Values=production" \
  --query 'Reservations[*].Instances[*].InstanceId'
```

### Automação com Lambda + CloudWatch

```python
# Exemplo: AWS Lambda para enforce de tags
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    
    # Encontra instâncias sem tag Environment
    instances = ec2.describe_instances(
        Filters=[{'Name': 'tag-key', 'Values': ['Environment']}]
    )
    
    tagged_ids = []
    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            tagged_ids.append(instance['InstanceId'])
    
    # Termina instâncias sem tag
    all_instances = ec2.describe_instances()
    to_terminate = []
    
    for reservation in all_instances['Reservations']:
        for instance in reservation['Instances']:
            if instance['InstanceId'] not in tagged_ids:
                to_terminate.append(instance['InstanceId'])
    
    if to_terminate:
        ec2.terminate_instances(InstanceIds=to_terminate)
        print(f"Terminated: {to_terminate}")
```

## 🔧 Solução de Problemas

### Erro: "UnauthorizedOperation"

- **Causa:** Permissões IAM insuficientes
- **Solução:**
```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeInstances",
    "ec2:CreateTags",
    "ec2:StopInstances",
    "ec2:StartInstances",
    "ec2:TerminateInstances"
  ],
  "Resource": "*"
}
```

### Erro: "InvalidInstanceID.NotFound"

- **Causa:** ID da instância incorreta ou instância já terminada
- **Solução:**
```bash
# Verificar instâncias ativas
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running,pending,stopped"
```

### JMESPath não retorna resultados

- **Causa:** Sintaxe incorreta ou tags com caracteres especiais
- **Solução:** Use escape com backticks:
```bash
--query 'Tags[?Key==`Environment`] | [0].Value'
```

### Script PHP não executa

- **Causa:** Permissões ou dependências faltando
- **Solução:**
```bash
# Verificar PHP
php -v

# Instalar dependências (se aplicável)
composer require aws/aws-sdk-php

# Dar permissão de execução
chmod +x scripts/*.php
```

## 📚 Referências

### Documentação Oficial

- [AWS Tagging Best Practices](https://aws.amazon.com/tagging-best-practices/)
- [JMESPath Tutorial](https://jmespath.org/tutorial.html)
- [AWS CLI EC2 Commands](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/index.html)
- [AWS SDK for PHP](https://docs.aws.amazon.com/sdk-for-php/v3/developer-guide/ec2-examples.html)

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.
