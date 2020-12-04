# Lab08

Estrutura de pastas:

```
└── README.md  <- arquivo apresentando a tarefa
```

## Tarefa de XPath e XQuery

## Questão 1

Construa uma comando SELECT que retorne dados equivalentes a este XPath

~~~xpath
//individuo[idade>20]/@nome
~~~

### Resolução

~~~sql
SELECT nome FROM Fichario WHERE idade > 20
~~~

## Questão 2

Qual a outra maneira de escrever esta query sem o where?

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~

### Resolução

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo[idade>17])
return {data($i/@nome)}
~~~

## Questão 3

Escreva uma consulta SQL equivalente ao XQuery:

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~

### Resolução

~~~sql
SELECT nome FROM Fichario WHERE idade > 17
~~~

## Questão 4

Retorne quantas publicações são posteriores ao ano de 2011.

### Resolução

~~~xquery
let $publi := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')
for $publication in ($publi/publications/publication[year>2011])
let $year := $publication/@year
group by $year
return {concat('Número de publicações posteriores ao ano de 2011: ', count($publication))}
~~~

## Questão 5

Retorne a categoria cujo <label> em inglês seja 'e-Science Domain'.

### Resolução

~~~xquery
let $publi := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')
for $category in ($publi/publications/categories/category)
where $category/label/@lang = 'en-US' and $category/label = 'e-Science Domain'
return $category
~~~

## Questão 6

Retorne as publicações associadas à categoria cujo <label> em inglês seja 'e-Science Domain'. A associação entre o label e a key da categoria deve ser feita na consulta.

### Resolução

~~~xquery
let $publi := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')
return <query>
{
    for $publication in ($publi/publications/publication),
        $category in ($publi/publications/categories/category)
    where $category/label/@lang = 'en-US' and $category/label = 'e-Science Domain' and $publication/key = $category/@key

    return $publication
}
</query>
~~~

## DRON - The Drug Ontology

## Questão 1

Liste todas as classificações que estão dois níveis abaixo da raiz

~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
for $second in ($data/*/*/*)
return {data($second/@name), '&#xa;'}
~~~

## Questão 2

Apresente todas as classificações de um componente a sua escolha (diferente de Acetylsalicylic Acid) que estejam hierarquicamente dois níveis acima. Note que no exemplo dado em sala foi considerado um nível hierárquico acima.

~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
for $class in ($data//*[*/*/@name = 'ANSAMYCIN'])
return {data($class/@name), '&#xa;'}
~~~

## Questão 3

Dado o dataset integrado criado via acesso a serviços no Jupyter. Este notebook tem os nomes (e sinônimos) de componentes recuperados do PubChem, em conjunto com o ChEBI do mesmo.

O dataset está disponível em:

https://github.com/santanche/lab2learn/blob/master/data/pubchem/pubchem-chebi-synonyms.xml

Deve ser recuperado em XQuery no endereço:

https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml

### Questão 3.1

Liste todos os códigos ChEBI dos componentes disponíveis.

~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')
for $component in ($data//Information)
return {data($component/Synonym[1]), '&#xa;'}
~~~

### Questão 3.2

Liste todos os códigos ChEBI e primeiro nome (sinônimo) de cada um dos componentes disponíveis.

~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')
for $component in ($data//Information)
return {data($component/Synonym[1]), '&#xa;', data($component/Synonym[2]) ,'&#xa;------------------------------------------------------&#xa;'}
~~~

### Questão 3.3

Para cada código ChEBI, liste os sinônimos e o nome que aparece para o mesmo componente no DRON (se existir). Para isso faça um JOIN com o DRON.

~~~xquery
let $PUBCHEM := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')
let $DRON := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
for $componentPUB in ($PUBCHEM//Information)
return
{
    '********************&#xa;Dados PubChem&#xa;********************&#xa;',
    for $synonym in ($componentPUB/Synonym)
        return{data($synonym),'&#xa;-----------------------------&#xa;'},
    '********************&#xa;Dados DRON&#xa;********************&#xa;',
    for $componentDRON in ($DRON//drug)
    where $componentDRON/@id = concat('http://purl.obolibrary.org/obo/CHEBI_', substring($componentPUB/Synonym[1], 7)) 
        return {data($componentDRON/@name),'&#xa;-----------------------------&#xa;'},
    '########################&#xa;'
}
~~~
