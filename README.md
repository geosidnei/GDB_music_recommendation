# GDB_music_recommendation
This page only have copied codes from Vlad Badushkov github page, his medium site and results from AI queries in an attempt to learn something about this theme. 

## 
# Bootcamp Neo4J - Análise de Dados com Grafos
## desafio do 2° módulo: Primeiros Passos com Cypher e Neo4j.

// Representar Usuários Músicas Artistas e Gêneros como nós em grafo
// achei estes dados: https://www.kaggle.com/code/vatsalmavani/music-recommendation-system-using-spotify-dataset/input 
// data.csv (30MB) 
// data_by_artist.csv(4.32 mb)  
// data_by_genres.csv (600 kb)  
// data_by_year.csv (22 kb)  
// data_w_gentes(5,2 mb) 

// Representar interações: artistas, gêneros, datas de gravações, escutar, curtir, seguir) como arestas com propriedades

// Utilizar consultas Cypher com base em conexões no grafo

// Query: "Quais artistas você recomendaria com base nos artistas de rock mais ouvidos?"

##
Fonte destes dados:

https://vladbatushkov.medium.com/one-month-graph-challenge-melodiya-3836951a905b

## 1a consulta: chama lastfm, desenrola os dados etc 

CALL apoc.load.html("https://www.last.fm/music", { data: "a.music-more-tags-tag-inner-wrap, section.music-section:eq(0) a.music-featured-item-heading-link" }) YIELD value

UNWIND value.data as n

MERGE (g:Genre { url: n.attributes.href, name: apoc.text.capitalizeAll(n.text) })

## Como a primeira consulta falhou recorri à IA do Google:

"O erro ocorre porque o Neo4J Aura (a versão em nuvem) restringe o uso de diversos procedimentos da biblioteca APOC por motivos de segurança, especialmente aqueles que realizam requisições HTTP externas ou acessam arquivos, como o apoc.load.html [2].
Para resolver isso e prosseguir com seu exercício em 2026, você tem duas alternativas principais:

## 1a alternativa: Usar uma aplicação intermediária (Recomendado)
Como o banco de dados não pode mais "buscar" a informação diretamente na web, você deve usar uma linguagem de programação (Python, JavaScript ou Java) para fazer o scraping e enviar os dados prontos para o Aura.

Exemplo em Python:
Você pode usar a biblioteca BeautifulSoup para extrair os dados e o driver oficial do Neo4j para Python para inseri-los:

#código python: 

import requests
from bs4 import BeautifulSoup
from neo4j import GraphDatabase

## 1. Extração dos dados
url = "https://www.last.fm/music"
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

tags = [a.get_text(strip=True) for a in soup.select("a.music-more-tags-tag-inner-wrap")]

## 2. Inserção no Neo4j Aura
driver = GraphDatabase.driver("neo4j+s://<seu-id>.databases.neo4j.io", auth=("neo4j", "sua-senha"))

with driver.session() as session:
    session.run("UNWIND $tags AS tagName MERGE (t:Tag {name: tagName})", tags=tags)"

## 2a alternativa para o problema: INSTALAR O NEO4J DESKTOP
## 2. Usar o Neo4j Desktop (Local)

Se o objetivo do exercício for estritamente aprender o comando apoc.load.html, você não conseguirá usar o Aura.
Baixe e instale o Neo4j Desktop.
Crie um banco de dados local.
Vá em Plugins e instale a biblioteca APOC.
Edite o arquivo neo4j.conf e adicione a linha:

apoc.import.file.enabled=true e apoc.http.browser_user_agent=true.

Nesse ambiente local, o seu comando original funcionará normalmente.
Por que a mudança?

O comando apoc.load.html foi movido para o pacote APOC Extended em versões recentes [1, 2]. 
O Neo4j Aura suporta apenas o APOC Core, que foca em utilitários internos de grafos e não em conexões externas de rede [2].
Para gerenciar seus bancos de dados e verificar as limitações atuais, você pode acessar o Console do Neo4j Aura.

De nada! Essa é a melhor estratégia: o Python resolve o problema de forma mais profissional e segura para ambientes em nuvem, enquanto o Desktop serve como um laboratório sem restrições.
Se você optar pelo Python, aqui estão três dicas rápidas para não ter erro:

## Instale as bibliotecas necessárias:

bash

pip install neo4j requests beautifulsoup4

Use o código com cuidado.

Cuidado com o Selecionador: O Last.fm costuma mudar as classes CSS com frequência. Se o código não retornar nada, abra o site no navegador, clique com o botão direito no elemento desejado e use a opção "Inspecionar" para confirmar se a classe music-more-tags-tag-inner-wrap ainda é a correta.

Credenciais do Aura: Lembre-se de baixar o arquivo credentials.txt que o Aura fornece ao criar a instância. Você precisará do Connection URI, Username (geralmente neo4j) e da Password.

Se precisar de ajuda para ajustar o código de inserção (o MERGE ou CREATE) quando os dados chegarem no Python, é só chamar! Boa sorte com o exercício.

De nada! Se precisar de mais alguma ajuda com o código em Python ou com as configurações do Neo4j Desktop em sua jornada de estudos em 2026, é só chamar.
## Dica final: Ao usar o Python, verifique sempre a documentação atualizada no Portal de Desenvolvedores do Neo4j para garantir a compatibilidade com as versões mais recentes das bibliotecas.

Boa sorte com seus grafos e com o Last.fm!

## Voltando aos códigos de Vlad Batushkov 
## 2a consulta: adicionando bandas aos gêneros

MATCH (g:Genre)

CALL apoc.load.html("https://last.fm" + g.url + "/artists", { data: "h3.big-artist-list-title a" }) YIELD value

UNWIND value.data as n

MERGE (b:Band { name: n.text })

