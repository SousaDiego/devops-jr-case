# DevOps Jr Challenge - VHL Sistemas

## Resumo

Este projeto foi desenvolvido como parte do teste técnico para a vaga de DevOps Jr da VHL Sistemas.

O objetivo foi construir um ambiente local completo utilizando Vagrant, Kubernetes e containers, contendo:

- Máquina virtual provisionada automaticamente com Vagrant;
- Cluster Kubernetes utilizando K3s;
- Aplicação web containerizada em Flask;
- Banco de dados PostgreSQL integrado;
- Exposição externa da aplicação através de NodePort;
- Infraestrutura documentada e versionada em Git.

---

# Arquitetura

```text
Máquina Host (Windows)
        |
        |
 localhost:30050
        |
        |
 VM Ubuntu (Vagrant)
        |
        |
 Kubernetes K3s
        |
   +----+-------------+
   |                  |
   |                  |
Flask App          PostgreSQL
Container          Database
   |
   |
NodePort 30050
```

## Tecnologias utilizadas

### Infraestrutura
- Vagrant
- VirtualBox
- Ubuntu 22.04

### Kubernetes
- K3s
- Kubernetes YAML Manifests

### Aplicação
- Python 3.12
- Flask
- Gunicorn

### Banco de dados
- PostgreSQL 16

### Containerização
- Docker

### Controle de versão
- Git
- GitHub

---

# Passo a passo de execução

## 1. Pré-requisitos

Antes de iniciar, certifique-se de possuir instalado:

- VirtualBox
- Vagrant
- Git
- Docker

---

## 2. Clonar o repositório

```bash
git clone https://github.com/SousaDiego/devops-jr-case.git

cd devops-jr-case
```

---

## 3. Provisionar a máquina virtual

O ambiente é criado utilizando Vagrant.

Execute:

```bash
vagrant up
```

Esse processo realiza:

- Criação da VM Ubuntu 22.04;
- Configuração dos recursos da máquina;
- Instalação do Kubernetes K3s;
- Configuração do ambiente necessário para execução da aplicação.

Para acessar a VM:

```bash
vagrant ssh
```

---

## 4. Verificar o cluster Kubernetes

Dentro da VM, valide o funcionamento do cluster:

```bash
sudo kubectl get nodes
```

Resultado esperado:

```text
NAME        STATUS   ROLES
devops-jr   Ready    control-plane
```

---

## 5. Aplicar os manifests Kubernetes

Os arquivos Kubernetes estão localizados no diretório:

```text
kubernetes/
```

Aplicar todos os recursos:

```bash
sudo kubectl apply -f /vagrant/kubernetes/
```

São criados:

- Namespace da aplicação;
- Deployment da aplicação Flask;
- Service NodePort;
- Deployment PostgreSQL;
- Service PostgreSQL;
- PersistentVolumeClaim;
- Secret com credenciais do banco.

---

## 6. Validar os pods

Verifique se os containers estão executando:

```bash
sudo kubectl get pods -n devops
```

Resultado esperado:

```text
NAME                             READY   STATUS
devops-jr-app                    1/1     Running
postgres                         1/1     Running
```

---

## 7. Acessar a aplicação

A aplicação fica disponível através do NodePort:

```text
http://localhost:30050
```

Resposta esperada:

```json
{
  "application": "DevOps Jr Challenge",
  "status": "online",
  "version": "1.0.0"
}
```

---

# Estrutura do projeto

A organização dos arquivos foi definida da seguinte forma:

```text
devops-jr-case/
|
├── app/
│   ├── Dockerfile
│   ├── app.py
│   ├── database.py
│   └── requirements.txt
|
├── kubernetes/
│   ├── namespace.yaml
│   ├── app-deployment.yaml
│   ├── app-service.yaml
│   ├── postgres-deployment.yaml
│   ├── postgres-service.yaml
│   ├── postgres-pvc.yaml
│   └── postgres-secret.yaml
|
├── scripts/
|
├── Vagrantfile
|
└── README.md
```

---

# Decisões técnicas

## Vagrant + VirtualBox

Foi utilizado Vagrant para automatizar a criação e configuração da máquina virtual.

A escolha permite que todo o ambiente seja reproduzido facilmente em outra máquina, garantindo padronização do processo de instalação.

