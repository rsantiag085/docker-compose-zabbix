# Docker-compose Zabbix com Grafana 

### Docker Compose para a Stack de Monitoramento Zabbix com Integração Grafana

Este arquivo `docker-compose.yml` configura uma stack completa de monitoramento com Zabbix integrada ao Grafana. Ele inclui os seguintes serviços:

---

## **Serviços**

### **1. MySQL**
- **Propósito**: Hospeda o banco de dados utilizado pelo servidor e pela interface do Zabbix.
- **Imagem**: `mysql:8.0`
- **Nome do Container**: `zabbix-mysql`
- **Variáveis de Ambiente**:
  - `MYSQL_ROOT_PASSWORD`: Senha para o usuário root do MySQL.
  - `MYSQL_DATABASE`: Nome do banco de dados do Zabbix.
  - `MYSQL_USER`: Usuário do banco de dados do Zabbix.
  - `MYSQL_PASSWORD`: Senha do usuário do banco de dados do Zabbix.
- **Comando**: Configura o MySQL para:
  - Permitir a criação de funções mesmo com o log binário ativado.
  - Utilizar codificação UTF-8 com collation `utf8_bin`.
- **Volumes**: Armazena os dados persistentes do MySQL no volume `zabbix_db`.
- **Rede**: Conectado à rede `zabbixnet`.

---

### **2. Servidor Zabbix**
- **Propósito**: Servidor backend responsável por gerenciar os agentes Zabbix, templates e coleta de dados.
- **Imagem**: `zabbix/zabbix-server-mysql:alpine-7.0.2`
- **Nome do Container**: `zabbix-server`
- **Variáveis de Ambiente**:
  - `DB_SERVER_HOST`: Host do servidor MySQL.
  - `MYSQL_DATABASE`: Nome do banco de dados do Zabbix.
  - `MYSQL_USER`: Usuário do banco de dados do Zabbix.
  - `MYSQL_PASSWORD`: Senha do usuário do banco de dados do Zabbix.
- **Dependências**: Aguarda a inicialização do container MySQL.
- **Rede**: Conectado à rede `zabbixnet`.

---

### **3. Frontend do Zabbix**
- **Propósito**: Interface web para gerenciar e visualizar os dados do Zabbix.
- **Imagem**: `zabbix/zabbix-web-apache-mysql:alpine-7.0.2`
- **Nome do Container**: `zabbix-frontend`
- **Variáveis de Ambiente**:
  - `DB_SERVER_HOST`: Host do servidor MySQL.
  - `MYSQL_DATABASE`: Nome do banco de dados do Zabbix.
  - `MYSQL_USER`: Usuário do banco de dados do Zabbix.
  - `MYSQL_PASSWORD`: Senha do usuário do banco de dados do Zabbix.
  - `ZBX_SERVER_HOST`: Host do servidor Zabbix.
- **Portas**:
  - Expõe a interface web do Zabbix na porta `8080`.
- **Dependências**: Aguarda a inicialização do container do servidor Zabbix.
- **Rede**: Conectado à rede `zabbixnet`.

---

### **4. Agente Zabbix**
- **Propósito**: Coleta dados de monitoramento do host e envia ao servidor Zabbix.
- **Imagem**: `zabbix/zabbix-agent:alpine-7.0.2`
- **Nome do Container**: `zabbix-agent`
- **Portas**:
  - Expõe a comunicação do agente na porta `10050`.
- **Dependências**: Aguarda a inicialização do container do servidor Zabbix.
- **Rede**: Conectado à rede `zabbixnet` com um endereço IP fixo (`172.18.0.6`).

---

### **5. Grafana**
- **Propósito**: Ferramenta de visualização para exibir dados do Zabbix em dashboards personalizáveis.
- **Imagem**: `grafana/grafana:latest`
- **Nome do Container**: `grafana`
- **Portas**:
  - Expõe a interface web do Grafana na porta `3000`.
- **Volumes**: Armazena dados persistentes do Grafana no volume `grafana_db`.
- **Rede**: Conectado à rede `zabbixnet`.

---

## **Volumes**
- **zabbix_db**: Persiste os dados do banco de dados do MySQL.
- **grafana_db**: Persiste as configurações e dashboards do Grafana.

---

## **Redes**
- **zabbixnet**:
  - Tipo: Rede do tipo bridge.
  - Faixa de IPs: Configurada com a sub-rede `172.18.0.0/24` para melhor isolamento e previsibilidade de endereços.

---

## **Instruções de Uso**

1. **Configuração e Inicialização dos Serviços**:
   Execute o seguinte comando para iniciar a stack:
   ```bash
   docker-compose up -d
   ```

2. **Acessando os Serviços**:
   - **Frontend do Zabbix**: Abra o navegador e acesse `http://<ip-do-servidor>:8080`.
     - Credenciais padrão:
       - Usuário: `Admin`
       - Senha: `zabbix`
   - **Grafana**: Abra o navegador e acesse `http://<ip-do-servidor>:3000`.
     - Credenciais padrão:
       - Usuário: `admin`
       - Senha: `admin` (você será solicitado a alterá-la no primeiro login).

3. **Agentes de Monitoramento**:
   - Adicione novos hosts na interface web do Zabbix e configure seus endereços IP para apontar para as máquinas ou serviços monitorados.

4. **Integração Zabbix-Grafana**:
   - Instale o plugin de datasource do Zabbix no Grafana.
   - Configure a API do Zabbix no Grafana usando as credenciais do frontend do Zabbix.

---

## **Personalização**
- **Variáveis de Ambiente**: Ajuste senhas e nomes de banco de dados conforme necessário na seção `environment`.
- **Portas**: Personalize as portas para evitar conflitos com serviços existentes.
- **Volumes**: Armazene dados em diretórios personalizados mapeando caminhos do host para os volumes dos containers.

---

### **Resolução de Problemas**

- **Erros no MySQL**: Certifique-se de que a opção `log_bin_trust_function_creators=1` está habilitada se estiver utilizando funções armazenadas.
- **Problemas no Frontend**: Verifique se todos os containers estão em execução e podem se comunicar pela rede `zabbixnet`.
- **Problemas no Grafana**: Certifique-se de que o endpoint da API do Zabbix está acessível e configurado corretamente no Grafana.

---

Este arquivo `docker-compose.yml` fornece uma stack Zabbix e Grafana pronta para uso, permitindo monitoramento robusto e visualização avançada. 🚀