MERGE (b)-[:OF]->(g)

## 3a consulta: finding a band workings in the many genres. Let’s name it «the most mix-genre bands».
MATCH (b:Band)-[:OF]->(g:Genre)

RETURN b.name, count(g) as genres

ORDER BY genres DESC

## 4a consulta: Finding a genre with bands connected with this genre, and at the same time these bands connected to other genres. 
// Let’s name it «the most mix-band genres».

MATCH (g1:Genre)<-[:OF]-(b:Band)-[:OF]->(g2:Genre)

RETURN g1.name as genre_name, count(DISTINCT b) as num_of_bands

ORDER BY num_of_bands DESC

## 5a consulta: list of Listeners. 
MATCH (b:Band)-[:OF]->(g:Genre { name: "Rock" })

CALL apoc.load.html("https://last.fm/music/" + replace(b.name, " ", "+") + "/+listeners", { data: "h4.user-list-name a" }) YIELD value

UNWIND value.data as n

MERGE (p:Person { name: n.text })

MERGE (p)-[:LIKES]->(b)

## 6a consulta: esses são os gostos do Vlad! 
### Eu perdi a vontade de fazer a minha versão logo depois de meu fracasso bem no começo...  :(
// I pick only 6 genres, that I prefer: Indie, Rock, British, Alternative, Metal, Electronic.

MATCH (p:Person)-[:LIKES]->(b:Band)

WITH p, count(b) as bands_likes

RETURN p.name as name, bands_likes

ORDER BY bands_likes DESC

### 6a consulta -- 2a parte (aqui seria a minha adaptação para meus gostos!)

WITH ["system of a down", "linkin park", "franz ferdinand", "oasis", "the killers", "arctic monkeys", "daft punk", "chemical brothers", "underworld", "kasabian", "queen", "red hot chili peppers", "the strokes", "foals", "the black keys", "rammstein", "imagine dragons", "coldplay"] as favorites

MATCH (b:Band)

WHERE apoc.coll.indexOf(favorites, toLower(b.name)) > -1

MERGE (p:Person { name: "vlad batushkov" })

MERGE (p)-[:LIKES]->(b)

## "This is not a full picture, but by this list I can recognize my music preferences" (Vlad Batushkov).
// 6a consulta -- 3a parte - a adaptação é aqui ó: E aqui tbm tem adaptação

MATCH (p:Person { name: "sidnei lopes" })-[:LIKES]->(b:Band)-[:OF]->(g:Genre) 

RETURN p, b, g

## 7a consulta:
## "What music bands can be recommended to me? Ok, how to solve this task." (Vlad Batushkov).
// One of the ways is: to find all persons, who listen same music bands as me. 
// Within  these persons find bands (excluding bands, that I already like) with biggest amount of followers.

MATCH (vb:Person { name: "sidnei lopes" })-[:LIKES]->(cross_band:Band)<-[:LIKES]-(p:Person)-[:LIKES]->(other_band:Band)

WHERE NOT (vb)-[:LIKES]->(other_band)

WITH other_band.name as name, count(p) as followers_num

RETURN name, followers_num

ORDER BY followers_num DESC

LIMIT 10

## References:
1) Github do projeto
https://gist.github.com/vladbatushkov/d92b4319763c99a5c5231fb699aaecc7#file-omgc-may-11-1-cypher
https://gist.github.com/vladbatushkov/4b70acb8d67f90e8b57853153b9beb27#file-omgc-may-11-2-cypher
https://gist.github.com/vladbatushkov/da5bb62d2c3b631d7d557f63fc315b24#file-omgc-may-11-3-cypher
https://gist.github.com/vladbatushkov/d7677a3c01d0ed82aeb3bd0f0ed6d2a6#file-omgc-may-11-4-cypher
https://gist.github.com/vladbatushkov/4c6061432ab78332f35e297f00018367#file-omgc-may-11-5-cypher

2) demais códigos foram retirados da página "Building a Neo4j Recommendation System with Cypher query
May 11: Melodiya, One Month Graph Challenge, de Vlad Batushkov - postado por ele em 
11 de Maio de 2019. Disponível em: 
https://vladbatushkov.medium.com/one-month-graph-challenge-melodiya-3836951a905b.
Acesso em: 27 de janeiro de 2026.
##
# Uma curiosidade minha sobre o nome da empresa virou uma pergunta à IA da Google:
Oi google, vc pode explicar o nome da empresa Neo4j pra mim?

O nome da empresa
Neo4j tem uma origem técnica, nascida no contexto do desenvolvimento de software em Java durante a década de 2000. 
Aqui está a explicação detalhada:

    Neo: Refere-se à Neo Technology, o nome original da empresa fundada por Emil Eifrem, Johan Svensson e Peter Neubauer, que iniciou o desenvolvimento do projeto em 2000.
    
    4j: É uma convenção comum em projetos open-source e bibliotecas daquela época (como Log4j, JUnit), que significa "for Java" (para Java), indicando que o banco de dados foi construído nativamente usando a linguagem de programação Java. 

Resumo: O nome representa a tecnologia original (Neo) feita para Java (4j). Embora hoje em dia o banco de dados tenha evoluído muito além de ser apenas uma biblioteca Java, o nome se consolidou. 

Curiosidade: Em um blog, o CEO Emil Eifrem explicou que, como o nome se tornou mundialmente reconhecido, ele "veio para ficar", mesmo que a parte "4j" pareça um anacronismo hoje em dia. 

O que faz a Neo4j?
É o banco de dados em grafos líder mundial, especializado em armazenar e consultar dados complexos e altamente conectados (como redes sociais, sistemas de recomendação e detecção de fraudes), diferentemente dos bancos de dados relacionais tradicionais que usam tabelas. 

Consultado em 28 de janeiro de 2026