O VirtualBox foi utilizado como provedor de virtualização por ser uma solução gratuita e amplamente utilizada em ambientes locais.

---

## Kubernetes K3s

Foi escolhido o K3s como distribuição Kubernetes devido ao seu baixo consumo de recursos e facilidade de instalação em ambientes locais.

O cluster é responsável por gerenciar:

- Deploy da aplicação Flask;
- Execução dos containers;
- Serviços internos;
- Persistência do banco de dados;
- Comunicação entre os componentes.

---

## Docker

A aplicação Flask foi containerizada utilizando Docker.

O container contém:

- Código da aplicação;
- Dependências Python;
- Configuração do servidor Gunicorn.

A imagem utilizada pela aplicação é:

```text
devops-jr-app:1.4
```

---

## Flask + Gunicorn

A aplicação web foi desenvolvida utilizando Flask por ser um framework leve e adequado para uma API simples.

O Gunicorn foi utilizado como servidor WSGI para execução da aplicação em ambiente containerizado.

---

## PostgreSQL

O banco PostgreSQL 16 foi escolhido por ser um banco relacional robusto e amplamente utilizado no mercado.

A comunicação entre aplicação e banco ocorre através de variáveis de ambiente configuradas no Deployment Kubernetes.

---

## NodePort

A exposição externa da aplicação foi realizada utilizando um Service do tipo NodePort.

A aplicação fica disponível no host através da porta:

```text
localhost:30050
```

---

# Troubleshooting

Durante o desenvolvimento do ambiente foram encontrados alguns problemas:

## Problema 1 - kubectl não disponível na VM

### Sintoma

Ao executar:

```bash
kubectl get pods
```

Foi retornado:

```text
command not found
```

### Causa

A VM utilizada para execução do cluster Kubernetes ainda não possuía o K3s instalado e configurado.

### Solução

Foi realizada a instalação do K3s, que disponibiliza o kubectl através do próprio binário:

```bash
sudo kubectl get nodes
```

Após a configuração, o cluster passou a responder normalmente.

---

## Problema 2 - Imagem Docker não encontrada no Kubernetes

### Sintoma

O pod da aplicação apresentava o status:

```text
ErrImagePull
```

### Causa

A imagem configurada no Deployment:

```text
devops-jr-app:1.4
```

não estava disponível no ambiente local do container runtime utilizado pelo K3s.

### Solução

A imagem da aplicação foi construída localmente:

```bash
sudo docker build -t devops-jr-app:1.4 .
```

Após a criação da imagem e atualização do Deployment, a aplicação iniciou corretamente no cluster.

---

## Problema 3 - PostgreSQL em estado Pending

### Sintoma

O pod do PostgreSQL permanecia no estado:

```text
Pending
```

### Investigação

Foi utilizado:

```bash
sudo kubectl describe pod postgres -n devops
```

Foi identificado problema relacionado ao agendamento do PersistentVolume entre os nós disponíveis do cluster.

### Solução

O ambiente Kubernetes foi reorganizado e os manifests foram reaplicados, permitindo que o PostgreSQL fosse provisionado corretamente utilizando o PersistentVolumeClaim configurado.

---

# Validação final

Após a configuração do ambiente, foram realizadas as seguintes validações:

## Status do cluster

```bash
sudo kubectl get nodes
```

Resultado esperado:

```text
devops-jr   Ready   control-plane
```

---

## Status dos componentes

```bash
sudo kubectl get pods -n devops
```

Resultado esperado:

```text
devops-jr-app    1/1   Running
postgres         1/1   Running
```

---

## Teste da aplicação

A aplicação foi validada através do acesso externo utilizando NodePort:

```text
http://localhost:30050
```

Resposta retornada:

```json
{
  "application": "DevOps Jr Challenge",
  "status": "online",
  "version": "1.0.0"
}
```

---

# Considerações finais

O ambiente desenvolvido atende aos requisitos obrigatórios definidos no desafio:

- Provisionamento automatizado de máquina virtual utilizando Vagrant;
- Cluster Kubernetes funcional utilizando K3s;
- Aplicação web containerizada utilizando Docker;
- Integração da aplicação com banco PostgreSQL;
- Persistência de dados utilizando Kubernetes PersistentVolumeClaim;
- Exposição externa da aplicação através de Service NodePort;
- Código, manifests e documentação versionados em repositório Git público.