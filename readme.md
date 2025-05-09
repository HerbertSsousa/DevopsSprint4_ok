

#  OdontoPrev Solution - Aplicação Java na Nuvem com CI/CD

Este repositório contém uma solução Java desenvolvida com Spring Boot e implantada automaticamente em um WebApp do Microsoft Azure por meio de uma pipeline CI/CD no Azure DevOps. A aplicação realiza operações básicas de CRUD conectando-se a um banco de dados em nuvem, com foco em automação, agilidade e confiabilidade.

---

##  Descrição da Solução

A aplicação Java foi construída utilizando o framework Spring Boot, com funcionalidades de CRUD integradas a um banco de dados SQL hospedado na nuvem (Azure SQL Database). O processo de build, testes e deploy foi totalmente automatizado através de uma pipeline CI/CD no Azure DevOps.

### Principais Funcionalidades

- Desenvolvimento com Java 17 + Spring Boot
- Conexão com banco de dados em nuvem (Azure SQL Database)
- Deploy automatizado no Azure App Service (WebApp Linux)
- CI/CD com Azure DevOps Pipelines (YAML)
- Operações de CRUD totalmente funcionais via API

---

##  Pipeline CI/CD - Visão Geral

O fluxo de CI/CD foi dividido em 3 estágios:

1. **CriarInfra**: Criação do grupo de recursos, App Service Plan e WebApp.
2. **BuildApp**: Compilação do projeto com Maven, execução de testes e publicação do artefato.
3. **DeployApp**: Realização do deploy automático do artefato no WebApp.



## Desenho da Pipeline e Detalhamento das Etapas

O fluxo de Integração Contínua e Entrega Contínua (CI/CD) foi planejado para garantir agilidade, confiabilidade e automação no deploy da aplicação Java em ambiente de nuvem usando serviços da Microsoft Azure.  

### Etapas Detalhadas

1. **Criação da API em Java e publicação no GitHub**  
Desenvolvemos uma API funcional usando Java com Spring Boot para operações CRUD. Após testes locais, publicamos o código no GitHub, garantindo versionamento e integração com o Azure DevOps.

2. **Criação da Pipeline CI/CD no Azure DevOps**  
Configuramos uma pipeline no Azure DevOps para integração contínua (CI) e entrega contínua (CD), garantindo build, testes e deploy automatizados.

3. **Clonagem do repositório na pipeline**  
A pipeline clona automaticamente o repositório do GitHub, sempre utilizando a versão mais recente do código.

4. **Criação do banco de dados em nuvem (Azure SQL)**  
Provisionamos o banco de dados no Azure SQL Database para armazenar os dados da aplicação com segurança e acessos controlados.

5. **Conexão do banco de dados à aplicação na pipeline**  
As credenciais e URL do banco foram configuradas como variáveis de ambiente seguras na pipeline, permitindo a conexão da aplicação.

6. **Execução da pipeline**  
O build da aplicação é executado, seguido pelos testes e deploy automático no WebApp do Azure.

7. **Validação do deploy e do ambiente em produção**  
Após o deploy, validamos o funcionamento do WebApp, a resposta da API e a conexão com o banco de dados.

8. **Demonstração do CRUD na aplicação em nuvem**  
Executamos operações de inserção, consulta, atualização e exclusão de dados via API para comprovar o funcionamento do sistema completo.

<img src="/img/desenho.png">


###  Etapas Detalhadas

1. **Criação da API Java com Spring Boot**
2. **Publicação do código no GitHub**
3. **Criação e configuração da pipeline CI/CD no Azure DevOps**
4. **Clonagem automática do repositório**
5. **Criação e configuração do Azure SQL Database**
6. **Configuração segura da conexão com o banco (via variáveis de ambiente)**
7. **Execução e validação do deploy**
8. **Demonstração funcional da aplicação na nuvem**

---

##  Tecnologias Utilizadas

- Java 17
- Spring Boot
- Maven
- Azure DevOps (Pipelines)
- Azure App Service (Linux)
- Azure SQL Database
- Git e GitHub
- YAML para configuração da pipeline

---

##  Estrutura da Pipeline (YAML)

```yaml
trigger:
  branches:
    include:
    - main
    - master
    - minharelease

pool:
  vmImage: "ubuntu-latest"

variables:
- name: rm
  value: 553227
- name: location
  value: eastus
- name: resourceGroup
  value: rg-OdontoPrev-Sprint
- name: service-plan
  value: odontoprevSprint
- name: app-name
  value: Odonto-PrevSolution-$(rm)
- name: runtime
  value: JAVA:17-java17
- name: sku
  value: F1
- name: nome-artefato
  value: OdontoSolution

stages:
- stage: CriarInfra
  jobs:
  - job: criaWebApp
    displayName: Criar ou atualizar o Serviço de Aplicativo
    steps:
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'MyAzureSubscription'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az group create --location $(location) --name $(resourceGroup)
          az appservice plan create -g $(resourceGroup) -n $(service-plan) --is-linux --sku $(sku)
          az webapp create -g $(resourceGroup) -p $(service-plan) -n $(app-name) --runtime "$(runtime)"
        visibleAzLogin: false

- stage: BuildApp
  variables:
  - name: mavenPOMFile
    value: 'pom.xml'
  jobs:
  - job: buildWebApp
    displayName: Realizar o Build do App
    steps:
    - task: Maven@4
      displayName: 'Build do Projeto'
      inputs:
        mavenPomFile: '$(mavenPOMFile)'
        testRunTitle: 'Testes Unitários'
        jdkVersionOption: 1.17
    - task: CopyFiles@2
      displayName: 'Copiar artefato'
      inputs:
        SourceFolder: '$(system.defaultworkingdirectory)'
        Contents: 'target/*.jar'
        TargetFolder: '$(build.artifactstagingdirectory)'
    - task: PublishBuildArtifacts@1
      displayName: 'Publicar artefato'
      inputs:
        PathtoPublish: '$(build.artifactstagingdirectory)'
        ArtifactName: $(nome-artefato)

- stage: DeployApp
  jobs:
  - job: DeployWebApp
    displayName: Deploy no Azure
    steps:
    - task: DownloadBuildArtifacts@1
      displayName: 'Baixar artefato gerado'
      inputs:
        buildType: 'current'
        downloadType: 'specific'
        downloadPath: '$(System.DefaultWorkingDirectory)'
    - task: AzureRmWebAppDeployment@4
      displayName: 'Deploy da Aplicação'
      inputs:
        azureSubscription: 'MyAzureSubscription'
        appType: 'webApp'
        WebAppName: $(app-name)
        packageForLinux: '$(System.DefaultWorkingDirectory)/$(nome-artefato)/target/*.jar'



 <img src="/desenho.png">





## Link do vídeo

[YouTube]https://youtu.be/Q4zXpiB7lR0



## Link do Github

