# DESAFIO DE PROJETO  
**Integrando Dados com MySQL Azure e Transformando com Power BI**

---

## Ementa - Etapas do Desafio

1. Descrevendo o desafio de projeto  
2. Criando uma instância do MySQL na Azure  
3. Explorando o Recurso - Instância do MySQL  
4. Conectando ao Banco de Dados via Cloud Shell  
5. Criando Regra no Firewall na Azure para Acesso ao banco de dados  
6. Conectando ao MySQL na Azure utilizando Workbench  
7. Integrando Power BI com MySQL na Azure  
8. Transformações de dados via Power Query  


---

## Descrição do desafio de projeto

O objetivo deste desafio foi:

1. Criar uma instância de MySQL no Azure para hospedar os dados do desafio.  
2. Criar e popular o banco de dados `company` com os dados fornecidos no GitHub.  
3. Integrar o banco MySQL do Azure com o Power BI Desktop.  
4. Realizar **limpeza e transformação de dados** para permitir análise e criação de dashboards simples.  

---

## Tecnologias Utilizadas

- **Azure MySQL Flexible Server** – instância criada via Azure CLI/PowerShell.  
- **MySQL Workbench** – criação de tabelas e inserção de dados.  
- **Power BI Desktop** – integração e transformação via Power Query.  
- **SQL** – manipulação de dados.  
- **CLI / Bash / PowerShell** – gerenciamento da instância Azure.  

---
**OBS: este projeto é resposta de um desafio proposto pelo Bootcamp de Analise de dados, parceria entre a plataforma Dio e Klabin.**

