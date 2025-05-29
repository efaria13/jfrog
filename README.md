

# Guia de Demonstração JFrog Platform - Principais Funcionalidades

## Introdução
A JFrog Platform é uma solução completa que unifica DevOps, DevSecOps e MLOps em uma única plataforma, oferecendo controle end-to-end sobre o ciclo de vida de software desde o desenvolvimento até a produção.

## Pré-requisitos
- Conta JFrog (teste gratuito disponível em https://jfrog.com/start-free/)
- JFrog CLI instalado
- Docker instalado (para demonstrações de containers)
- Um projeto de exemplo (Java, Node.js, Python, etc.)

---

## **1. JFrog Artifactory - Gerenciamento Universal de Artefatos**

### 1.1 Configuração Inicial
**Passo 1: Acesso ao Artifactory**
- Acesse sua instância JFrog Platform
- Navegue para `Administration > Repositories`
- Observe os repositórios pré-configurados

**Passo 2: Criar Repositório Local**
- Clique em `New Repository > Local`
- Selecione o tipo de package (Maven, npm, Docker, etc.)
- Configure nome: `demo-maven-local`
- Salve a configuração

**Passo 3: Criar Repositório Remoto**
- Clique em `New Repository > Remote`
- Para Maven: URL = https://repo1.maven.org/maven2/
- Nome: `maven-central-remote`
- Configure cache e políticas de retenção

**Passo 4: Criar Repositório Virtual**
- Clique em `New Repository > Virtual`
- Nome: `maven-virtual`
- Adicione repositórios locais e remotos
- Configure ordem de resolução

### 1.2 Upload e Download de Artefatos
**Demonstração prática:**
```bash
# Configurar JFrog CLI
jfrog config add --url=https://yourinstance.jfrog.io --user=username --password=password

# Upload de arquivo
jfrog rt upload "target/*.jar" demo-maven-local/com/example/app/1.0.0/

# Download de arquivo
jfrog rt download demo-maven-local/com/example/app/1.0.0/*.jar ./downloads/

# Upload com propriedades customizadas
jfrog rt upload "target/*.jar" demo-maven-local/ --props="version=1.0.0;stage=dev"
```

### 1.3 Build Info e Rastreabilidade
**Passo 1: Configurar Build Info**
```bash
# Iniciar coleta de build info
jfrog rt build-collect-env myapp 1.0.0

# Executar build (Maven exemplo)
jfrog rt mvn clean install --build-name=myapp --build-number=1.0.0

# Publicar build info
jfrog rt build-publish myapp 1.0.0
```

**Passo 2: Visualizar no Artifactory**
- Navegue para `Application > Builds`
- Encontre build `myapp 1.0.0`
- Explore dependências, artefatos produzidos e metadados

---

## **2. JFrog Xray - Segurança e Análise**

### 2.1 Configuração de Políticas de Segurança
**Passo 1: Criar Política de Segurança**
- Acesse `Administration > Xray > Policies`
- Clique `New Policy`
- Nome: `Critical-Vulnerabilities-Policy`
- Configure regras:
  - Severidade: Critical
  - CVE Score: ≥ 9.0
  - Ação: Fail Build

**Passo 2: Criar Watch**
- Vá para `Administration > Xray > Watches`
- Nome: `Production-Watch`
- Adicione repositórios de produção
- Associe a política criada
- Configure notificações por email

### 2.2 Scan de Vulnerabilidades
**Demonstração de Scan Manual:**
```bash
# Scan de repositório específico
jfrog xr scan --repo=demo-maven-local

# Scan de build específico
jfrog xr build-scan myapp 1.0.0

# Scan de artefato específico
jfrog xr scan demo-maven-local/com/example/app/1.0.0/app-1.0.0.jar
```

**Passo 3: Análise de Resultados**
- Navegue para `Application > Security & Compliance`
- Visualize violations encontradas
- Explore impact analysis graph
- Analise componentes afetados

### 2.3 Container Scanning
**Demonstração com Docker:**
```bash
# Build de imagem Docker
docker build -t myapp:1.0.0 .

# Tag para Artifactory
docker tag myapp:1.0.0 yourinstance.jfrog.io/docker-local/myapp:1.0.0

# Push para Artifactory
docker push yourinstance.jfrog.io/docker-local/myapp:1.0.0

# Scan automático será executado pelo Xray
```

---

## **3. JFrog Pipelines - CI/CD Nativo**

### 3.1 Criação de Pipeline Básico
**Passo 1: Configurar Pipeline YAML**
```yaml
# pipelines.yml
pipelines:
  - name: demo_pipeline
    steps:
      - name: build_step
        type: Bash
        configuration:
          inputResources:
            - name: myapp_git
        execution:
          onExecute:
            - echo "Building application"
            - mvn clean install
            - jfrog rt upload target/*.jar demo-maven-local/
      
      - name: scan_step
        type: Bash
        configuration:
          inputSteps:
            - name: build_step
        execution:
          onExecute:
            - jfrog xr build-scan $pipeline_name $run_number
      
      - name: deploy_step
        type: Bash
        configuration:
          inputSteps:
            - name: scan_step
        execution:
          onExecute:
            - echo "Deploying to staging"
            - kubectl apply -f deployment.yaml
```

**Passo 2: Configurar Recursos**
- Adicione Git resource apontando para seu repositório
- Configure integrations necessárias (Docker, Kubernetes)
- Execute pipeline e monitore logs

### 3.2 Release Bundles
**Demonstração de Release Bundle:**
```bash
# Criar Release Bundle
jfrog ds create-bundle myapp-bundle 1.0.0 --spec=bundle-spec.json

# Assinar Release Bundle
jfrog ds sign-bundle myapp-bundle 1.0.0

# Distribuir para edge nodes
jfrog ds distribute-bundle myapp-bundle 1.0.0 --site="edge-site-1"
```

---

## **4. JFrog ML - MLOps (Novo em 2025)**

### 4.1 Configuração de Ambiente ML
JFrog ML é uma solução revolucionária de MLOps que permite aos times de desenvolvimento, cientistas de dados e engenheiros ML desenvolver e deployar rapidamente aplicações empresariais.

**Passo 1: Setup de Projeto ML**
- Acesse `Application > ML`
- Crie novo projeto: `demo-ml-project`
- Configure repositórios Python e HuggingFace
- Setup de token de acesso

**Passo 2: Model Training**
```python
# exemplo_training.py
import jfrog_ml as jml

# Inicializar experimento
experiment = jml.create_experiment("image-classification")

# Log de parâmetros
experiment.log_param("learning_rate", 0.001)
experiment.log_param("batch_size", 32)

# Training loop
model = train_model(data, params)

# Log de métricas
experiment.log_metric("accuracy", 0.95)
experiment.log_metric("loss", 0.05)

# Salvar modelo
experiment.log_model(model, "pytorch-classifier")
```

### 4.2 Model Serving e Monitoring
**Passo 3: Deploy de Modelo**
- Configure model serving endpoint
- Setup de monitoring e drift detection
- Configure alertas de performance

---

## **5. JFrog CLI - Automação Avançada**

### 5.1 Configurações Avançadas
```bash
# Configurar múltiplas instâncias
jfrog config add prod --url=https://prod.jfrog.io --user=prod-user
jfrog config add staging --url=https://staging.jfrog.io --user=staging-user

# Usar configuração específica
jfrog rt upload --server-id=prod target/*.jar prod-repo/

# Upload paralelo (performance)
jfrog rt upload "target/*.jar" prod-repo/ --threads=8

# Sync entre repositórios
jfrog rt copy prod-repo/path/ staging-repo/path/ --flat=true
```

### 5.2 Automação com Scripts
```bash
#!/bin/bash
# deploy-script.sh

# Build info collection
jfrog rt build-collect-env $BUILD_NAME $BUILD_NUMBER

# Maven build with Artifactory integration
jfrog rt mvn clean install --build-name=$BUILD_NAME --build-number=$BUILD_NUMBER

# Security scan
jfrog xr build-scan $BUILD_NAME $BUILD_NUMBER

# Conditional deployment based on scan results
if [ $? -eq 0 ]; then
    echo "Security scan passed, deploying..."
    jfrog rt build-promote $BUILD_NAME $BUILD_NUMBER staging-repo
else
    echo "Security scan failed, blocking deployment"
    exit 1
fi
```

---

## **6. Integrações e Automação**

### 6.1 Integração com GitHub Actions
```yaml
# .github/workflows/ci.yml
name: CI/CD with JFrog
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup JFrog CLI
        uses: jfrog/setup-jfrog-cli@v2
        env:
          JF_URL: ${{ secrets.JF_URL }}
          JF_USER: ${{ secrets.JF_USER }}
          JF_PASSWORD: ${{ secrets.JF_PASSWORD }}
      
      - name: Build and Upload
        run: |
          jfrog rt build-collect-env $GITHUB_REPOSITORY $GITHUB_RUN_NUMBER
          mvn clean install
          jfrog rt upload target/*.jar libs-release-local/
          jfrog rt build-publish $GITHUB_REPOSITORY $GITHUB_RUN_NUMBER
      
      - name: Security Scan
        run: jfrog xr build-scan $GITHUB_REPOSITORY $GITHUB_RUN_NUMBER
```

### 6.2 Integração com Jenkins
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    def server = Artifactory.server 'jfrog-instance'
                    def buildInfo = Artifactory.newBuildInfo()
                    
                    sh 'mvn clean install'
                    
                    def uploadSpec = """{
                        "files": [{
                            "pattern": "target/*.jar",
                            "target": "libs-release-local/"
                        }]
                    }"""
                    
                    server.upload(uploadSpec, buildInfo)
                    server.publishBuildInfo(buildInfo)
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    def xrayConfig = [
                        'buildName': env.JOB_NAME,
                        'buildNumber': env.BUILD_NUMBER
                    ]
                    xrayScan xrayConfig
                }
            }
        }
    }
}
```

---

## **7. Monitoramento e Observabilidade**

### 7.1 Configuração de Dashboards
**Métricas Principais:**
- Download/Upload rates
- Storage consumption
- Security violations
- Build success/failure rates
- Pipeline performance

**Passo 1: Configurar Logs e Métricas**
- Acesse `Administration > General > Logs`
- Configure log levels
- Setup de exportação para Prometheus/Grafana

### 7.2 Alertas e Notificações
```bash
# Configurar webhook para Slack
curl -X POST https://yourinstance.jfrog.io/xray/api/v1/webhooks \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://hooks.slack.com/services/your/webhook/url",
    "event_types": ["violation_alert"],
    "criteria": {
      "min_severity": "High"
    }
  }'
```

---

## **8. Casos de Uso Avançados**

### 8.1 Multi-Cloud Deployment
- Configuração de replicação entre regiões
- Sincronização de artefatos
- Disaster recovery procedures

### 8.2 Enterprise Governance
- RBAC (Role-Based Access Control)
- Audit trails
- Compliance reporting
- License management

### 8.3 Edge Computing
Gerenciamento de dispositivos IoT e Edge com atualizações OTA completas como parte da JFrog Platform

---

## Próximos Passos

1. **Teste Gratuito**: Inicie com a versão gratuita da JFrog Platform
2. **Proof of Concept**: Implemente um pipeline completo com seu projeto
3. **Treinamento**: Explore a documentação oficial em docs.jfrog.com
4. **Community**: Participe do JFrog User Community para best practices

## Recursos Adicionais

- [Documentação Oficial](https://jfrog.com/help/)
- [JFrog University](https://jfrog.com/university/)
- [Community Forums](https://jfrog.com/community/)
- [Blog Técnico](https://jfrog.com/blog/)

---

**Nota**: Este guia cobre as funcionalidades principais da JFrog Platform em 2025. A plataforma continua evoluindo com novos recursos como JFrog ML para MLOps e integrações aprimoradas.
