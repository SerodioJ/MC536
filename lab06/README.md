# Lab06

Estrutura de pastas:

```
└── README.md  <- arquivo apresentando a tarefa
```

## Tarefa de Cypher e o FDA Adverse Event Reporting System (FAERS)

## Exercício 1

Escreva uma sentença em Cypher que crie o medicamento de nome `Metamizole`, código no DrugBank `DB04817`.

### Resolução
~~~cypher
CREATE (:Drug {drugbank: "DB04817", name:"Metamizole"})
~~~

## Exercício 2

Considerando que a `Dipyrone` e `Metamizole` são o mesmo medicamento com nomes diferentes, crie uma aresta com o rótulo `:SameAs` que ligue os dois.

### Resolução
~~~cypher
MATCH (d1:Drug {name:"Dipyrone"})
MATCH (d2:Drug {name:"Metamizole"})
CREATE (d1)-[:SameAs]->(d2)
CREATE (d1)<-[:SameAs]-(d2)
~~~

## Exercício 3

Use o `DELETE` para excluir o relacionamento que você criou (apenas ele).

### Resolução
~~~cypher
MATCH (d1:Drug)-[r:SameAs]->(d2:Drug) WHERE d1.name in ['Dipyrone', 'Metamizole'] AND d2.name in ['Dipyrone', 'Metamizole']
DELETE r
~~~

## Exercício 4

Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
MATCH (p1:Pathology)<-[a:Treats]-(d:Drug)-[b:Treats]->(p2:Pathology)
MERGE (p1)<-[s:SameTreatment]->(p2)
ON CREATE SET s.weight=1
ON MATCH SET s.weight=s.weight+1
~~~

## Exercício 5

Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
~~~cypher
//Criacao de nos para cada uma das pessoas
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MERGE (p:Person {code: line.idperson})

//Indice para os nos de pessoas usando o code(idPerson) como chave
CREATE INDEX ON :Person(code)

//Criacao das arestas de Uso de Medicamento por Pessoa (Peson USES Drug)
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS use
MATCH (d:Drug {code: use.codedrug})
MATCH (p:Person {code: use.idperson})
MERGE (p)-[u:Uses]->(d)

//Criacao das arestas de Efeito Colateral por Pessoa (Person HAS SideEffect(:Pathology))
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS side
MATCH (p:Person {code: side.idPerson})
MATCH (s:Pathology {code: side.codePathology})
MERGE (p)-[:Has]->(s)

//Criacao da aresta de Medicamentos que causam Efeito Colateral utilizado Pessoa como base (Drug CAUSES SideEffect(:Pathology))
MATCH (s:Pathology)<-[:Has]-(p:Person)-[:Uses]->(d:Drug)
MERGE (d)-[c:Causes]->(s)
ON CREATE SET c.weight=1
ON MATCH SET c.weight=c.weight+1
~~~

## Exercício 6

Que tipo de análise interessante pode ser feita com esse grafo?

Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

Podemos verificar quais medicamentos possuem efeitos colaterais em comum (i.e. A causa P1 e B cause P1 então A e B são relacionados).

### Resolução
~~~cypher
//Sao verificados os medicamentos que causam o mesmo efeito colateral uma quantidade consideravel de vezes(10 no nosso caso), eles sao entao relacionados e a aresta que os conecta possui um peso indicando quantos efeitos colaterais os medicamentos tem em comum (i.e. Medicamento A causa 1,2,5 (mais de 10 vezes) e 3 (apenas uma vez) e Medicamento B causa 2,3,4,5 (mais de 10 vezes) -> a aresta que os conecta tera peso 2, ja que possuem 2 e 5 como efeitos colaterais em comum que ocorreram mais de 10 vezes). 

MATCH (d1:Drug)-[a:Causes]->(s:Pathology)<-[b:Causes]-(d2:Drug)
WHERE a.weight>10 and b.weight>10
MERGE (d1)<-[e:SameSideEffect]->(d2)
ON CREATE SET e.weight=1
ON MATCH SET e.weight=e.weight+1
~~~