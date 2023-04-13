-- 1) Quantos candidatos fizeram os concursos por Ano?

select ano_enem "Ano Enem", count(ano_enem) as "Candidatos" from candidatos_enem as candidatos
group by ano_enem

-- 2) Quantos Municipios existem em cada Unidade da Federação (Estados), 
--    informando o nome do Estado?

select uf_nome as "Estado", count(codigo_municipio) as "Municipios" from municipios_ibge mi 
group by uf_nome
order by "Municipios" desc

-- 3) Quantos candidatos fizeram os concursos somente nos municipios Capitais 
--    dos estados em cada Ano?

select nome_municipio "Municipio",uf "UF", count(uf_codigo) as "Candidatos" from candidatos_enem ce
left join municipios_ibge mi on ce.municipio_prova = mi.codigo_municipio_dv
where municipio_capital = 'Sim'
group by nome_municipio, uf
order by "Candidatos" desc

-- 4) Quantos candidatos fizeram os concursos por Ano, em cada Região?

select ano_enem "Ano Enem", regiao_nome "Região", count(numero_inscricao) "Candidatos" from candidatos_enem ce
left join municipios_ibge mi on ce.municipio_prova = mi.codigo_municipio_dv
group by regiao_nome, ano_enem
order by regiao_nome desc, "Candidatos" desc

-- 5) Crie a tabela "Candidatos_ENEM_2021" somente com os candidatos do Ano de 2021;

create table Candidatos_ENEM_2021
(
 Ano_Enem          Char(04) Not null check (Ano_Enem = '2021'),
 Numero_Inscricao  Char(20) Not Null, 
 UF_Prova          char(2) not null,  
 MUNICIPIO_PROVA   Char(7) not null,
 COR_RACA          Char(15) not null, 
 SEXO              Char(15) Not Null,
 Idade             Numeric (3) Not Null,
 NOTA_CIENCIAS_HUMANAS        Numeric (6,2) Not Null,
 NOTA_LINGUAGENS_E_CODIGOS    Numeric (6,2) Not Null,
 NOTA_CIENCIAS_DA_NATUREZA   Numeric (6,2) Not Null,
 NOTA_MATEMATICA             Numeric (6,2) Not Null,
 NOTA_REDACAO                Numeric (6,2) Not Null,
 NOTA_MEDIA                  Numeric (6,2) Not Null,
Constraint pk_Candidatos_ENEM_2021_W Primary key(Ano_Enem, Numero_Inscricao),
Constraint fk_candidatos_uf foreign key (UF_PROVA) references unidade_federacao (co_uf_prova),
Constraint fk_candidatos_municipio foreign key (MUNICIPIO_PROVA) references municipios_ibge (CODIGO_MUNICIPIO_DV)
);

-- 6) Utilizando a tabela "Candidatos_ENEM_2021", selecione os candidatos que 
--    tiraram a nota de matemática maior que a média aritmética de matemática;

select numero_inscricao "Inscrição", nota_matematica "Nota Matemática", round((select AVG(nota_matematica) from candidatos_enem_2021), 1) as "Média Matematica"
from candidatos_enem_2021
where nota_matematica > (select AVG(nota_matematica) from candidatos_enem_2021)
order by nota_matematica desc

-- 7) Utilizando a tabela "Candidatos_ENEM_2021", selecione os candidatos que 
--    tiraram a nota de ciências da natureza maior que a mediana da nota de ciências da Natureza;

select numero_inscricao AS "Inscrição", nota_ciencias_da_natureza AS "Nota Ciências", round( cast(
      (select percentile_cont(0.5) within group (order by nota_ciencias_da_natureza) from candidatos_enem_2021) as numeric), 1) as "Média Ciências"
from candidatos_enem_2021
where nota_ciencias_da_natureza > (select PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY nota_ciencias_da_natureza) FROM candidatos_enem_2021)
ORDER BY nota_ciencias_da_natureza DESC

-- 8) Crie a tabela "Candidatos_ENEM_Sudeste" somente com os candidatos da Região Sudeste;

CREATE TABLE Candidatos_ENEM_Sudeste AS
SELECT *
FROM Candidatos_ENEM
WHERE UF_Prova IN ('SP', 'RJ', 'MG', 'ES');


-- 9) Utilizando a tabela "Candidatos_ENEM_Sudeste", selecione os candidatos que 
--    tiraram a nota de matemática maior que a média aritmética de linguagens e códigos;

select numero_inscricao "Inscrição", nota_matematica "Nota Matemática", round((select AVG(nota_linguagens_e_codigos) from Candidatos_ENEM_Sudeste), 1) as "Nota Linguagens"
from Candidatos_ENEM_Sudeste
where nota_matematica > (select AVG(nota_linguagens_e_codigos) from Candidatos_ENEM_Sudeste)
order by nota_matematica desc