O repositório com o desafio original pode ser encontrado em: [Desafio Power BI Cursp - Juliana Mascarenhas](https://github.com/julianazanelatto/power_bi_analyst/tree/main/M%C3%B3dulo%203/Desafio%20de%20Projeto)


---

## Passo a Passo

### 1. Criando a instância MySQL no Azure

```bash
az mysql flexible-server create \
  --location northeurope \
  --resource-group larissammysql \
  --name larissamsqldesafiodio \
  --admin-user  \
  --admin-password  \
  --sku-name Standard_B1ms \
  --storage-size 32 \
  --version 8.0.21
```


**Configurado firewall para permitir acesso local e pelo Power BI:**

```bash
Copy code
az mysql flexible-server firewall-rule create \
  --name larissamsqldesafiodio \
  --resource-group larissammysql \
  --rule-name AllowMyIP \
  --start-ip-address <SEU_IP> \
  --end-ip-address <SEU_IP>
```

### 2. Criando o banco e tabelas no MySQL Workbench
sql

```
-- Criar banco de dados
CREATE DATABASE IF NOT EXISTS company;
USE company;

-- Tabela employee
CREATE TABLE employee (
    Fname VARCHAR(50),
    Minit CHAR(1),
    Lname VARCHAR(50),
    Ssn BIGINT PRIMARY KEY,
    Bdate DATE,
    Address VARCHAR(100),
    Sex CHAR(1),
    Salary DOUBLE,
    Super_ssn BIGINT,
    Dno INT
);

-- Tabela dependent
CREATE TABLE dependent (
    Essn BIGINT,
    Dependent_name VARCHAR(50),
    Sex CHAR(1),
    Bdate DATE,
    Relationship VARCHAR(20)
);

-- Tabela departament
CREATE TABLE departament (
    Dname VARCHAR(50),
    Dnumber INT PRIMARY KEY,
    Mgr_ssn BIGINT,
    Mgr_start_date DATE,
    Mgr_end_date DATE
);

-- Tabela dept_locations
CREATE TABLE dept_locations (
    Dnumber INT,
    Dlocation VARCHAR(50)
);

-- Tabela project
CREATE TABLE project (
    Pname VARCHAR(50),
    Pnumber INT PRIMARY KEY,
    Plocation VARCHAR(50),
    Dnum INT
);

-- Tabela works_on
CREATE TABLE works_on (
    Essn BIGINT,
    Pno INT,
    Hours DOUBLE
);
```


Ingestão de dados:

```
use company_constraints;

insert into employee values ('John', 'B', 'Smith', 123456789, '1965-01-09', '731-Fondren-Houston-TX', 'M', 30000, 333445555, 5),
							('Franklin', 'T', 'Wong', 333445555, '1955-12-08', '638-Voss-Houston-TX', 'M', 40000, 888665555, 5),
                            ('Alicia', 'J', 'Zelaya', 999887777, '1968-01-19', '3321-Castle-Spring-TX', 'F', 25000, 987654321, 4),
                            ('Jennifer', 'S', 'Wallace', 987654321, '1941-06-20', '291-Berry-Bellaire-TX', 'F', 43000, 888665555, 4),
                            ('Ramesh', 'K', 'Narayan', 666884444, '1962-09-15', '975-Fire-Oak-Humble-TX', 'M', 38000, 333445555, 5),
                            ('Joyce', 'A', 'English', 453453453, '1972-07-31', '5631-Rice-Houston-TX', 'F', 25000, 333445555, 5),
                            ('Ahmad', 'V', 'Jabbar', 987987987, '1969-03-29', '980-Dallas-Houston-TX', 'M', 25000, 987654321, 4),
                            ('James', 'E', 'Borg', 888665555, '1937-11-10', '450-Stone-Houston-TX', 'M', 55000, NULL, 1);

insert into dependent values (333445555, 'Alice', 'F', '1986-04-05', 'Daughter'),
							 (333445555, 'Theodore', 'M', '1983-10-25', 'Son'),
                             (333445555, 'Joy', 'F', '1958-05-03', 'Spouse'),
                             (987654321, 'Abner', 'M', '1942-02-28', 'Spouse'),
                             (123456789, 'Michael', 'M', '1988-01-04', 'Son'),
                             (123456789, 'Alice', 'F', '1988-12-30', 'Daughter'),
                             (123456789, 'Elizabeth', 'F', '1967-05-05', 'Spouse');

insert into departament values ('Research', 5, 333445555, '1988-05-22','1986-05-22'),
							   ('Administration', 4, 987654321, '1995-01-01','1994-01-01'),
                               ('Headquarters', 1, 888665555,'1981-06-19','1980-06-19');

insert into dept_locations values (1, 'Houston'),
								 (4, 'Stafford'),
                                 (5, 'Bellaire'),
                                 (5, 'Sugarland'),
                                 (5, 'Houston');

insert into project values ('ProductX', 1, 'Bellaire', 5),
						   ('ProductY', 2, 'Sugarland', 5),
						   ('ProductZ', 3, 'Houston', 5),
                           ('Computerization', 10, 'Stafford', 4),
                           ('Reorganization', 20, 'Houston', 1),
                           ('Newbenefits', 30, 'Stafford', 4)
;

insert into works_on values (123456789, 1, 32.5),
							(123456789, 2, 7.5),
                            (666884444, 3, 40.0),
                            (453453453, 1, 20.0),
                            (453453453, 2, 20.0),
                            (333445555, 2, 10.0),
                            (333445555, 3, 10.0),
                            (333445555, 10, 10.0),
                            (333445555, 20, 10.0),
                            (999887777, 30, 30.0),
                            (999887777, 10, 10.0),
                            (987987987, 10, 35.0),
                            (987987987, 30, 5.0),
                            (987654321, 30, 20.0),
                            (987654321, 20, 15.0),
                            (888665555, 20, 0.0);
```

**OBS: Scripts SQL fornecido para o desafio**

### 3. Conectando Power BI ao MySQL Azure

No Power BI Desktop:


1. Obter Dados → Banco de Dados → MySQL Database
2. Servidor: larissamsqldesafiodio.mysql.database.azure.com
3. Banco: company
4. Usuário: serv@larissamsqldesafiodio
5. Senha: 
6. Habilite SSL obrigatório se solicitado
7. Conectar e carregar as tabelas
8. Transformações de dados via Power Query
Todas as transformações foram realizadas no Power Query, seguindo as diretrizes:

Verificação de cabecalhos e tipos de dados

Conversão de valores monetários para tipo decimal/double

Tratamento de valores nulos em colaboradores e departamentos

Mescla das tabelas employee e departament para associar os nomes dos departamentos

Criação de coluna única para Nome Completo (Nome + Sobrenome)

Mescla de departamentos e localização para criar combinações únicas

Agrupamento de dados para contar colaboradores por gerente

Remoção de colunas desnecessárias

Observação: todas as mesclas e transformações foram feitas via Power Query, sem necessidade de SQL adicional.

Estrutura do Banco de Dados
employee – dados dos funcionários

dependent – dependentes de cada funcionário

departament – departamentos da empresa

dept_locations – localização de cada departamento

project – projetos e departamentos associados

works_on – relação funcionário-projeto com horas trabalhadas

Resultado
Banco company hospedado no Azure MySQL com todas as tabelas e dados

Conexão funcional com Power BI

Transformações aplicadas via Power Query para análise de inconsistências

Dashboards simples gerados para verificação de padrões e possíveis problemas nos dados

## Observações Finais:

Este projeto é uma prova de conceito para integração de dados entre Azure MySQL e Power BI

O foco foi limpeza, transformação e análise básica de dados
