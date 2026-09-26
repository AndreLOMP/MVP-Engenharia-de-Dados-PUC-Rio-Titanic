MVP Engenharia de Dados - Análise de Sobrevivência do Titanic
Introdução
Neste MVP são explorados os dados do histórico de passageiros do fatídico desastre do RMS Titanic
. O projeto foi construído utilizando a plataforma Databricks e a linguagem PySpark, aplicando os conceitos de Engenharia de Dados modernizados pela Arquitetura Medalhão (camadas Bronze, Silver e Gold) e modelagem dimensional em Star Schema (Esquema Estrela).
A análise desses dados busca entender os fatores socioeconômicos e demográficos que influenciaram diretamente nas chances de sobrevivência dos passageiros durante o naufrágio.
Objetivo
O objetivo deste trabalho é processar, limpar, modelar e analisar o conjunto de dados do Titanic
, transformando dados brutos em visões analíticas de negócio prontas para consumo.
Para responder às dúvidas centrais sobre o desastre, foram estabelecidas 5 hipóteses/perguntas de negócio:
Os passageiros de primeira classe sobreviveram mais do que os passageiros de outras classes?
A porcentagem de sobrevivência das mulheres foi maior do que a dos homens?
A idade foi um fator determinante na sobrevivência?
A presença de familiares a bordo (irmãos/cônjuges e pais/filhos) facilitou ou dificultou o processo de sobrevivência?
A tarifa (Fare) paga pelo passageiro influenciou diretamente na sua sobrevivência?
Detalhamento
Busca e Coleta de Dados
A base primária utilizada foi o arquivo Dataset_Titanic.csv
, contendo 891 registros de passageiros e 12 colunas originais (PassengerId, Survived, Pclass, Name, Sex, Age, SibSp, Parch, Ticket, Fare, Cabin e Embarked)
.
O arquivo foi ingerido no ambiente do Databricks via PySpark a partir de repositório remoto, garantindo persistência e processamento distribuído nativo.
Modelagem (Arquitetura Medalhão & Star Schema)
A modelagem seguiu rigorosamente a Arquitetura Medalhão:
Camada Bronze (Raw): Ingestão do CSV bruto para a tabela Delta bronze_titanic, preservando o esquema original e adicionando a coluna de auditoria _ingestion_time.
Camada Silver (Cleaned & Enriched): Tabela Delta silver_titanic com tratamento de nulos (Age, Cabin, Embarked) e criação de novas variáveis (feature engineering): Has_Cabin, Title, Family_Size e Is_Alone.
Camada Gold (Curated & Dimensional): Estruturação do modelo dimensional em Star Schema para consumo em BI:
Tabela Fato: gold_fact_passenger_survival (contém métricas e chaves estrangeiras).
Tabelas Dimensão: gold_dim_passenger, gold_dim_pclass e gold_dim_embarked.
Tabelas Agregadas de Negócio: gold_survival_by_class, gold_survival_by_gender, gold_survival_by_age, gold_survival_by_family e gold_survival_by_fare.
Dicionário de Metadados (Tabela Silver)
Coluna
Tipo
Descrição / Transformação
PassengerId
Integer
Identificador único do passageiro (Chave Primária)
Survived
Integer
Indicador de sobrevivência (0 = Não, 1 = Sim)
Pclass
Integer
Classe do bilhete (1 = 1ª, 2 = 2ª, 3 = 3ª Classe)
Name
String
Nome completo do passageiro
Sex
String
Gênero do passageiro (male / female)
Age
Double
Idade em anos (Imputada mediana nos nulos)
SibSp
Integer
Número de irmãos/cônjuges a bordo
Parch
Integer
Número de pais/filhos a bordo
Ticket
String
Número do bilhete
Fare
Double
Tarifa paga pela passagem
Cabin
String
Número da cabine (Preenchido "Unknown" nos nulos)
Embarked
String
Porto de embarque (S = Southampton, C = Cherbourg, Q = Queenstown)
Has_Cabin
Integer
Flag binária indicando se possuía cabine registrada (1 ou 0)
Title
String
Título social extraído do nome (Mr, Mrs, Miss, Master, etc.)
Family_Size
Integer
Tamanho total da família a bordo (SibSp + Parch + 1)
Is_Alone
Integer
Flag binária indicando se viajava desacompanhado (1 se Family_Size == 1)
Carga e Pipeline
A subida e o processamento dos dados na nuvem foram realizados inteiramente dentro do Databricks, utilizando PySpark e o formato de armazenamento Delta Lake para garantir transações ACID.
Notebooks e Scripts do Projeto:
Ingestão Bronze: Converte o CSV baixado em PySpark DataFrame, adiciona metadados de auditoria e grava a tabela bronze_titanic.
Transformação Silver: Trata valores ausentes em Age (imputação por mediana), Cabin e Embarked, extrai recursos (Title, Family_Size, Is_Alone) e grava a tabela silver_titanic.
Modelagem Gold: Constrói o modelo dimensional Star Schema (fact + dims) e gera as visões agregadas para responder às 5 perguntas de negócio com o comando display().
Análise
Qualidade dos Dados e Tratamentos
Age: Apresentava 177 valores nulos no dataset original (ex: passageiros 6, 18, 20)
. Foram preenchidos utilizando a mediana das idades para não distorcer a distribuição.
Cabin: Apresentava elevada taxa de valores ausentes
. Os nulos foram substituídos por "Unknown" e criou-se a flag Has_Cabin para indicar passageiros com cabine declarada.
Embarked: Valores ausentes foram preenchidos com o porto mais frequente ('S' - Southampton).
Name: Foi extraído o título social (Title) via expressão regular (r"([A-Za-z]+)\.") para identificar classes sociais e faixas etárias específicas (ex: Master para meninos).
SibSp e Parch: Unificados na métrica de tamanho de família (Family_Size) para melhor interpretação sociológica.
Solução do Problema / Resposta às Perguntas de Negócio
1. Os passageiros de primeira classe sobreviveram mais do que os de outras classes?
Análise: Sim. A classe social teve impacto direto na sobrevivência. A 1ª Classe obteve uma taxa de sobrevivência de 62,96% (136 de 216), a 2ª Classe obteve 47,28% (87 de 184) e a 3ª Classe registrou a menor taxa, com apenas 24,24% (119 de 491).
2. A porcentagem de sobrevivência das mulheres foi maior do que a dos homens?
Análise: Sim. O gênero foi a variável de maior peso. Das 314 mulheres a bordo, 233 sobreviveram (74,20%). Em contrapartida, dos 577 homens, apenas 109 sobreviveram (18,89%), refletindo a política de evacuação "mulheres e crianças primeiro".
3. A idade foi um fator determinante?
Análise: Sim. Crianças (0-12 anos) obtiveram a maior taxa de sobrevivência entre as faixas etárias (57,97% - 40 de 69). Adolescentes (13-18 anos) registraram 42,86%, Adultos (19-60 anos) registraram 38,88%, enquanto Idosos (60+ anos) tiveram a menor taxa (22,73%).
4. A presença de familiares facilitou ou dificultou a sobrevivência?
Análise: Pequenas famílias tiveram vantagem, enquanto famílias grandes sofreram prejuízo. Passageiros em famílias de 2 a 4 pessoas apresentaram taxas de sobrevivência elevadas (entre 55,28% e 72,41%). Passageiros sozinhos (Is_Alone = 1) tiveram 30,35% de sobrevivência. Já grupos familiares com 5 ou mais membros tiveram taxas reduzidas (famílias de 5 pessoas tiveram 20,00% e famílias com 8 ou 11 membros tiveram 0% de sobrevivência).
5. A tarifa (Fare) paga influenciou diretamente na sobrevivência?
Análise: Sim. Ao dividir os bilhetes em quartis de tarifa:
1º Quartil (menores tarifas): 19,73% de sobrevivência.
2º Quartil: 30,36% de sobrevivência.
3º Quartil: 45,50% de sobrevivência.
4º Quartil (maiores tarifas): 58,11% de sobrevivência.
A tarifa média dos sobreviventes foi de $48,39, contra $22,12 entre os não sobreviventes.
Autoavaliação
O objetivo principal de construir um pipeline de Engenharia de Dados completo no Databricks utilizando PySpark e responder a todas as perguntas de negócio foi atingido com sucesso.
A implementação da Arquitetura Medalhão permitiu manter a rastreabilidade desde o dado bruto na camada Bronze, passando pela limpeza na Silver, até a estruturação de um Star Schema maduro na camada Gold.
Entre as principais dificuldades superadas, destacam-se a adequação do acesso ao sistema de arquivos no Databricks (evitando erros de permissão em caminhos locais), a escolha adequada das técnicas de imputação de nulos para a coluna Age e a aplicação correta do comando display() para renderização imediata das tabelas agregadas.
Para trabalhos futuros, almeja-se implementar a orquestração automatizada desse pipeline via Databricks Workflows (Jobs) e a conexão direta do modelo Star Schema ao Power BI para disponibilização de dashboards interativos em tempo real.
